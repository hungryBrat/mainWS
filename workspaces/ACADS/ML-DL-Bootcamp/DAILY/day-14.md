# Day 14 — Capstone & Interview Prep

**Industry context:** You've shipped 13 days of work. The last day is about packaging it: GitHub repo polished, README compelling, model cards in place, and your interview skills sharpened. The portfolio you ship today is what gets you the interview; the prep today is what gets you the offer.

**Objective by EOD:**
- A polished GitHub repo a recruiter would not be embarrassed by.
- A LIVE demo (Hugging Face Space or Streamlit Cloud) of at least one project.
- 10 ML coding drills completed under time pressure.
- 3 ML system design dry-runs done.
- An updated resume bullet list quoting your bootcamp metrics.

---

## 09:00–10:00 — GitHub Audit & Repo Polish

### Open your repo. Click through it as if you were a recruiter.

Score each (0–2) and total at the end:

| Item | 0 | 1 | 2 | Yours |
|---|---|---|---|---|
| README at root | absent | bare | with badges + screenshots + one-line value prop | |
| Top-level structure | flat / messy | organized | logical, navigable | |
| Best project linked | no | yes | linked + demo deployed | |
| Notebooks | dead, unreviewed | viewable | render cleanly on GitHub | |
| Model cards | none | one project | every Day 7+ project | |
| Reproducibility | no env | requirements.txt | uv lock or Dockerfile | |
| Commits | "wip" everywhere | mostly clear | conventional, signed | |
| Tests | none | a few | day-12 + day-2 have tests, others noted | |

Target: ≥ 12/16.

### Write the master README

`ML-DL-Bootcamp/README.md` already exists with bootcamp rules. **Now write the public-facing root README** at the actual GitHub repo root, by overwriting your initial commit's README. The audience is recruiters and hiring managers who'll spend 90 seconds on it.

Template (fill in numbers, replace `[..]` placeholders):

```markdown
# ML/DL Bootcamp Portfolio

14 days. 14 projects. Built from scratch to production-ready.

## Highlights

| Day | Project | Result | Stack |
|---|---|---|---|
| 4 | Home Credit default risk (XGBoost + SHAP) | ROC-AUC [X.XX] | XGBoost, LightGBM, SHAP |
| 7 | CIFAR-10 ResNet | [XX.X]% test accuracy | PyTorch |
| 8 | Oxford Flowers 102 (transfer learning) | [XX.X]% test accuracy | torchvision, W&B |
| 9 | nanoGPT char-level transformer | val loss [X.XX] | PyTorch — written by hand |
| 10 | DistilBERT + LoRA on SST-2 | [XX.X]% accuracy, ~1% params trained | transformers, peft |
| 11 | RAG over PyTorch docs | [XX]/20 eval correct | FAISS, sentence-transformers, Claude |
| 12 | FastAPI flower classifier | p95 [XXX]ms, Dockerized | FastAPI, Docker, Locust |
| 13 | Prometheus + Grafana monitoring | 4-panel dashboard, drift sentinel | Prometheus, structlog |

## Live demo

🔗 **[demo link to Hugging Face Space or Streamlit]** ← we deploy this today

## What I learned

- Implementing attention from scratch (Day 9) and shipping a fine-tuned BERT (Day 10).
- The full ML system lifecycle: data → model → eval → API → monitoring.
- LoRA, RAG, and the modern LLM stack in production-ready Python.

## How to run a project locally

Each `code/dayNN/` has its own README with one-command setup. Example:

```bash
git clone ...
cd ml-bootcamp/code/day12
docker compose up
```

## About the bootcamp

Self-designed 14-day curriculum, ~10 hrs/day. Built without Lightning, Trainer wrappers,
or notebook copy-paste. Every model trained from a hand-written loop or instrumented
HF pipeline. Every project has a code review file ([reviews/](reviews/)) and a metric.
```

Add a few screenshots: Grafana dashboard, W&B run, SHAP plot. Drop them in `assets/` and link from README.

### Promote the best project

Pick ONE flagship from your portfolio. Best candidate: **Day 12 + Day 13 combo** (FastAPI flower classifier with monitoring). It demonstrates *end-to-end* shipping.

Create `code/day12/README.md`:

```markdown
# Flower Classifier — production-ready service

Fine-tuned ResNet50 (Day 8) deployed as a Dockerized FastAPI service with
Prometheus metrics and Grafana dashboards.

## Quick start (one command)

```bash
docker compose up
```

- API: http://localhost:8000/docs
- Metrics: http://localhost:9090
- Dashboard: http://localhost:3000 (admin/admin)

## Architecture

[ascii or PNG diagram: client → FastAPI → TorchScript model; FastAPI → /metrics → Prometheus → Grafana]

## Performance

- Median latency: XX ms (CPU)
- p95 latency: XX ms (CPU)
- Image accuracy on Oxford Flowers 102 test: XX.X%

## What's here

- `app/` — FastAPI service
- `Dockerfile` — slim Python 3.11 image, ~1.2GB
- `docker-compose.yaml` — full local stack
- `grafana_dashboard.json` — exported 4-panel dashboard
- `tests/` — API tests
- `load_test.py` — Locust scenario
```

Push:

```bash
git add -A
git commit -m "day-14: GitHub polish + flagship project README"
git push
```

---

## 10:00–13:00 — Deploy a Live Demo

Pick **one** of these. Time-boxed: must be live by 13:00, not perfect.

### Option A (recommended): Hugging Face Space — Gradio + Day 10 sentiment

Easiest, smoothest. Free tier handles a small model.

```bash
uv add gradio
mkdir code/day14/space
```

`code/day14/space/app.py`:

```python
import gradio as gr
import torch
from transformers import AutoModelForSequenceClassification, AutoTokenizer
from peft import PeftModel

BASE = "distilbert-base-uncased"
ADAPTER = "<your-hf-handle>/distilbert-sst2-lora-bootcamp"

tok = AutoTokenizer.from_pretrained(BASE)
base = AutoModelForSequenceClassification.from_pretrained(BASE, num_labels=2)
model = PeftModel.from_pretrained(base, ADAPTER).eval()

def classify(text: str):
    enc = tok(text, return_tensors="pt", truncation=True, max_length=128)
    with torch.no_grad():
        probs = torch.softmax(model(**enc).logits, dim=-1)[0]
    return {"negative": float(probs[0]), "positive": float(probs[1])}

gr.Interface(
    fn=classify,
    inputs=gr.Textbox(label="Sentence", placeholder="It's a great day for ML."),
    outputs=gr.Label(num_top_classes=2),
    title="Sentiment classifier (DistilBERT + LoRA)",
    description="Trained Day 10 of my ML bootcamp. ~1% of params fine-tuned via LoRA.",
).launch()
```

`code/day14/space/requirements.txt`:

```
transformers>=4.40
peft>=0.11
torch>=2.5
gradio>=4.0
```

`code/day14/space/README.md` (with HF Space frontmatter):

```markdown
---
title: SST-2 Sentiment LoRA
emoji: 🌸
colorFrom: pink
colorTo: indigo
sdk: gradio
sdk_version: "4.0"
app_file: app.py
pinned: false
---

Try it: type a sentence, hit submit.
```

Create the space at https://huggingface.co/new-space, pick Gradio + CPU basic, then:

```bash
git clone https://huggingface.co/spaces/<your-handle>/sst2-bootcamp space-deploy
cp code/day14/space/* space-deploy/
cd space-deploy && git add -A && git commit -m "initial deploy" && git push
```

Wait ~3 min for build. **Live URL goes in your portfolio README.**

### Option B: Streamlit Cloud — Day 11 RAG demo

If you prefer to demo the RAG:

```python
import streamlit as st
from rag import answer

st.title("PyTorch Docs Q&A (RAG)")
q = st.text_input("Question:")
if q:
    with st.spinner("Retrieving + generating..."):
        r = answer(q)
    st.markdown("### Answer")
    st.write(r["answer"])
    with st.expander("Show context"):
        for i, c in enumerate(r["context"], 1):
            st.markdown(f"**[{i}]** {c['path']}\n\n{c['text'][:500]}")
```

Push to GitHub → Streamlit Community Cloud (free).

**Acceptance criteria:**
- A public URL anyone can open and use.
- Linked from portfolio root README.

---

## 13:00–14:00 — Lunch + Day-13 Quiz

---

## 14:00–17:00 — Interview Drills

Re-read `INTERVIEW_PREP.md` first.

### Coding drills (60 min, all ten)

Open `code/day14/interview/`. Time yourself: **6 minutes per drill, no Googling, no Claude.**

1. `drill_01_softmax.py` — vectorized stable softmax over the last axis.
2. `drill_02_cross_entropy.py` — cross-entropy from logits (numerical stability via log-sum-exp).
3. `drill_03_kmeans.py` — K-means from scratch, 10 iters, random init.
4. `drill_04_linreg_gd.py` — linear regression with gradient descent + plot loss curve.
5. `drill_05_knn.py` — vectorized k-NN classifier using broadcast trick.
6. `drill_06_auc.py` — AUC-ROC from `(y_true, y_score)`, sort + cumulative trick.
7. `drill_07_precision_at_k.py` — precision@k for a single ranked list.
8. `drill_08_sliding_window.py` — sliding-window batch generator.
9. `drill_09_attention.py` — single-head and multi-head attention forward pass (no PyTorch nn.Module).
10. `drill_10_lora.py` — LoRA `forward`: `y = Wx + α*B(Ax)`. With and without scaling.

After the timer rings, paste each drill into Claude with:

```
Review code/day14/interview/drill_<N>.py for vectorization, numerical stability,
and clarity. Be brief — 3 bullets max per drill.
```

### ML System Design dry-run (60 min, 3 prompts × 20 min)

Use a whiteboard, Excalidraw, or paper. Out loud. **Set a timer.**

**Prompt 1:** "Design a fraud detection system for an e-commerce platform processing 5M txns/day. Latency budget: 200ms per transaction. False positive cost: customer service ticket. False negative cost: chargeback."

**Prompt 2:** "Design a recommendation system for short-form video (TikTok-like). Cold-start matters. Engagement loops matter. Explain how you'd avoid filter bubbles."

**Prompt 3:** "Design a real-time speech-to-text service for a video-conferencing app. 100ms target latency. Multi-language. How do you evaluate? How do you ship updates without breaking live calls?"

For each prompt, follow the 8-step template from `INTERVIEW_PREP.md`. Time yourself. Record yourself if you can stomach it.

After each, paste your outline (text, not voice) to Claude:

```
I just rehearsed this ML system design prompt: "<prompt>". Here's my outline:
<paste 8 bullet sections>

Critique me as if I just gave this to you in a Meta L5 interview loop. Where am
I shallow? Where did I miss a tradeoff? What did I get right?
```

Save responses to `reviews/day14-systemdesign.md`.

---

## 17:00–18:00 — Resume Bullet Update

Open your resume. Find or create the "Projects" section. Replace with concrete bullets:

```
ML/DL BOOTCAMP — 14-day intensive
GitHub: github.com/<handle>/ml-bootcamp · Live demo: <hf-space-url>

- Fine-tuned ResNet50 on Oxford Flowers 102, hit 92.X% test accuracy using
  two-stage transfer learning with discriminative learning rates; exported to
  TorchScript for production.
- Built decoder-only transformer (nanoGPT-style) from scratch in PyTorch:
  custom self-attention, causal masking, multi-head, layer norm; trained
  on Shakespeare corpus to validation loss <X.X.
- Fine-tuned DistilBERT on SST-2 with LoRA: ~1.1% of params trained,
  achieved 90.X% accuracy on validation set vs 90.X% for full fine-tune,
  with adapter shrunk to ~3 MB. Pushed to HuggingFace Hub.
- Deployed flower classifier as FastAPI service in Docker container with
  Prometheus metrics, Grafana dashboards, and PSI drift detector. Sustained
  p95 latency <X ms under 10-user load test.
- Built RAG system over PyTorch docs with sentence-transformers + FAISS
  + Claude Sonnet, evaluated via LLM-as-judge on 20-question Q&A set,
  achieved XX/20 correct.

Stack: PyTorch · HuggingFace Transformers/PEFT · scikit-learn · XGBoost ·
W&B · Optuna · FastAPI · Docker · Prometheus · Grafana
```

Quantify everything. **No "worked with ML." Bad bullet. Numbers always.**

Send the bullets to Claude:

```
Here are 5 resume project bullets. Critique them for:
1. Verb strength (every bullet starts with a strong past-tense action verb).
2. Quantification — every bullet has a number.
3. Length — under 2 lines each.
4. Hiring-manager scannability.

Rewrite the weakest 2.
```

Save to `reviews/day14-resume.md`. Update your real resume PDF.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Final commit + reflection

### Final commit

```bash
git add -A
git commit -m "day-14: capstone — README, demo deploy, interview drills, model cards"
git push
```

### Update INDEX

Add this workspace to `mainWS/workspaces/INDEX.md` (the parent does this automatically — see CLAUDE.md for the workspace).

### Write a final review

Create `BOOTCAMP_RETROSPECTIVE.md` in the workspace root:

```markdown
# 14-Day Bootcamp Retrospective

## Numbers

- Hours actually worked: [X]
- Quizzes scored ≥ 80%: [X / 13]
- Days that hit acceptance criteria: [X / 14]
- Final GitHub stars: [N]
- Final HuggingFace model downloads: [N]
- Total experiments logged in W&B: [N]

## What worked

- 3 bullets

## What didn't

- 3 bullets

## What I'd change about the bootcamp itself

- 3 bullets

## Next 30 days

- 5 specific, dated commitments. Examples:
  - Apply to N roles by [date]
  - Complete [specific course or paper] by [date]
  - Build [specific extension project, e.g., extend the RAG to a different corpus] by [date]
```

### Done.

Close the laptop. You've earned it.

Tomorrow: open `INTERVIEW_PREP.md` again, re-read it, and start applying.

---

## Common Pitfalls (last time)

- **Pitfall 1:** Posting the GitHub repo with no README. The README is the entire first impression.
- **Pitfall 2:** Quantifying with vague terms ("significantly improved", "fast inference"). Use numbers.
- **Pitfall 3:** "I worked on..." Always "I built / shipped / fine-tuned / deployed..."
- **Pitfall 4:** Deploying a demo and then never opening it again — link rot. Add it to a monthly check.
- **Pitfall 5:** Treating the bootcamp end as the goal. The goal is the job. Keep building one project per week.

---

## Final Quiz

This one isn't graded by Claude. **Sit down with a notebook. Answer out loud, in full sentences.**

1. Walk me through your favorite project from these 14 days. Why was it your favorite?
2. Explain attention to someone who's only used scikit-learn.
3. What was the most surprising thing you learned?
4. If you had 7 more days, what would you build?
5. The 2026 ML job market is competitive. Why should a hiring manager pick you?

If you can answer #5 cleanly in 90 seconds, you're ready.

---

→ **The bootcamp ends here. The work begins.** Apply to roles tomorrow.
