# Interview Prep — Reference Card

Used heavily on Day 14. Print this. Stick it on the wall.

## The ML Engineer Interview Loop (typical)

| Round | What they really test | Your prep |
|---|---|---|
| 1. Recruiter screen | Communication, salary alignment | Have a 90-second story ready |
| 2. ML coding | Vectorized NumPy/PyTorch from scratch | See "Coding drills" below |
| 3. Stats/ML breadth | Bias-variance, regularization, evaluation | See "Concept cards" |
| 4. ML system design | Can you scope, scale, and evaluate? | See "System design template" |
| 5. Behavioral | Conflict, ownership, failure | STAR stories: 3 prepared |

## Coding Drills (15 min each, do all on Day 14 morning)

These come up in ~80% of ML coding rounds. No imports allowed beyond NumPy or PyTorch.

1. **Vectorized softmax** with numerical stability (subtract max).
2. **Cross-entropy loss** from logits (no `nn.CrossEntropyLoss`).
3. **K-means** in pure NumPy, ≤ 30 lines, 10 iterations on random data.
4. **Gradient descent for linear regression** — closed form vs gradient version, plot loss.
5. **k-NN classifier** — vectorized pairwise distance (use `(a-b)^2 = a^2 + b^2 - 2ab`).
6. **AUC-ROC** from scratch from (y_true, y_score) — sort + cumulative sums.
7. **Precision@k / Recall@k** for a ranking output.
8. **Sliding-window batched data loader** in NumPy (time series).
9. **Attention forward pass** — `softmax(QK^T / sqrt(d_k)) V` — for one head and multi-head.
10. **Trie or BPE step** — only if you're applying to NLP-heavy roles.

For each drill: write it, test it on a 5-line input, then ask Claude to review with this prompt:

```
Review code/interview/drill_<N>.py. Check for:
1. Vectorization (any Python-level loops over N or D should be flagged)
2. Numerical stability
3. Off-by-one errors
4. What would a senior reviewer at FAANG say? Be ruthless.
```

## Concept Cards (memorize, don't paraphrase)

**Bias–variance.** High bias = underfit, model too simple. High variance = overfit, model too sensitive to training set. Increasing model capacity decreases bias but increases variance. Regularization, more data, and ensembling reduce variance. Cross-validation is how you measure variance.

**Regularization.** L1 (Lasso) creates sparsity → feature selection. L2 (Ridge) shrinks coefficients toward zero. ElasticNet combines them. In neural nets: weight decay (= L2 on weights), dropout (random zero-ing of activations), data augmentation, early stopping.

**Train/val/test split.** Always 3-way. Use val for hyperparameter selection, test for final reporting. For time series: time-based split, never random. For groups (e.g. multiple records per user): GroupKFold.

**Cross-validation.** Stratified K-Fold for classification. TimeSeriesSplit for temporal. GroupKFold for clustered data. Nested CV when you're also tuning hyperparameters.

**Imbalanced classes.** Don't rely on accuracy. Use PR-AUC, F1, or recall@k depending on cost. Solutions: class weights, threshold tuning, resampling (SMOTE is fine but apply only on train fold). Never resample test.

**Calibration.** A classifier outputting 0.9 should be right 90% of the time. Check with reliability diagrams. Fix with Platt scaling (sigmoid on top of scores) or isotonic regression.

**Cross-entropy vs MSE.** Use cross-entropy for classification (gradient is `p - y`, large when wrong). MSE has vanishing gradient at saturation for sigmoid output.

**Batch normalization.** Normalizes pre-activation across the batch, learns shift+scale. Stabilizes training, acts as regularization. At inference uses running mean/var. Conflicts with small batches → use GroupNorm or LayerNorm instead.

**Dropout.** Randomly zeros activations with probability p at train time. At inference, no dropout but scale up by 1/(1-p) (already handled by PyTorch).

**Learning rate.** Single most important hyperparameter. Too high → diverge. Too low → slow + stuck in saddle. Schedule (cosine, one-cycle) > constant. LR finder: increase LR exponentially, plot loss, pick LR just before loss explodes.

**Optimizer choice.** Adam default. AdamW (with decoupled weight decay) for transformers. SGD+momentum for CNN-from-scratch when you have time to tune (often beats Adam at convergence).

**Attention.** `softmax(QK^T / sqrt(d_k)) V`. Q, K, V are linear projections of the same input (self-attention) or different inputs (cross-attention). Multi-head splits Q/K/V into `h` heads, runs attention in parallel, concatenates, and projects.

**Transformer block.** LayerNorm → MultiHeadAttention → residual → LayerNorm → FFN (2 linear layers with GELU) → residual. Repeat N times.

**Pre-norm vs post-norm.** Modern transformers use pre-norm (LN before sub-layer) — easier to train deep.

**Positional encoding.** Original: sinusoidal. Modern (LLaMA, GPT): RoPE (rotary). Allows extrapolation to longer sequences than seen in training.

**KV cache.** At inference, cache K and V for past tokens so decoding step is O(1) in past length instead of O(n²). Memory grows linearly with sequence length.

**LoRA.** Fine-tune only `W + αBA` where `A ∈ R^(d×r)`, `B ∈ R^(r×d)`, `r << d`. Reduces trainable params by 100–1000x with near-equal performance.

**RAG.** Retrieve top-k chunks from vector store with semantic similarity, stuff into LLM prompt as context. Eval: faithfulness, answer relevance, context relevance. Add re-rankers for accuracy.

**Embeddings.** Vectors that capture semantic similarity. Modern: BGE, E5, Cohere v3, OpenAI text-embedding-3. Use cosine similarity. Normalize first.

**Evaluation metrics matrix:**

| Task | Primary | Secondary |
|---|---|---|
| Binary classification, balanced | AUC-ROC | Accuracy |
| Binary classification, imbalanced | PR-AUC | Recall@k, F1 |
| Multi-class | Macro F1 | Confusion matrix |
| Regression | RMSE | MAE, R² |
| Ranking | NDCG@k | MRR, Recall@k |
| LLM generation | Task-specific | Human eval, LLM-judge |

## System Design Template

Given a prompt like "Design a fraud detection system for X," walk through:

1. **Clarify scope (3 min).**
   - Volume? (req/sec, daily transactions)
   - Latency budget? (real-time vs batch)
   - False-positive vs false-negative cost?
   - Online vs offline learning?
   - Existing data? Labeled or not?
2. **Define the ML problem (2 min).**
   - Inputs (features), outputs (binary? score? top-k?)
   - Training data source, ground truth signal
   - Granularity (per-transaction, per-session, per-user)
3. **Data pipeline (5 min).**
   - Ingestion (Kafka? Batch ETL?)
   - Feature store (online + offline; mention Feast or in-house)
   - Labeling lag (when do you know it was fraud?)
4. **Model (5 min).**
   - Baseline (logistic regression). Why start here.
   - Production candidate (XGBoost / two-tower / fine-tuned transformer)
   - Cold start strategy
5. **Serving (5 min).**
   - Online inference: low-latency REST / gRPC, model in memory, feature lookup
   - Caching: features and predictions
   - Fallbacks: cached prediction, default rule
6. **Evaluation (5 min).**
   - Offline metric (PR-AUC on holdout)
   - Online metric (action rate, $ saved)
   - A/B test design: traffic split, duration, primary metric, guardrails
7. **Monitoring (3 min).**
   - Prediction distribution drift
   - Feature drift (PSI, KS test)
   - Latency p50/p99
   - Retraining cadence + triggers
8. **Failure modes (2 min).**
   - Adversaries adapting (fraudsters)
   - Feedback loop (block legit users → no future label)
   - Data corruption upstream

Print the bold headers. Use them as your spoken outline.

## STAR stories — prepare 3 (45 min on Day 14)

For each, write 4 bullets: **S**ituation, **T**ask, **A**ction, **R**esult.

1. **A technical decision you owned.** Show judgment, trade-offs, defending your choice.
2. **A failure or bug that taught you something.** Show humility + learning loop.
3. **A teammate disagreement.** Show conflict resolution + outcome.

Each should be 90 seconds spoken. Time yourself.

## Behavioral red flags to avoid

- "I" vs "we": use "I" when describing your action. "We" hides your contribution.
- Avoid generic adjectives ("complex," "challenging") without numbers. Replace with metrics: "10x latency reduction from 800ms p99 to 80ms."
- Never trash a former employer.
- Never say "I don't know how to do that." Say "I haven't done X but here's how I'd approach it."

## Salary conversation

When asked for expectations: "I'm calibrating against the market. Based on my research, the band for this role at this level seems to be X to Y in this region. I'm flexible on the specifics — comp, equity, ramp — and would love to hear what the role looks like from your side first."

Don't anchor low. Don't anchor high without a competing offer.

## The 24 hours before an interview

- Re-read the company's most recent ML blog post.
- Re-read the JD. Note 3 keywords from it that match your projects. Plan to drop those keywords.
- Sleep. Don't cram concept cards past 9pm.
- Run a 5-min coding warmup the morning of (vectorized softmax → AUC from scratch).

## The hour after an interview

Write down: questions asked, what you answered well, what you fumbled. This is gold for the next loop.
