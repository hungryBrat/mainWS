# Day 11 — RAG (Retrieval-Augmented Generation)

**Industry context:** Half of LLM "AI" job postings in 2026 are some flavor of RAG. The pattern: retrieve relevant chunks from a vector store, stuff them into the prompt, let the LLM answer. Today you build one end-to-end on real documents (PyTorch's own docs), evaluate it, and learn the failure modes.

**Objective by EOD:**
- Build a RAG pipeline over the PyTorch docs corpus.
- Stack: `sentence-transformers` embeddings + FAISS index + Anthropic Claude Sonnet for generation.
- Evaluate on 20 hand-written Q&A pairs.
- **Target:** ≥ 15/20 answers judged "correct" by an LLM-as-judge.

---

## 09:00–10:00 — Theory

**Read:**
1. *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks* (Lewis et al. 2020) — **Abstract + Section 2.1 + Section 3 only**. https://arxiv.org/abs/2005.11401. (The original RAG paper.)
2. Pinecone's *RAG Overview*: https://www.pinecone.io/learn/retrieval-augmented-generation/ — read all of it.
3. Anthropic's prompting docs — **Long context tips**: https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/long-context-tips. 10 min.

**Two ideas to internalize:**

1. **Retrieval quality is the bottleneck.** A bad retriever gives the LLM bad context, and even the best LLM hallucinates. Most "RAG failures" are actually retrieval failures.
2. **Chunking strategy matters more than embedding model choice for first-pass quality.** Get chunking right (semantic, with overlap, ~500 tokens), then tune embeddings.

---

## 10:00–13:00 — Build A: corpus → index

Create `code/day11/`:

### 1. Get the corpus

PyTorch docs cloned as Markdown. Use the docs subset of the PyTorch tutorials repo:

```bash
mkdir -p data/raw/day11
git clone --depth=1 https://github.com/pytorch/tutorials.git data/raw/day11/tutorials
# Use just the docstring + tutorials, ~150 .py and .rst files
find data/raw/day11/tutorials -name "*.rst" -o -name "*.py" | head -5
```

Plus 3–5 articles you choose — for variety, also pull:

```bash
mkdir -p data/raw/day11/extra
curl -o data/raw/day11/extra/torch_tensors.md \
  https://raw.githubusercontent.com/pytorch/pytorch/main/docs/source/torch.rst
# Add a few more .rst files from pytorch/pytorch/docs/source — torch.nn, optim, autograd
```

Aim for ≥ 200 chunks total.

### 2. Build the index

`code/day11/build_index.py`:

```python
"""Build FAISS index over PyTorch docs."""
from __future__ import annotations
import json
import logging
from pathlib import Path

import faiss
import numpy as np
from sentence_transformers import SentenceTransformer

ART = Path("artifacts/day11"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day11")

EMBED_MODEL = "BAAI/bge-small-en-v1.5"   # 33M params, strong baseline, 384-d
CHUNK_SIZE = 500       # chars (≈ tokens for English)
CHUNK_OVERLAP = 75


def load_corpus(roots: list[Path]) -> list[dict]:
    """Return [{'path': str, 'text': str}] for every .rst / .md / .py file."""
    docs = []
    for root in roots:
        for ext in ("*.rst", "*.md", "*.py"):
            for p in root.rglob(ext):
                try:
                    txt = p.read_text(encoding="utf-8", errors="ignore")
                    if len(txt) > 200:
                        docs.append({"path": str(p), "text": txt})
                except Exception as e:
                    log.warning(f"skipping {p}: {e}")
    log.info(f"Loaded {len(docs)} files")
    return docs


def chunk(doc: dict, size: int = CHUNK_SIZE, overlap: int = CHUNK_OVERLAP) -> list[dict]:
    txt = doc["text"]
    chunks = []
    i = 0
    while i < len(txt):
        end = min(i + size, len(txt))
        # break at nearest \n
        if end < len(txt):
            soft = txt.rfind("\n", i + size - 100, end)
            if soft != -1:
                end = soft
        chunks.append({"path": doc["path"], "start": i, "end": end,
                       "text": txt[i:end].strip()})
        if end >= len(txt): break
        i = end - overlap
    return chunks


def main():
    roots = [Path("data/raw/day11/tutorials"), Path("data/raw/day11/extra")]
    docs = load_corpus(roots)

    chunks = [c for d in docs for c in chunk(d)]
    chunks = [c for c in chunks if len(c["text"]) > 50]
    log.info(f"Built {len(chunks):,} chunks")

    model = SentenceTransformer(EMBED_MODEL)
    texts = [c["text"] for c in chunks]
    embs = model.encode(texts, batch_size=64, show_progress_bar=True,
                        convert_to_numpy=True, normalize_embeddings=True)
    log.info(f"Embeddings shape: {embs.shape}")

    index = faiss.IndexFlatIP(embs.shape[1])   # inner-product over normalized = cosine
    index.add(embs.astype("float32"))

    faiss.write_index(index, str(ART / "index.faiss"))
    (ART / "chunks.jsonl").write_text("\n".join(json.dumps(c) for c in chunks))
    log.info(f"Saved {ART/'index.faiss'} and {ART/'chunks.jsonl'}")


if __name__ == "__main__":
    main()
```

Run: `uv run python code/day11/build_index.py`. Takes ~3–5 min depending on corpus size.

**Acceptance criteria:**
- ≥ 200 chunks indexed.
- `index.faiss` and `chunks.jsonl` saved.

---

## 13:00–14:00 — Lunch + Day-10 Quiz

---

## 14:00–17:00 — Build B: RAG + eval set

### 1. Retrieval + generation

`code/day11/rag.py`:

```python
"""RAG over the PyTorch docs index."""
from __future__ import annotations
import json
import os
from pathlib import Path

import faiss
import numpy as np
from anthropic import Anthropic
from dotenv import load_dotenv
from sentence_transformers import SentenceTransformer

load_dotenv()
ART = Path("artifacts/day11")
EMBED_MODEL = "BAAI/bge-small-en-v1.5"
GEN_MODEL = "claude-sonnet-4-6"
TOP_K = 5

_emb_model = SentenceTransformer(EMBED_MODEL)
_index = faiss.read_index(str(ART / "index.faiss"))
_chunks = [json.loads(l) for l in (ART / "chunks.jsonl").read_text().splitlines()]
_client = Anthropic()


def retrieve(query: str, k: int = TOP_K) -> list[dict]:
    q_emb = _emb_model.encode([query], normalize_embeddings=True).astype("float32")
    scores, ids = _index.search(q_emb, k)
    results = []
    for s, i in zip(scores[0], ids[0]):
        c = _chunks[int(i)]
        results.append({**c, "score": float(s)})
    return results


SYSTEM = """You are a helpful PyTorch expert. Answer the user's question using ONLY the provided context chunks.
If the answer is not in the context, say "I don't have enough information." Cite chunks by their bracket number, e.g., [2]."""


def answer(query: str, k: int = TOP_K) -> dict:
    ctx = retrieve(query, k)
    context_str = "\n\n".join(
        f"[{i+1}] (from {Path(c['path']).name})\n{c['text']}" for i, c in enumerate(ctx)
    )
    user_msg = f"Context:\n{context_str}\n\nQuestion: {query}"
    msg = _client.messages.create(
        model=GEN_MODEL, max_tokens=600, system=SYSTEM,
        messages=[{"role": "user", "content": user_msg}],
    )
    return {"query": query, "answer": msg.content[0].text, "context": ctx}


if __name__ == "__main__":
    import sys
    q = " ".join(sys.argv[1:]) or "What is autograd in PyTorch?"
    res = answer(q)
    print("\n=== ANSWER ===\n" + res["answer"])
    print("\n=== TOP-K CHUNKS ===")
    for i, c in enumerate(res["context"], 1):
        print(f"\n[{i}] (score={c['score']:.3f}) {c['path']}")
        print(c["text"][:200] + "...")
```

Test:

```bash
uv run python code/day11/rag.py "What is the difference between tensor.contiguous() and tensor.view()?"
```

You should get a coherent answer that references the retrieved chunks.

### 2. Eval set (20 Q&A pairs)

Create `code/day11/eval_set.jsonl`. Write **20 questions yourself** based on the indexed docs. Each line:

```json
{"q": "What does torch.no_grad() do?", "expected_substring": ["disables gradient", "memory", "inference"]}
```

A few categories to cover:
- Definitional ("What is X?")
- How-to ("How do I save a model?")
- Comparative ("Difference between X and Y?")
- Negative test ("How do I use Tensorflow in PyTorch?" → expect "I don't have enough information.")

### 3. Evaluation

`code/day11/eval.py`:

```python
"""Evaluate RAG answers via substring check + LLM-as-judge."""
import json
import os
from pathlib import Path

from anthropic import Anthropic
from dotenv import load_dotenv
from rag import answer

load_dotenv()
client = Anthropic()
JUDGE_MODEL = "claude-sonnet-4-6"

with open("code/day11/eval_set.jsonl") as f:
    eval_set = [json.loads(l) for l in f]

JUDGE_SYS = """You are an evaluator. Given a question and an answer, judge whether the answer is CORRECT, PARTIAL, or WRONG.
Respond with ONLY one word: CORRECT, PARTIAL, or WRONG."""

results = []
for ex in eval_set:
    out = answer(ex["q"])
    # substring check
    ans_lower = out["answer"].lower()
    substring_hit = any(s.lower() in ans_lower for s in ex.get("expected_substring", []))
    # LLM judge
    judge_resp = client.messages.create(
        model=JUDGE_MODEL, max_tokens=10, system=JUDGE_SYS,
        messages=[{"role": "user",
                   "content": f"Question: {ex['q']}\n\nAnswer: {out['answer']}"}],
    )
    verdict = judge_resp.content[0].text.strip().upper()
    results.append({"q": ex["q"], "verdict": verdict, "substring": substring_hit})
    print(f"{verdict:8s} subs={substring_hit}  Q: {ex['q'][:60]}")

correct = sum(r["verdict"] == "CORRECT" for r in results)
partial = sum(r["verdict"] == "PARTIAL" for r in results)
wrong   = sum(r["verdict"] == "WRONG" for r in results)
print(f"\n{correct} correct / {partial} partial / {wrong} wrong  out of {len(results)}")
Path("artifacts/day11/eval_results.json").write_text(json.dumps(results, indent=2))
```

Run: `uv run python code/day11/eval.py`

**Acceptance criteria:**
- 20 eval questions written.
- `eval.py` produces a verdict per question.
- ≥ 15/20 CORRECT (or CORRECT+PARTIAL together if you're generous — at least 18/20 in that bucket).

Below the bar? Common fixes (in priority):
1. **Increase top_k** to 8 or 10 (more context).
2. **Larger chunks** (1000 chars instead of 500) — better for definitional Qs.
3. **Better embedding model** — try `BAAI/bge-base-en-v1.5` (110M).
4. **Re-rank** — fetch top 20 with embeddings, then re-rank with a cross-encoder.

---

## 17:00–18:00 — Code Review

```bash
git add code/day11/ artifacts/day11/
git commit -m "day-11: RAG over PyTorch docs with FAISS + Claude Sonnet"
```

Prompt:

```
Review code/day11/. You are an LLM platform engineer at a Series-B startup
about to deploy this.

1. Chunking strategy: 500 chars with 75 overlap. Defend or refute for this corpus.
2. Embedding model: bge-small-en-v1.5. When would you upgrade and to what?
3. The eval set is 20 hand-written Qs. Industry standard is 200+. What's a
   reasonable next step to expand without me writing them all by hand?
4. The LLM-as-judge is the same model as the generator. What bias does this
   introduce, and how would you mitigate?
5. My retrieval uses cosine similarity only. When would I add BM25 / hybrid search?
6. The system prompt asks for citations. How would I extract those programmatically
   and verify they're real (not hallucinated)?
7. Cost: each eval call hits Anthropic 2x (gen + judge). Estimate the cost for
   1000-question eval. What's the cheaper alternative?

End: give me 3 changes for production deployment, ranked by impact.
```

Save to `reviews/day11-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Hybrid search.** Add BM25 (`rank_bm25`) alongside FAISS, rank-fuse the top-k from each. Re-evaluate. Should help on rare-keyword questions.
- **B: Cross-encoder re-ranking.** Use `cross-encoder/ms-marco-MiniLM-L-6-v2` to re-rank the top 20 from FAISS down to top 5. Re-evaluate. Usually +5–10% accuracy.
- **C: Citation verification.** Extract `[N]` references from each answer, check each cited chunk actually appears in the retrieved context. Report % of "honest" citations.
- **D: Read** "Retrieval Augmented Generation: Streamlining the creation of intelligent natural language processing models" (Meta engineering blog) — actually skip that, read instead Anthropic's own RAG cookbook: https://github.com/anthropics/anthropic-cookbook/tree/main/skills/retrieval_augmented_generation. 30 min.
- **E: A `streamlit run app.py` or FastAPI front-end.** Wrap RAG in a tiny web UI. Hold this thought — Day 12 is FastAPI for real.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Forgetting `normalize_embeddings=True` and then using `IndexFlatIP` — inner product over un-normalized vectors is *not* cosine similarity. Either normalize, or use `IndexFlatL2`.
- **Pitfall 2:** Chunk size > LLM context window-ish → wasted tokens. Chunk size too small (e.g., 50 chars) → fragments lose meaning.
- **Pitfall 3:** Asking the LLM to "use the context if relevant, otherwise your own knowledge" → hallucinations. Be strict: "ONLY the context."
- **Pitfall 4:** Eval set leakage — writing eval Qs after seeing what the system can answer. Write them *before*, blind.
- **Pitfall 5:** Token limits — long context chunks + long system prompt can blow past max input tokens silently. Track total tokens with `client.beta.messages.count_tokens(...)` or guard with a hard limit.

---

## Quiz (tomorrow 13:00)

`code/day11/quiz_answers.md`.

1. Why is normalized inner-product equivalent to cosine similarity?
2. What is the curse of dimensionality and why doesn't it sink modern dense retrieval?
3. When would you use a Sparse (BM25) retriever in addition to a dense one?
4. Re-ranking: why is a cross-encoder more accurate than a bi-encoder, and why is it slower?
5. Chunk overlap of 75 — what problem does it solve, and what's the cost?
6. RAG vs fine-tuning: which would you use for "the company has 10k internal product docs" vs "we want the model to write in our brand voice"?
7. Three signals that RAG retrieval is broken (vs the LLM being the problem)?
8. How does FAISS speed up nearest neighbor search vs brute-force? (HNSW, IVF, PQ — at a high level.)

---

→ Tomorrow at 09:00: open `DAILY/day-12.md`. **Deployment day.**
