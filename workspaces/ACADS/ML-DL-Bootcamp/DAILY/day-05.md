# Day 5 — Experiment Tracking & Hyperparameter Optimization

**Industry context:** Untracked experiments are unfindable experiments. Within 3 days, you'll forget which set of hyperparameters produced which result. W&B + Optuna together solve this — they are *the* combo used by every modern ML team. Today you build the habit that separates hobby-grade from production-grade work.

**Objective by EOD:**
- Log every Day-4 experiment to a W&B project.
- Run a 30-trial Optuna sweep that beats your Day-4 hand-tuned model.
- Produce a written analysis of the best 3 trials and what they tell you.
- Save a `best_params.json` artifact that any future colleague could load and run.

---

## 09:00–10:00 — Theory

**Read:**
1. W&B Quickstart: https://docs.wandb.ai/quickstart (5 min). Then *Tracking experiments*: https://docs.wandb.ai/guides/track (skim, ~15 min).
2. W&B Sweeps overview: https://docs.wandb.ai/guides/sweeps (just the "concepts" page).
3. Optuna in 30 seconds + Key Features: https://optuna.readthedocs.io/en/stable/index.html (~10 min).
4. Optuna *Pythonic Search Space*: https://optuna.readthedocs.io/en/stable/tutorial/10_key_features/002_configurations.html.

**Concept to internalize:** Optuna's TPE (Tree-structured Parzen Estimator) sampler is *not* random search and *not* grid search. It builds a probabilistic model of "good" vs "bad" hyperparameters from past trials and samples the next trial from where the ratio is highest. For ≥20 trials, TPE substantially outperforms random search on most problems.

---

## 10:00–13:00 — Build A: W&B integration

Create `code/day05/sweep.py`:

```python
"""Day 5 — Optuna sweep + W&B tracking for Home Credit XGBoost."""
from __future__ import annotations
import json
import logging
import os
from pathlib import Path

import numpy as np
import optuna
import pandas as pd
import wandb
import xgboost as xgb
from dotenv import load_dotenv
from optuna.integration import WeightsAndBiasesCallback
from sklearn.metrics import roc_auc_score
from sklearn.model_selection import StratifiedKFold, train_test_split

load_dotenv()  # picks up WANDB_API_KEY from .env
ART = Path("artifacts/day05"); ART.mkdir(parents=True, exist_ok=True)
PROJECT = "bootcamp"
SWEEP_NAME = "day05-home-credit-xgb"
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day05")
RNG = 42


def load_data():
    df = pd.read_csv("data/raw/day04/application_train.csv")
    df["DAYS_EMPLOYED"] = df["DAYS_EMPLOYED"].replace(365243, np.nan)
    for col in ["DAYS_BIRTH", "DAYS_EMPLOYED", "DAYS_REGISTRATION", "DAYS_ID_PUBLISH"]:
        df[col] = -df[col] / 365.25
    y = df["TARGET"].astype(int); df = df.drop(columns=["TARGET", "SK_ID_CURR"])
    for c in df.select_dtypes(include="object").columns:
        df[c] = df[c].astype("category")
    return df, y


def cv_score(params: dict, X, y, n_splits: int = 3) -> float:
    """3-fold stratified CV AUC. Returns mean."""
    cv = StratifiedKFold(n_splits, shuffle=True, random_state=RNG)
    aucs = []
    for fold_idx, (tr, va) in enumerate(cv.split(X, y)):
        model = xgb.XGBClassifier(
            **params,
            scale_pos_weight=(y.iloc[tr] == 0).sum() / max((y.iloc[tr] == 1).sum(), 1),
            eval_metric="auc",
            enable_categorical=True,
            tree_method="hist",
            early_stopping_rounds=30,
            n_jobs=-1,
            random_state=RNG,
        )
        model.fit(X.iloc[tr], y.iloc[tr],
                  eval_set=[(X.iloc[va], y.iloc[va])], verbose=False)
        p = model.predict_proba(X.iloc[va])[:, 1]
        aucs.append(roc_auc_score(y.iloc[va], p))
    return float(np.mean(aucs))


def objective_factory(X, y):
    def objective(trial: optuna.Trial) -> float:
        params = {
            "n_estimators": trial.suggest_int("n_estimators", 200, 1500, step=100),
            "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.2, log=True),
            "max_depth": trial.suggest_int("max_depth", 3, 10),
            "min_child_weight": trial.suggest_int("min_child_weight", 1, 50, log=True),
            "subsample": trial.suggest_float("subsample", 0.5, 1.0),
            "colsample_bytree": trial.suggest_float("colsample_bytree", 0.5, 1.0),
            "reg_alpha": trial.suggest_float("reg_alpha", 1e-3, 10.0, log=True),
            "reg_lambda": trial.suggest_float("reg_lambda", 1e-3, 10.0, log=True),
        }
        auc = cv_score(params, X, y)
        # Optional: log per-trial extra info to W&B via the callback
        return auc
    return objective


def main():
    X, y = load_data()
    X_tr, X_te, y_tr, y_te = train_test_split(
        X, y, test_size=0.2, stratify=y, random_state=RNG)

    wandb_kwargs = dict(project=PROJECT, group=SWEEP_NAME, reinit=True)
    wandbc = WeightsAndBiasesCallback(metric_name="cv_auc", wandb_kwargs=wandb_kwargs)

    sampler = optuna.samplers.TPESampler(seed=RNG, multivariate=True)
    study = optuna.create_study(direction="maximize",
                                study_name=SWEEP_NAME,
                                sampler=sampler,
                                storage=f"sqlite:///{ART}/optuna.db",
                                load_if_exists=True)
    study.optimize(objective_factory(X_tr, y_tr), n_trials=30, callbacks=[wandbc])

    log.info(f"Best CV AUC: {study.best_value:.4f}")
    log.info(f"Best params: {study.best_params}")
    (ART / "best_params.json").write_text(json.dumps(study.best_params, indent=2))

    # Final fit on full train, evaluate on test
    best = study.best_params.copy()
    best.update(scale_pos_weight=(y_tr == 0).sum() / (y_tr == 1).sum(),
                eval_metric="auc", enable_categorical=True,
                tree_method="hist", n_jobs=-1, random_state=RNG)
    final_model = xgb.XGBClassifier(**best, early_stopping_rounds=50)
    final_model.fit(X_tr, y_tr, eval_set=[(X_te, y_te)], verbose=False)
    p = final_model.predict_proba(X_te)[:, 1]
    test_auc = roc_auc_score(y_te, p)
    log.info(f"FINAL TEST AUC: {test_auc:.4f}")

    final_model.save_model(str(ART / "best_xgb.json"))
    # Log a final summary run to W&B
    with wandb.init(project=PROJECT, name=f"{SWEEP_NAME}-final", group=SWEEP_NAME) as run:
        run.log({"test_auc": test_auc, "best_cv_auc": study.best_value})
        run.config.update(study.best_params)


if __name__ == "__main__":
    main()
```

Run: `uv run python code/day05/sweep.py`

It will take ~60–90 minutes for 30 trials. Use the time to:
- Open https://wandb.ai/<your-handle>/bootcamp and watch trials populate in real time.
- Browse the *Parallel Coordinates* view (W&B → your run → "+ Add panel" → Parallel coordinates).
- Switch to *Parameter Importance* view.

**Acceptance criteria:**
- 30 trials complete, logged to W&B.
- Final test AUC ≥ Day-4's AUC. The bar.
- `artifacts/day05/best_params.json` exists.

---

## 13:00–14:00 — Lunch + Day-4 Quiz

Grade with Claude.

---

## 14:00–17:00 — Build B: analysis & study

While trials run, work on `code/day05/analyze.py`:

```python
"""Pull insights from completed Optuna study."""
import json
from pathlib import Path
import optuna
import pandas as pd

ART = Path("artifacts/day05")
study = optuna.load_study(study_name="day05-home-credit-xgb",
                          storage=f"sqlite:///{ART}/optuna.db")

# Top 5 trials
df = study.trials_dataframe()
df = df.sort_values("value", ascending=False).head(5)
print("\n=== Top 5 trials ===")
print(df[["number", "value"] + [c for c in df.columns if c.startswith("params_")]])

# Param importance — Optuna's fANOVA-based estimator
importance = optuna.importance.get_param_importances(study)
print("\n=== Parameter importance ===")
for k, v in sorted(importance.items(), key=lambda kv: -kv[1]):
    print(f"  {k:25s} {v:.3f}")

# Did the search saturate? Check best-so-far over time
best_so_far = []
running_best = -1
for t in study.trials:
    if t.value is not None:
        running_best = max(running_best, t.value)
    best_so_far.append(running_best)

import matplotlib.pyplot as plt
plt.plot(best_so_far)
plt.xlabel("Trial #"); plt.ylabel("Best CV AUC so far")
plt.title("Optuna convergence")
plt.savefig(ART / "convergence.png", dpi=120, bbox_inches="tight")
print(f"\nConvergence plot → {ART/'convergence.png'}")
```

Run: `uv run python code/day05/analyze.py`

Now write a 3-paragraph note in `code/day05/NOTES.md` covering:

1. **Top 3 trials.** What did they have in common? What differed? Did Optuna find a clear optimum or did several distinct configs score similarly?
2. **Most important params.** Which 3 hyperparameters most affected CV AUC? Is this consistent with general XGBoost wisdom (LR + n_estimators + reg_lambda are usually the top 3)?
3. **Did the search saturate?** Look at convergence plot. If the curve is still climbing at trial 30, you should run more trials in real life. If it flatlined at trial 20, you'd stop early.

**Acceptance criteria:**
- `NOTES.md` exists with the 3 paragraphs.
- `convergence.png` saved.
- W&B project has a sweep group with ≥30 runs visible.

---

## 17:00–18:00 — Code Review

```bash
git add code/day05/ artifacts/day05/
git commit -m "day-5: Optuna sweep + W&B tracking"
```

Prompt:

```
Review code/day05/. You are a Staff ML Engineer who's lived through 100 hyperparameter sweeps.

1. Search space — are my ranges reasonable, or did I clip the optimum at the edge of a range?
2. CV folds — I used 3 for speed. When is that enough vs 5 or 10? What did I sacrifice?
3. Early stopping inside CV — is it leaking? (Yes. Convince me why and how to fix.)
4. Optuna's TPE sampler — when does random search beat it? Was 30 trials enough?
5. W&B usage — am I logging at the right granularity? What else should be logged
   (artifacts, configs, system metrics)?
6. The final fit re-uses early stopping on test — this peeks at test. Walk me through
   the correct three-way split workflow for an "interview-defensible" answer.

End with 3 ways to make this study reproducible by a teammate who has only this repo.
```

Save to `reviews/day05-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Pruning.** Add an Optuna pruner (`optuna.pruners.MedianPruner`) and instrument the CV loop to `trial.report()` per fold + `trial.should_prune()`. Re-run 30 trials. Did wall-clock drop without quality loss?
- **B: Sweep config YAML.** Write a `sweep.yaml` that defines the same search space as W&B native sweeps. Launch via `wandb sweep sweep.yaml` and `wandb agent <id>`. Compare with Optuna.
- **C: Multi-objective.** Optuna supports multi-objective (return a tuple from the objective). Optimize for (CV AUC, training time) — `study = optuna.create_study(directions=["maximize", "minimize"])`. Plot the Pareto front.
- **D: Read Bergstra & Bengio (2012) "Random Search for Hyper-parameter Optimization" — just sections 1, 2, 4.** It's the foundational paper showing random search > grid search. 6 pages.
- **E: Write `make_run_card.py`** that, given a study and final test result, produces a clean Markdown "model card" you'd attach to a PR.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Using `RandomizedSearchCV` for 100+ trials when Optuna would converge in 30.
- **Pitfall 2:** Setting `n_estimators` as a hyperparameter alongside `early_stopping_rounds` — they conflict. Either fix `n_estimators` high and let early stopping decide, OR tune `n_estimators` without early stopping. Not both.
- **Pitfall 3:** Not seeding the sampler — your sweep is irreproducible.
- **Pitfall 4:** Searching `learning_rate` on a linear scale. Always log-scale for rate-like params.
- **Pitfall 5:** Failing to differentiate CV best from test best — your CV-best params will overestimate true test performance (winner's curse). Nested CV or a held-out test set is essential.

---

## Quiz (tomorrow 13:00)

`code/day05/quiz_answers.md`.

1. What does TPE stand for and in one sentence, how is it different from random search?
2. Why log-scale `learning_rate` but linear-scale `subsample`?
3. You ran 30 trials, best CV AUC went 0.74 → 0.755. You ran 200 trials, best became 0.760. Was the extra 170 trials worth it? When?
4. What is the "winner's curse" in hyperparameter optimization?
5. Three-way split (train/val/test) — what does each set actually validate?
6. If you have 1M rows, why is 3-fold CV more defensible than 10-fold?
7. Your CV AUC has std 0.012 across 5 folds. Two configs differ by 0.003 in mean. Is the difference significant?
8. When would you replace Optuna with a Bayesian optimizer like BoTorch or Ax?

---

→ Tomorrow at 09:00: open `DAILY/day-06.md`. **Today was the last classical ML day. From tomorrow it's PyTorch.**
