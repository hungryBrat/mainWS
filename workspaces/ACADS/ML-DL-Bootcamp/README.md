# ML/DL Bootcamp — 14 Days to Industry-Ready

This is a sealed, high-intensity bootcamp. It is not a buffet of options. It is a march.
You will follow it day-by-day, block-by-block. No substitutions, no procrastination, no decision fatigue.

## The Rules

1. **One task at a time.** Open today's daily file. Do not read ahead.
2. **Time blocks are commitments, not suggestions.** When the block ends, you stop and move to the next, even if incomplete. Mark incomplete items as `TODO-overflow` in the journal and revisit only on Sunday review.
3. **No optional reading.** If a resource is listed, you read/watch it. If it isn't, you don't go hunting.
4. **Code goes in `code/dayXX/`.** Every day has its own subdirectory. Commit at the end of each block (`git commit -m "day-N: <block>"`).
5. **End-of-day code review is mandatory.** Run the exact prompt at the end of each daily file. Save the review output to `reviews/dayXX-review.md`.
6. **Quizzes are oral.** During lunch the next day, run the quiz prompt and answer aloud or type. No googling. Score yourself honestly.
7. **The journal is your accountability ledger.** End every day with the journal entry in `journal.md` — 5 minutes max, the template is below.
8. **Skip nothing. Add nothing.** The plan is calibrated. Padding kills it.

## Stack — locked, no debate

| Layer | Tool | Why |
|---|---|---|
| Language | Python 3.11 | Industry default for ML |
| Env mgmt | `uv` | Fastest, modern, replaces pip+venv+poetry |
| Data | pandas 2.x, polars | pandas for muscle memory, polars when speed matters |
| Classical ML | scikit-learn 1.5+ | The interview language |
| Gradient boosting | XGBoost + LightGBM | What 70% of tabular prod jobs use |
| Deep learning | PyTorch 2.5+ | Industry default, period. No TensorFlow. |
| Transformers | HuggingFace `transformers` + `datasets` + `peft` | The ecosystem |
| Tracking | Weights & Biases (free tier) | Industry standard, beautiful UI |
| Tuning | Optuna | TPE sampler, async, integrates with W&B |
| Vector store | FAISS | In-process, fast, no infra |
| Embeddings | `sentence-transformers` | Local, free, strong baseline |
| LLM API | Anthropic Claude (Sonnet) | You already have access |
| Serving | FastAPI + uvicorn | Async, type-safe, prod-grade |
| Containers | Docker (with `nvidia-docker` if GPU) | Universal |
| Notebooks | Jupyter Lab | Exploration only — production is `.py` files |
| Linting | ruff + black | Fast, opinionated |
| Testing | pytest | Standard |

If a daily file says "use X", you use X. If you're tempted to substitute, re-read rule 3.

## Hardware Reality Check

Most of the bootcamp runs on CPU. The GPU days are 7, 8, 9, 10, 11. If you don't have a local NVIDIA GPU:

- **Free option:** Google Colab (free T4, ~12 hrs/day). Save checkpoints to Google Drive.
- **Cheap reliable option:** Kaggle Notebooks (free P100, 30 hrs/week, no time pressure).
- **Paid:** Lambda Labs on-demand A10 (~$0.75/hr) or RunPod.

Default: **Kaggle Notebooks**. Decision made. Move on.

## Daily Structure (the 10-hour day)

| Block | Time | Purpose |
|---|---|---|
| Theory | 09:00–10:00 | Read/watch only what's listed |
| Build A | 10:00–13:00 | First half of the day's project |
| Lunch + Quiz | 13:00–14:00 | Eat, run yesterday's quiz |
| Build B | 14:00–17:00 | Finish the build, hit acceptance criteria |
| Review | 17:00–18:00 | Run the code review prompt with Claude |
| Dinner | 18:00–19:00 | Step away from the screen |
| Hardening | 19:00–21:00 | Stretch goals, polish, write tests |
| Journal | 21:00–21:30 | Daily journal entry |

This adds up to a 12.5-hour gross day with 2.5 hours of breaks → **10 hours of focused work**. Sustain this for 14 days.

## Journal Template (paste into `journal.md` every night)

```markdown
## Day NN — YYYY-MM-DD

**Acceptance criteria met:** Y / N (which ones not?)
**Hours of focused work:** ~N
**Hardest 30 min:** ...
**One concept that clicked:** ...
**One concept still murky:** ...   ← bring to tomorrow's theory block
**Overflow TODOs:** ...
**Tomorrow at 09:00 I open:** DAILY/day-NN+1.md
```

## How to use Claude during the bootcamp

You have Claude Code. Use it as your TA, not your ghostwriter.

- **Allowed:** asking Claude to explain a concept, debug a stack trace, review your code, generate test cases, quiz you.
- **Not allowed:** asking Claude to write the day's project for you. You'll feel productive and learn nothing. The grading rubric for each day assumes you wrote it.
- **End-of-day code review:** every daily file gives you a prompt. Paste it into a fresh Claude conversation pointed at your `code/dayXX/` folder. Save the response to `reviews/dayXX-review.md`. Read it carefully — that's where the real learning compounds.

## Order of Operations

1. Read this file once. Now.
2. Run `SETUP.md` end-to-end. This is **Day 0** — do it tonight or tomorrow morning before Day 1.
3. Open `BOOTCAMP.md` for the 14-day overview.
4. From there, open `DAILY/day-01.md` and follow it block by block.
5. At end of Week 1 (after Day 7), do the mid-bootcamp checkpoint in `BOOTCAMP.md`.
6. At end of Day 14, do the capstone wrap-up and update your GitHub.

Your job depends on this. Treat it accordingly.
