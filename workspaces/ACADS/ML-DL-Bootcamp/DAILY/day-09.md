# Day 9 — Transformer Internals (build nanoGPT)

**Industry context:** You can't ship LLM products if you don't know how attention works. Every interview will ask you to derive the attention formula or compare GPT to BERT. Today you build a character-level GPT *from scratch* in PyTorch, following Karpathy's video — the single highest-quality educational ML video in existence.

**Objective by EOD:**
- Implement multi-head self-attention from scratch.
- Build a decoder-only transformer with N blocks.
- Train it on a small Shakespeare corpus.
- Sample text from it.
- **Target:** validation loss ≤ 1.5 after 5000 iters (≈ "reads like noisy Shakespeare").

**The video:** Andrej Karpathy — *Let's build GPT: from scratch, in code, spelled out.* https://www.youtube.com/watch?v=kCc8FmEb1nY (1h 56min).

**Repo to skim afterwards:** https://github.com/karpathy/nanoGPT (don't copy — write your own).

---

## 09:00–10:00 — Theory

**Read first, then watch.** This is non-negotiable.

1. *Attention Is All You Need* (Vaswani et al. 2017) — read **Abstract, Section 3.1, 3.2, 3.3**. Skip 3.4 and beyond. https://arxiv.org/abs/1706.03762.
2. *The Annotated Transformer* — first half only: https://nlp.seas.harvard.edu/2018/04/03/attention.html.
3. Open the *Illustrated Transformer* — just read it once, no notes: https://jalammar.github.io/illustrated-transformer/.

**The formula to memorize:**

```
Attention(Q, K, V) = softmax( (Q K^T) / sqrt(d_k) ) V
```

`Q, K, V` are `[batch, seq_len, d]`. `Q K^T` is `[batch, seq_len, seq_len]` — the affinity matrix. Softmax normalizes affinities to weights. Multiply by `V` to get output values weighted by attention.

For *self-attention*, Q, K, V are all linear projections of the **same** input. For *cross-attention*, Q comes from the decoder and K, V from the encoder.

For *causal* (decoder-only) self-attention, mask out future positions before softmax. That's the GPT trick.

---

## 10:00–13:00 — Build A: follow Karpathy (first 90 minutes of the video)

Watch on 1.25x or 1.5x speed. **Pause and type with him.** Do not copy-paste from his GitHub.

Create `code/day09/gpt.py`. By minute 90 of the video, your file should contain:

1. Loaded Shakespeare data (`tinyshakespeare.txt` — Karpathy provides).
2. Character-level tokenizer (`stoi`, `itos`, `encode`, `decode`).
3. Batch generator (`get_batch`).
4. A `BigramLanguageModel` (the simplest baseline).
5. Training loop.

```bash
mkdir -p data/raw/day09 && cd data/raw/day09
wget https://raw.githubusercontent.com/karpathy/char-rnn/master/data/tinyshakespeare/input.txt
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
```

**Don't skip the bigram model.** The pedagogical point is that even a 1-token-context model achieves loss ≈ 2.5, and any attention-based model beats that. The bigram is your sanity baseline.

**Acceptance for Build A:**
- Your bigram model trains, loss decreases from ~4.6 → ~2.5.
- You can sample text from it. It looks like random Shakespeare-noise but at least respects character frequencies.

---

## 13:00–14:00 — Lunch + Day-8 Quiz

---

## 14:00–17:00 — Build B: from bigram to transformer (minutes 90–116 of video)

Now you add **the attention machinery**. Type along with the video.

Your final architecture in `gpt.py`:

```python
"""Day 9 — Character-level GPT, hand-written, trained on tiny Shakespeare."""
from __future__ import annotations
import math
import torch
import torch.nn as nn
import torch.nn.functional as F
from pathlib import Path

ART = Path("artifacts/day09"); ART.mkdir(parents=True, exist_ok=True)
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"

# --- Hyperparameters ---
BATCH_SIZE = 64
BLOCK_SIZE = 256       # context length (chars)
MAX_ITERS = 5000
EVAL_INTERVAL = 500
LR = 3e-4
N_EMBED = 384
N_HEAD = 6
N_LAYER = 6
DROPOUT = 0.2
torch.manual_seed(1337)

# --- Data ---
text = open("data/raw/day09/input.txt").read()
chars = sorted(set(text))
vocab_size = len(chars)
stoi = {c: i for i, c in enumerate(chars)}
itos = {i: c for c, i in stoi.items()}
encode = lambda s: [stoi[c] for c in s]
decode = lambda ids: "".join(itos[i] for i in ids)

data = torch.tensor(encode(text), dtype=torch.long)
n = int(0.9 * len(data))
train_data, val_data = data[:n], data[n:]


def get_batch(split: str):
    d = train_data if split == "train" else val_data
    ix = torch.randint(len(d) - BLOCK_SIZE, (BATCH_SIZE,))
    x = torch.stack([d[i:i + BLOCK_SIZE] for i in ix])
    y = torch.stack([d[i + 1:i + 1 + BLOCK_SIZE] for i in ix])
    return x.to(DEVICE), y.to(DEVICE)


@torch.no_grad()
def estimate_loss(model, eval_iters: int = 200):
    out = {}
    model.eval()
    for split in ["train", "val"]:
        losses = torch.zeros(eval_iters)
        for k in range(eval_iters):
            x, y = get_batch(split)
            _, loss = model(x, y)
            losses[k] = loss.item()
        out[split] = losses.mean().item()
    model.train()
    return out


class Head(nn.Module):
    """One head of self-attention."""

    def __init__(self, head_size: int):
        super().__init__()
        self.key = nn.Linear(N_EMBED, head_size, bias=False)
        self.query = nn.Linear(N_EMBED, head_size, bias=False)
        self.value = nn.Linear(N_EMBED, head_size, bias=False)
        self.register_buffer("tril", torch.tril(torch.ones(BLOCK_SIZE, BLOCK_SIZE)))
        self.dropout = nn.Dropout(DROPOUT)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        B, T, C = x.shape
        k = self.key(x)    # (B,T,hs)
        q = self.query(x)  # (B,T,hs)
        wei = q @ k.transpose(-2, -1) * (k.shape[-1] ** -0.5)
        wei = wei.masked_fill(self.tril[:T, :T] == 0, float("-inf"))
        wei = F.softmax(wei, dim=-1)
        wei = self.dropout(wei)
        v = self.value(x)
        return wei @ v


class MultiHeadAttention(nn.Module):
    def __init__(self, num_heads: int, head_size: int):
        super().__init__()
        self.heads = nn.ModuleList([Head(head_size) for _ in range(num_heads)])
        self.proj = nn.Linear(head_size * num_heads, N_EMBED)
        self.dropout = nn.Dropout(DROPOUT)

    def forward(self, x):
        out = torch.cat([h(x) for h in self.heads], dim=-1)
        return self.dropout(self.proj(out))


class FeedForward(nn.Module):
    def __init__(self, n_embed: int):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_embed, 4 * n_embed),
            nn.GELU(),                # GPT-style: GELU not ReLU
            nn.Linear(4 * n_embed, n_embed),
            nn.Dropout(DROPOUT),
        )

    def forward(self, x):
        return self.net(x)


class Block(nn.Module):
    """Pre-norm transformer block."""

    def __init__(self, n_embed: int, n_head: int):
        super().__init__()
        head_size = n_embed // n_head
        self.sa = MultiHeadAttention(n_head, head_size)
        self.ffwd = FeedForward(n_embed)
        self.ln1 = nn.LayerNorm(n_embed)
        self.ln2 = nn.LayerNorm(n_embed)

    def forward(self, x):
        x = x + self.sa(self.ln1(x))
        x = x + self.ffwd(self.ln2(x))
        return x


class GPT(nn.Module):
    def __init__(self):
        super().__init__()
        self.tok_emb = nn.Embedding(vocab_size, N_EMBED)
        self.pos_emb = nn.Embedding(BLOCK_SIZE, N_EMBED)
        self.blocks = nn.Sequential(*[Block(N_EMBED, N_HEAD) for _ in range(N_LAYER)])
        self.ln_f = nn.LayerNorm(N_EMBED)
        self.head = nn.Linear(N_EMBED, vocab_size)

    def forward(self, idx, targets=None):
        B, T = idx.shape
        tok = self.tok_emb(idx)
        pos = self.pos_emb(torch.arange(T, device=DEVICE))
        x = tok + pos
        x = self.blocks(x)
        x = self.ln_f(x)
        logits = self.head(x)
        if targets is None:
            return logits, None
        B, T, V = logits.shape
        loss = F.cross_entropy(logits.view(B * T, V), targets.view(B * T))
        return logits, loss

    @torch.no_grad()
    def generate(self, idx, max_new_tokens: int = 500):
        for _ in range(max_new_tokens):
            idx_cond = idx[:, -BLOCK_SIZE:]
            logits, _ = self(idx_cond)
            logits = logits[:, -1, :]
            probs = F.softmax(logits, dim=-1)
            next_id = torch.multinomial(probs, num_samples=1)
            idx = torch.cat([idx, next_id], dim=1)
        return idx


def main():
    model = GPT().to(DEVICE)
    n_params = sum(p.numel() for p in model.parameters())
    print(f"Model params: {n_params/1e6:.2f}M")

    optim = torch.optim.AdamW(model.parameters(), lr=LR)

    for it in range(MAX_ITERS + 1):
        if it % EVAL_INTERVAL == 0 or it == MAX_ITERS:
            losses = estimate_loss(model)
            print(f"step {it:5d}: train_loss={losses['train']:.4f}  val_loss={losses['val']:.4f}")

        x, y = get_batch("train")
        _, loss = model(x, y)
        optim.zero_grad(set_to_none=True)
        loss.backward()
        optim.step()

    torch.save(model.state_dict(), ART / "gpt.pt")

    # Generate sample
    ctx = torch.zeros((1, 1), dtype=torch.long, device=DEVICE)
    sample = decode(model.generate(ctx, max_new_tokens=1000)[0].tolist())
    Path(ART / "sample.txt").write_text(sample)
    print(sample[:500])


if __name__ == "__main__":
    main()
```

Run on GPU. Expected timings:
- T4 (Kaggle): ~25 min for 5000 iters
- A100 / 4090: ~5 min

**Acceptance criteria:**
- 5000 iters completed.
- Val loss ≤ 1.5.
- `sample.txt` contains 1000 chars that *look* like Shakespeare (correct character distribution, syllable-like words, named speakers — actual words may be made up; that's fine for char-level).

If val loss plateaus above 1.5: most likely cause is wrong block size or you forgot the LayerNorm. Debug with Karpathy's video paused.

---

## 17:00–18:00 — Code Review (and self-check)

```bash
git add code/day09/ artifacts/day09/
git commit -m "day-9: nanoGPT char-level transformer trained on Shakespeare"
```

Prompt:

```
Review code/day09/. Tomorrow's interviewer is going to ask "build self-attention
on a whiteboard" — calibrate me.

1. In my `Head.forward`, walk through every tensor shape from input to output.
   What is the matrix `wei` representing physically?
2. Why multiply by `head_size ** -0.5` before softmax?
3. The mask `tril` — what does it implement and why is it called "causal"?
4. Pre-norm vs post-norm: I used pre-norm. Why is it easier to train deep?
5. `MultiHeadAttention` runs heads in a Python loop. Is this efficient? Show me the
   single-matrix-multiplication version (one big linear, reshape to heads).
6. Position embedding is learned, not sinusoidal. When does that matter?
7. My `generate` clips idx to last BLOCK_SIZE tokens — what does that mean about
   context length at inference?

End: write me a 30-second whiteboard answer to "explain attention."
```

Save to `reviews/day09-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Rewrite `MultiHeadAttention` as a single matmul.** This is the production version — one `nn.Linear(N_EMBED, 3*N_EMBED)` that projects to `[Q, K, V]` at once, reshape to `[B, n_head, T, head_size]`, do attention in batch. Verify outputs match the loop version. This is the form interviewers will ask you to whiteboard.
- **B: Use `F.scaled_dot_product_attention`.** PyTorch 2.x has a fused attention op with Flash Attention support. Drop it in and benchmark.
- **C: Read** "Generating Long Sequences with Sparse Transformers" abstract + section 3 only — to understand context-length scaling beyond what nanoGPT does.
- **D: Sample with temperature + top-k.** Modify `generate` to accept `temperature` and `top_k`. Compare samples at temp=0.3 (conservative) vs temp=1.0 (default) vs temp=1.5 (chaotic).
- **E: Track loss in W&B.** Already a habit; add it.

---

## 21:00–21:30 — Journal

Add one extra line tonight: **"the one thing I'd struggle to explain to an interviewer."** That goes on tomorrow's morning re-read.

---

## Common Pitfalls

- **Pitfall 1:** Forgetting the mask → information leaks from future positions during training → loss looks too good, model fails to generate.
- **Pitfall 2:** Scaling by `1/sqrt(d_k)` not `1/sqrt(head_size)`. They're the same in single-head but you'd be surprised how often this is wrong in homegrown code.
- **Pitfall 3:** Position embeddings of wrong shape — must be `[BLOCK_SIZE, N_EMBED]`, indexed by `torch.arange(T)`. Off-by-one or wrong-axis here silently destroys the model.
- **Pitfall 4:** Computing `cross_entropy` on the wrong reshape — `logits.view(-1, V)` and `targets.view(-1)` must be aligned. If you mix up B and T, loss looks too high.
- **Pitfall 5:** Generating with no `max(idx_cond, BLOCK_SIZE)` clipping → tensor grows unboundedly and crashes.

---

## Quiz (tomorrow 13:00)

`code/day09/quiz_answers.md`.

1. In the formula `softmax(QK^T / sqrt(d_k)) V`, why specifically `sqrt(d_k)` and not just any normalization?
2. Why does causal masking happen *before* softmax (using `-inf`), not after?
3. Self-attention's complexity in sequence length T is O(T²). Why? What does this mean for context windows of 100k tokens?
4. Multi-head attention with `n_head=6` and `head_size=64` — what's the equivalent single-head dimensionality, and why is multi-head still better?
5. Pre-norm: `x = x + attn(LN(x))`. Post-norm: `x = LN(x + attn(x))`. Which one is GPT-style and why?
6. What's the difference between GPT (decoder-only), BERT (encoder-only), and T5 (encoder-decoder)? In one sentence each.
7. KV-cache at inference — what's cached, why, and what does it save?
8. A 7B parameter LLM uses ~14GB in fp16. Walk through the math.

---

→ Tomorrow at 09:00: open `DAILY/day-10.md`. **You'll fine-tune a real BERT model.**
