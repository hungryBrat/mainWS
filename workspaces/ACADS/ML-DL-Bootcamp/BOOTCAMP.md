# The 14-Day Bootcamp — Master Plan

Two phases. No filler.

## Phase 1: Classical ML + DL Foundations (Days 1–7)

| Day | Title | Outcome (shippable artifact) | Stack |
|---|---|---|---|
| 1 | Data Wrangling Mastery | EDA notebook on NYC Taxi data with 12 chart cells + clean dataframe | pandas, polars, matplotlib |
| 2 | Scikit-Learn Pipelines | Production sklearn pipeline (joblib) for California Housing regression — RMSE under 0.55 on log-target | sklearn |
| 3 | Classification + Imbalance | Fraud detection model with tuned threshold, PR-AUC ≥ 0.78 | sklearn, imbalanced-learn |
| 4 | XGBoost + SHAP | Home Credit subset model with SHAP report — AUC ≥ 0.74 | XGBoost, SHAP |
| 5 | Experiment Tracking + HPO | Optuna sweep (≥30 trials) logged to W&B with a clear best-trial writeup | Optuna, W&B |
| 6 | PyTorch from Scratch | MNIST classifier, training loop hand-written, ≥98% test accuracy | PyTorch |
| 7 | CNNs on CIFAR-10 | ResNet-style CNN, ≥88% test accuracy, augmentation + LR schedule | PyTorch |

**Sunday checkpoint (after Day 7):**
- Re-run the Day-1 quiz, Day-3 quiz, Day-6 quiz. Score ≥80%? If not, identify the lowest-scoring day and spend 1 hour re-doing the muddiest concept tomorrow morning before the Day 8 block starts.
- Push everything to a public GitHub repo `ml-bootcamp-<your-handle>`.

## Phase 2: Deep Learning + Deployment (Days 8–14)

| Day | Title | Outcome | Stack |
|---|---|---|---|
| 8 | Transfer Learning | Fine-tuned ResNet50 on Oxford Flowers 102, ≥92% test accuracy | torchvision |
| 9 | Transformer Internals | nanoGPT-style char-level model on Shakespeare, val loss ≤ 1.5 | PyTorch |
| 10 | HuggingFace Fine-tuning | DistilBERT fine-tuned on SST-2 via Trainer, ≥90% accuracy + LoRA variant | transformers, peft |
| 11 | RAG System | RAG over PyTorch docs with FAISS + Claude Sonnet, evaluation set of 20 Q&A pairs | sentence-transformers, FAISS, anthropic |
| 12 | FastAPI + Docker | Day-8 model deployed as REST endpoint in a Docker container; passes a 50-request load test | FastAPI, Docker |
| 13 | MLOps + Monitoring | Prometheus metrics endpoint, structured logging, basic drift check | prometheus_client, structlog |
| 14 | Capstone + Interview | Polished GitHub portfolio + 10-question ML system design crib sheet | — |

## Decision Log — already made for you

These are the questions you don't have to ask:

- **Q: PyTorch or TensorFlow?** → PyTorch. Always.
- **Q: Conda or pip?** → Neither. `uv`.
- **Q: Jupyter or VSCode?** → Both. Exploration in Jupyter Lab, production in VSCode `.py`.
- **Q: W&B or MLflow?** → W&B. Free tier is more than enough.
- **Q: Kaggle or HuggingFace for datasets?** → Both. Kaggle for tabular, HF Datasets for NLP.
- **Q: Should I learn Keras?** → No.
- **Q: Should I learn DataBricks / Spark?** → Not in these 14 days. Add it later when a job demands it.
- **Q: Streamlit / Gradio for demos?** → Skip. FastAPI is the prod skill. Add a Streamlit wrapper in Day 14 if time.
- **Q: GPT-4 / Gemini / Claude for LLM days?** → Claude Sonnet. You already have an account.
- **Q: Coursera / fast.ai / Karpathy / 3Blue1Brown?** → Karpathy for transformers (Day 9). Otherwise read the exact links in each daily file.
- **Q: Should I do LeetCode?** → Day 14 only, and only the ML coding flavor (gradient descent from scratch, vectorized softmax, etc.).
- **Q: Cloud — AWS / GCP / Azure?** → Skip cloud certs. Focus on Docker + a hosted demo. Cloud is project-specific.
- **Q: Pandas or polars?** → Both. Day 1 covers both. Default to pandas, reach for polars when >1GB or speed-critical.

## Reading List (the only reading list)

Each day points you to one section. This is the union — don't read ahead.

- Aurélien Géron — *Hands-On Machine Learning, 3rd ed.* — Ch. 2, 3, 6, 7 (Days 1–4)
- Sebastian Raschka — *Build a Large Language Model (From Scratch)* — Ch. 3 (Day 9, supplemental)
- Andrej Karpathy — "Let's build GPT: from scratch, in code, spelled out" (YouTube, Day 9)
- PyTorch docs — "Training a Classifier" tutorial (Day 6 only)
- HuggingFace NLP Course — Chapters 1, 3, 7 sections only (Day 10)
- Chip Huyen — *Designing Machine Learning Systems* — Ch. 7, 8, 9 (Day 12, Day 13)

That's it. Six sources. Anything else is decision fatigue.

## What you will be able to do on Day 15

- Build an end-to-end tabular ML pipeline from raw CSV to served REST API in under a day.
- Fine-tune any vision model with `torchvision` or any NLP model with `transformers`.
- Explain attention to an interviewer without notes.
- Stand up a RAG prototype for a new corpus in two hours.
- Containerize and deploy an ML service with metrics and logging.
- Handle the standard ML system design interview (recommendation, fraud detection, search ranking).

That is the job market floor for an ML engineer in 2026. You'll be at the floor — the next 6 months of real work get you higher.

---

→ Next: Open `SETUP.md` and complete Day 0.
