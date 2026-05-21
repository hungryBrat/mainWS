# Day 10 — Hugging Face: Fine-Tune DistilBERT + LoRA

**Industry context:** Whatever NLP problem your future employer has — sentiment, classification, NER, semantic search — your starting point will be a HuggingFace model. The `transformers` + `datasets` + `peft` libraries are *the* ecosystem. Today you fine-tune a real BERT-class model two ways: full fine-tune (the brute method) and LoRA (the modern parameter-efficient method).

**Objective by EOD:**
- Fine-tune DistilBERT on SST-2 (binary sentiment) using HF `Trainer`.
- Re-do it with LoRA via `peft` and compare quality vs trainable param count.
- Push your fine-tuned model to the HuggingFace Hub.
- **Target:** ≥90% accuracy on SST-2 validation.

**Dataset (locked):** GLUE — SST-2 (Stanford Sentiment Treebank, 67k train / 872 val).

```python
from datasets import load_dataset
ds = load_dataset("glue", "sst2")
```

The validation set is the eval target (the test set's labels are hidden — GLUE leaderboard).

---

## 09:00–10:00 — Theory

**Read:**
1. HuggingFace NLP Course — **Chapter 1 (only sections 1, 4, 6) + Chapter 3 (whole, ~25 min)**: https://huggingface.co/learn/nlp-course/chapter1.
2. *LoRA: Low-Rank Adaptation of Large Language Models* (Hu et al. 2021) — read **Abstract + Section 4 + Section 5.1**. https://arxiv.org/abs/2106.09685.

**LoRA in one paragraph:** Instead of updating `W ∈ R^{d×d}`, update `W + ΔW` where `ΔW = αBA`, `A ∈ R^{d×r}`, `B ∈ R^{r×d}`, and `r ≪ d` (typically 8–32). You train only `A` and `B` (≈ 0.1–1% of params), freeze the rest. At inference you can either keep them separate (one base model, many LoRA adapters) or merge: `W_new = W + αBA`.

Industry uses LoRA because:
- Trainable params reduced ~100x → fits much larger models on the same GPU.
- Same task quality (often within 0.5% of full fine-tune).
- Adapters are tiny (~MBs) → easy to ship/swap.
- No serving cost if merged at deploy time.

---

## 10:00–13:00 — Build A: full fine-tune with Trainer

Create `code/day10/finetune_full.py`:

```python
"""Day 10 — Full fine-tune of DistilBERT on SST-2."""
import logging
from pathlib import Path

import evaluate
import numpy as np
from datasets import load_dataset
from transformers import (
    AutoModelForSequenceClassification, AutoTokenizer,
    DataCollatorWithPadding, Trainer, TrainingArguments,
)

ART = Path("artifacts/day10/full"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day10")

MODEL = "distilbert-base-uncased"
NUM_LABELS = 2

tokenizer = AutoTokenizer.from_pretrained(MODEL)

def tokenize(batch):
    return tokenizer(batch["sentence"], truncation=True, max_length=128)

ds = load_dataset("glue", "sst2")
ds = ds.map(tokenize, batched=True)
ds = ds.rename_column("label", "labels")
ds.set_format("torch", columns=["input_ids", "attention_mask", "labels"])

log.info(f"Train: {len(ds['train']):,}  Val: {len(ds['validation']):,}")

model = AutoModelForSequenceClassification.from_pretrained(MODEL, num_labels=NUM_LABELS)
collator = DataCollatorWithPadding(tokenizer)

acc_metric = evaluate.load("accuracy")
f1_metric = evaluate.load("f1")

def compute_metrics(p):
    preds = p.predictions.argmax(-1)
    return {**acc_metric.compute(predictions=preds, references=p.label_ids),
            **f1_metric.compute(predictions=preds, references=p.label_ids)}

args = TrainingArguments(
    output_dir=str(ART),
    num_train_epochs=3,
    per_device_train_batch_size=32,
    per_device_eval_batch_size=64,
    learning_rate=2e-5,
    weight_decay=0.01,
    warmup_ratio=0.1,
    eval_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    logging_steps=100,
    fp16=True,
    report_to="wandb",
    run_name="day10-distilbert-full",
)

trainer = Trainer(
    model=model, args=args,
    train_dataset=ds["train"], eval_dataset=ds["validation"],
    tokenizer=tokenizer, data_collator=collator,
    compute_metrics=compute_metrics,
)
trainer.train()
metrics = trainer.evaluate()
log.info(f"Final eval: {metrics}")
trainer.save_model(str(ART / "best"))
```

Run on GPU:

```bash
uv run python code/day10/finetune_full.py
```

Time: ~15–20 min on T4. Expected validation accuracy: 90–91%.

**Acceptance criteria:**
- Training completes 3 epochs.
- Final eval accuracy ≥ 90%.
- Model saved to `artifacts/day10/full/best/`.
- W&B run "day10-distilbert-full" exists.

---

## 13:00–14:00 — Lunch + Day-9 Quiz

---

## 14:00–17:00 — Build B: LoRA fine-tune + comparison

Create `code/day10/finetune_lora.py`:

```python
"""Day 10 — LoRA fine-tune of DistilBERT on SST-2."""
import logging
from pathlib import Path

import evaluate
import numpy as np
import torch
from datasets import load_dataset
from peft import LoraConfig, TaskType, get_peft_model
from transformers import (
    AutoModelForSequenceClassification, AutoTokenizer,
    DataCollatorWithPadding, Trainer, TrainingArguments,
)

ART = Path("artifacts/day10/lora"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day10")

MODEL = "distilbert-base-uncased"
NUM_LABELS = 2

tokenizer = AutoTokenizer.from_pretrained(MODEL)
ds = load_dataset("glue", "sst2")
ds = ds.map(lambda b: tokenizer(b["sentence"], truncation=True, max_length=128), batched=True)
ds = ds.rename_column("label", "labels")
ds.set_format("torch", columns=["input_ids", "attention_mask", "labels"])

base = AutoModelForSequenceClassification.from_pretrained(MODEL, num_labels=NUM_LABELS)

# LoRA config: target attention projections only
lora_cfg = LoraConfig(
    task_type=TaskType.SEQ_CLS,
    r=8,                                    # rank
    lora_alpha=16,
    lora_dropout=0.1,
    target_modules=["q_lin", "v_lin"],      # DistilBERT attention proj names
    modules_to_save=["classifier"],         # train the new classification head too
)
model = get_peft_model(base, lora_cfg)
model.print_trainable_parameters()

collator = DataCollatorWithPadding(tokenizer)
acc = evaluate.load("accuracy")

args = TrainingArguments(
    output_dir=str(ART),
    num_train_epochs=3,
    per_device_train_batch_size=32,
    per_device_eval_batch_size=64,
    learning_rate=1e-3,                     # LoRA tolerates higher LR than full FT
    warmup_ratio=0.06,
    weight_decay=0.0,
    eval_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    logging_steps=100,
    fp16=True,
    report_to="wandb",
    run_name="day10-distilbert-lora",
)

trainer = Trainer(
    model=model, args=args,
    train_dataset=ds["train"], eval_dataset=ds["validation"],
    tokenizer=tokenizer, data_collator=collator,
    compute_metrics=lambda p: acc.compute(predictions=p.predictions.argmax(-1), references=p.label_ids),
)
trainer.train()
log.info(trainer.evaluate())
trainer.save_model(str(ART / "best"))   # saves adapter only (~3MB!)
```

Run: `uv run python code/day10/finetune_lora.py`

You should see `model.print_trainable_parameters()` print something like:

```
trainable params: 738,818 || all params: 67,693,058 || trainable%: 1.09
```

**Acceptance criteria:**
- Training completes 3 epochs.
- LoRA accuracy ≥ 89% (typical: 89–90.5, vs 90–91 for full).
- Adapter saved (~3MB only).
- W&B has the second run.

### Comparison table

In `code/day10/COMPARISON.md`:

```markdown
| Method | Trainable params | Train time | Val accuracy | Disk |
|---|---|---|---|---|
| Full fine-tune | ~67M | X min | X.XX | ~270MB |
| LoRA (r=8) | ~0.74M | X min | X.XX | ~3MB |
```

Industry takeaway: **LoRA is the default for any model > 100M parameters.** Full fine-tune is for under-100M models or when last 0.5% accuracy matters.

---

## 17:00–18:00 — Inference + push to Hub

### Inference

`code/day10/predict.py`:

```python
"""Use both fine-tuned models for inference."""
from pathlib import Path
import torch
from peft import PeftModel
from transformers import AutoModelForSequenceClassification, AutoTokenizer

ART = Path("artifacts/day10")
tok = AutoTokenizer.from_pretrained("distilbert-base-uncased")

# Full FT
full_model = AutoModelForSequenceClassification.from_pretrained(ART / "full/best").eval()

# LoRA (load base + adapter)
base = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased", num_labels=2)
lora_model = PeftModel.from_pretrained(base, ART / "lora/best").eval()

samples = [
    "This movie absolutely blew me away — best film I've seen all year.",
    "Boring, predictable, and the acting was terrible.",
    "It was okay. Not great, not bad.",
    "I'd watch it again, but only on a long flight.",
]

def predict(model, text):
    enc = tok(text, return_tensors="pt", truncation=True, max_length=128)
    with torch.no_grad():
        logits = model(**enc).logits
    probs = torch.softmax(logits, dim=-1)[0]
    label = "positive" if probs[1] > probs[0] else "negative"
    return label, probs[1].item()

print(f"{'TEXT':70s}  {'FULL':18s}  {'LORA':18s}")
for s in samples:
    f_lbl, f_p = predict(full_model, s)
    l_lbl, l_p = predict(lora_model, s)
    print(f"{s[:70]:70s}  {f_lbl} ({f_p:.2f})   {l_lbl} ({l_p:.2f})")
```

Run: `uv run python code/day10/predict.py`. Note any disagreements between full and LoRA.

### Push to Hub (15 min)

```bash
uv run huggingface-cli login
```

Paste a HF access token from https://huggingface.co/settings/tokens (write scope).

Then in Python:

```python
from huggingface_hub import HfApi
api = HfApi()
api.create_repo(repo_id="<your-handle>/distilbert-sst2-bootcamp", exist_ok=True)
trainer.push_to_hub("Trained in 14-day ML bootcamp.")
```

Or:

```python
model.push_to_hub("<your-handle>/distilbert-sst2-lora-bootcamp")
tokenizer.push_to_hub("<your-handle>/distilbert-sst2-lora-bootcamp")
```

Visit your model page — it's a real model on the public internet now. Add a model card (`README.md` in the HF repo) with metrics and intended use.

**Acceptance criteria:**
- `predict.py` runs and produces 4 sensible predictions on each model.
- At least one model published to HF Hub.

---

## 18:00 — Code Review (squeezed before dinner today)

```bash
git add code/day10/ artifacts/day10/
git commit -m "day-10: DistilBERT SST-2 — full fine-tune + LoRA + push to Hub"
```

Prompt:

```
Review code/day10/. Be a senior NLP engineer at a startup.

1. The full fine-tune used LR=2e-5 and LoRA used LR=1e-3 — explain in 2 sentences
   why LoRA tolerates 50x higher LR.
2. `target_modules=["q_lin", "v_lin"]` — why these and not q+k+v+output projections?
   When would I add more?
3. The Trainer hides a lot. Walk me through what `load_best_model_at_end=True`
   actually does, including which checkpoint files are involved.
4. SST-2 has class balance ~52/48 — almost balanced. What changes if my real dataset
   is 90/10? (Class weights in Trainer, focal loss, threshold tuning.)
5. fp16 vs bf16 — which would you pick on an A100, and why?
6. The LoRA adapter is 3MB. Walk me through the file structure and how to
   recombine it with the base model at inference.

End: give me a one-paragraph summary I could paste in a slide titled "When to use LoRA."
```

Save to `reviews/day10-review.md`.

---

## 18:30–19:30 — Dinner

(Yes — shifted 30 min today because of the Hub push. Make it up in hardening.)

---

## 19:30–21:00 — Hardening (pick two)

- **A: 8-bit / QLoRA.** Replace base loading with `AutoModelForSequenceClassification.from_pretrained(..., load_in_8bit=True)` (requires `bitsandbytes`). Re-run LoRA. Memory should drop ~50%.
- **B: Inference latency benchmark.** Time both models on 100 samples. Latency per sample on CPU and GPU. The deltas matter for serving.
- **C: Try a different target task.** Switch from SST-2 to AG News (4-class). Verify your script generalizes by changing only `dataset name` + `num_labels`.
- **D: Read** "How to Generate Text" by Patrick von Platen (HF blog) for sampling strategies: https://huggingface.co/blog/how-to-generate. Mostly relevant tomorrow for RAG.
- **E: A model card.** Write `artifacts/day10/MODEL_CARD.md` properly — model, dataset, metrics, biases, intended use, limitations.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Forgetting `tokenizer(..., truncation=True, max_length=N)` → some batches OOM on long sequences.
- **Pitfall 2:** Not setting `modules_to_save=["classifier"]` in LoRA → the new task head isn't trained → 50% accuracy (random).
- **Pitfall 3:** Picking the wrong `target_modules` name for the model — DistilBERT uses `q_lin/v_lin`, BERT uses `query/value`, LLaMA uses `q_proj/v_proj`. `model.print_modules_with_lora()` (or just `print(model)`) reveals the names.
- **Pitfall 4:** Loading the full model when only adapter is needed → wastes 250MB+ at inference. Use `PeftModel.from_pretrained(base, adapter_path)`.
- **Pitfall 5:** Using `Trainer.predict()` on a dataset that still has string columns → `RuntimeError`. Set format to torch and drop unused columns first.

---

## Quiz (tomorrow 13:00)

`code/day10/quiz_answers.md`.

1. Tokenization: what's the difference between WordPiece (BERT) and BPE (GPT)?
2. `attention_mask` — what does it tell the model and what would happen without it?
3. Why does HF `Trainer` `load_best_model_at_end` require `metric_for_best_model`?
4. Why is `weight_decay` typically 0 for LoRA but 0.01 for full FT?
5. LoRA rank `r`: what's the trade-off as you change it from 4 → 16 → 64?
6. `lora_alpha` — what is it scaling, and what's the typical relationship to `r`?
7. You want to fine-tune Llama-3-8B on a 24GB GPU. Walk me through QLoRA — what's quantized, what isn't, what's the memory math?
8. Why is "pre-trained then fine-tuned" so much better than "trained from scratch on your dataset" for NLP?

---

→ Tomorrow at 09:00: open `DAILY/day-11.md`. **RAG day — you build a working retrieval system.**
