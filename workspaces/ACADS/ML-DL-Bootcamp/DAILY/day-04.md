# Day 4 — XGBoost & SHAP Explainability

**Industry context:** XGBoost (or LightGBM) wins more tabular Kaggle competitions and production deployments than any other model. SHAP is the only feature-attribution tool stakeholders actually understand. Together they are the workhorse + the communication layer.

**Objective by EOD:**
- Train a tuned XGBoost classifier on a real credit risk dataset.
- Produce SHAP summary, dependence, and force plots.
- Compare LightGBM and XGBoost head-to-head.
- **Target:** ROC-AUC ≥ 0.74 on test set.

**Dataset (locked):** Kaggle — *Home Credit Default Risk* (use only `application_train.csv` for today; the joinable bureau/POS files are Day 4 stretch material).

```bash
mkdir -p data/raw/day04 && cd data/raw/day04
uv run kaggle competitions download -c home-credit-default-risk -f application_train.csv
unzip -o application_train.csv.zip
ls -lh application_train.csv     # ~166MB, 307k rows, 122 cols
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
```

If the competition rules need accepting: https://www.kaggle.com/c/home-credit-default-risk/rules.

---

## 09:00–10:00 — Theory

**Read:**
1. Géron HOML, **Ch. 7 — Ensemble Learning and Random Forests**, pages 191–217 (3rd ed.). Focus on the *gradient boosting* section. Skip the voting classifier deep dive.
2. XGBoost docs — *Introduction to Boosted Trees*: https://xgboost.readthedocs.io/en/stable/tutorials/model.html. Read once, no notes.
3. SHAP README — first half only: https://github.com/shap/shap (just the intro figures and the "Tree ensemble example" section).

**Two ideas to internalize:**
- **Gradient boosting fits residuals.** Each new tree is trained to predict the negative gradient of the loss w.r.t. the current prediction. For squared loss, that gradient is literally the residual; for log-loss, it's a scaled probability error.
- **Shapley values** assign each feature a contribution to a single prediction, averaging over all subsets of features. SHAP just computes them efficiently for trees.

---

## 10:00–13:00 — Build A: data prep + XGBoost baseline

The Home Credit data is gnarly — 122 columns, mixed types, NaNs everywhere. Industry-realistic.

Create `code/day04/train.py`:

```python
"""Day 4 — Home Credit Default Risk with XGBoost + SHAP."""
from __future__ import annotations
import logging
from pathlib import Path

import joblib
import numpy as np
import pandas as pd
import xgboost as xgb
from sklearn.metrics import roc_auc_score, average_precision_score
from sklearn.model_selection import StratifiedKFold, train_test_split

ART = Path("artifacts/day04"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day04")
RNG = 42


def load_and_clean() -> tuple[pd.DataFrame, pd.Series]:
    df = pd.read_csv("data/raw/day04/application_train.csv")
    log.info(f"Loaded {len(df):,} rows × {df.shape[1]} cols")

    # Target
    y = df["TARGET"].astype(int)
    df = df.drop(columns=["TARGET", "SK_ID_CURR"])

    # Replace anomalous DAYS_EMPLOYED sentinel (365243 = "never employed")
    df["DAYS_EMPLOYED"] = df["DAYS_EMPLOYED"].replace(365243, np.nan)
    # Days are stored as negative — convert to positive years for interpretability
    for col in ["DAYS_BIRTH", "DAYS_EMPLOYED", "DAYS_REGISTRATION", "DAYS_ID_PUBLISH"]:
        df[col] = -df[col] / 365.25

    # Encode categoricals — XGBoost can handle pandas Categorical with enable_categorical=True
    cat_cols = df.select_dtypes(include="object").columns
    for c in cat_cols:
        df[c] = df[c].astype("category")
    log.info(f"{len(cat_cols)} categorical features encoded")
    return df, y


def main() -> None:
    X, y = load_and_clean()
    X_tr, X_te, y_tr, y_te = train_test_split(
        X, y, test_size=0.2, stratify=y, random_state=RNG)

    pos_weight = (y_tr == 0).sum() / (y_tr == 1).sum()
    log.info(f"Class imbalance: scale_pos_weight = {pos_weight:.2f}")

    model = xgb.XGBClassifier(
        n_estimators=800,
        learning_rate=0.05,
        max_depth=6,
        min_child_weight=10,
        subsample=0.8,
        colsample_bytree=0.8,
        reg_alpha=0.1,
        reg_lambda=1.0,
        scale_pos_weight=pos_weight,
        eval_metric="auc",
        early_stopping_rounds=50,
        enable_categorical=True,
        tree_method="hist",
        n_jobs=-1,
        random_state=RNG,
    )

    model.fit(X_tr, y_tr, eval_set=[(X_te, y_te)], verbose=50)
    p = model.predict_proba(X_te)[:, 1]
    log.info(f"Test ROC-AUC: {roc_auc_score(y_te, p):.4f}")
    log.info(f"Test PR-AUC : {average_precision_score(y_te, p):.4f}")

    model.save_model(str(ART / "xgb.json"))
    np.savez(ART / "splits.npz",
             X_te_idx=X_te.index.values,
             y_te=y_te.values,
             p=p)
    log.info(f"Saved model → {ART/'xgb.json'}")


if __name__ == "__main__":
    main()
```

Run: `uv run python code/day04/train.py`

Expected ROC-AUC around 0.755–0.77. PR-AUC around 0.22–0.25 (class is 8% positive). Hits the bar.

**Acceptance criteria:**
- Test ROC-AUC ≥ 0.74.
- Model saved as `artifacts/day04/xgb.json`.
- Test set predictions + labels persisted for the SHAP step.

---

## 13:00–14:00 — Lunch + Day-3 Quiz

Run Day 3 quiz. Grade with Claude.

---

## 14:00–17:00 — Build B: SHAP analysis + LightGBM comparison

Create `code/day04/explain.py`:

```python
"""SHAP analysis of the XGBoost model from Day 4."""
import joblib
import numpy as np
import pandas as pd
import shap
import xgboost as xgb
import matplotlib.pyplot as plt
from pathlib import Path

ART = Path("artifacts/day04")

# Reload data + model
df = pd.read_csv("data/raw/day04/application_train.csv")
df["DAYS_EMPLOYED"] = df["DAYS_EMPLOYED"].replace(365243, np.nan)
for col in ["DAYS_BIRTH", "DAYS_EMPLOYED", "DAYS_REGISTRATION", "DAYS_ID_PUBLISH"]:
    df[col] = -df[col] / 365.25
y = df["TARGET"]; df = df.drop(columns=["TARGET", "SK_ID_CURR"])
for c in df.select_dtypes(include="object").columns:
    df[c] = df[c].astype("category")

splits = np.load(ART / "splits.npz", allow_pickle=True)
X_te = df.loc[splits["X_te_idx"]]
y_te = splits["y_te"]; p = splits["p"]

model = xgb.XGBClassifier(enable_categorical=True)
model.load_model(str(ART / "xgb.json"))

# Sample 5000 rows for SHAP (full 60k too slow)
sample = X_te.sample(5000, random_state=42)
explainer = shap.TreeExplainer(model)
sv = explainer(sample)

# 1. Summary plot
shap.summary_plot(sv, sample, max_display=20, show=False)
plt.savefig(ART / "shap_summary.png", dpi=120, bbox_inches="tight")
plt.close()

# 2. Bar plot (mean |SHAP|)
shap.summary_plot(sv, sample, plot_type="bar", max_display=20, show=False)
plt.savefig(ART / "shap_bar.png", dpi=120, bbox_inches="tight")
plt.close()

# 3. Dependence plot for the top-importance feature
top_feat = pd.Series(np.abs(sv.values).mean(axis=0), index=sample.columns).idxmax()
shap.dependence_plot(top_feat, sv.values, sample, show=False)
plt.savefig(ART / f"shap_dep_{top_feat}.png", dpi=120, bbox_inches="tight")
plt.close()

print(f"Top feature: {top_feat}")
```

Run: `uv run python code/day04/explain.py`

Open `artifacts/day04/shap_summary.png`. The top features should include `EXT_SOURCE_1/2/3`, `DAYS_BIRTH`, and `AMT_GOODS_PRICE`. Spend 10 minutes reading the plot:

- Each row = one feature.
- Each dot = one instance (5000 instances).
- X-axis = SHAP value (impact on log-odds of default). Negative = pushes prediction toward "no default", positive toward "default".
- Color = feature value (red = high, blue = low).

**Industry skill:** be able to look at a SHAP summary and tell a stakeholder a story. Practice with `EXT_SOURCE_1`: "external credit score data; low values (blue dots) push predictions strongly toward default (positive SHAP) — i.e., a low external score is the strongest single signal of default risk in this model."

### Now LightGBM head-to-head

Append to `train.py` or make `code/day04/train_lgbm.py`:

```python
import lightgbm as lgb

lgbm = lgb.LGBMClassifier(
    n_estimators=2000, learning_rate=0.02, num_leaves=63,
    min_child_samples=20, subsample=0.8, colsample_bytree=0.8,
    reg_alpha=0.1, reg_lambda=1.0, scale_pos_weight=pos_weight,
    objective="binary", metric="auc", n_jobs=-1, random_state=42,
)
lgbm.fit(X_tr, y_tr, eval_set=[(X_te, y_te)], callbacks=[lgb.early_stopping(50)])
p_lgbm = lgbm.predict_proba(X_te)[:, 1]
print(f"LightGBM ROC-AUC: {roc_auc_score(y_te, p_lgbm):.4f}")
print(f"Blend (avg) ROC-AUC: {roc_auc_score(y_te, (p + p_lgbm)/2):.4f}")
```

Note which wins and by how much. The blend almost always beats either alone — that's a free lesson in ensembling.

**Acceptance criteria for Build B:**
- Three SHAP plots saved.
- A one-paragraph interpretation of the top feature in `code/day04/INTERPRETATION.md`.
- LightGBM result and blend score logged.

---

## 17:00–18:00 — Code Review

```bash
git add code/day04/ artifacts/day04/
git commit -m "day-4: home credit XGBoost + SHAP + LightGBM comparison"
```

Prompt:

```
Review code/day04/. You are a credit risk modeling lead.

1. Look at my data cleaning — what's missing? Bureau and POS files weren't joined.
   Should they have been today, or is that a tomorrow problem?
2. Hyperparameters — are my XGBoost defaults reasonable for credit risk, or am
   I just copying StackOverflow? Defend each non-default choice in 1 sentence.
3. SHAP interpretation — read code/day04/INTERPRETATION.md. Is my story
   honest and grounded, or am I cherry-picking?
4. Imbalance handling — scale_pos_weight vs is_unbalance vs SMOTE — when do you
   actually reach for each?
5. Is `early_stopping_rounds=50` on the test set leaking? (Yes, partly. Explain to me.)
6. The LightGBM/XGBoost blend works — is averaging the right way or should I rank-average?

Then quiz me on 3 SHAP gotchas (TreeExplainer interventional vs path-dependent, etc.).
```

Save to `reviews/day04-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Permutation importance.** Compute `sklearn.inspection.permutation_importance` and compare to SHAP ranking. Do they agree on the top 10? Where they disagree — why?
- **B: Force plot for one customer.** Pick a single high-risk and a single low-risk prediction; produce SHAP force plots: `shap.force_plot(explainer.expected_value, sv.values[i], sample.iloc[i])`. Save as HTML.
- **C: Add `bureau.csv` features.** Download it, aggregate by `SK_ID_CURR` (mean/max/min of `DAYS_CREDIT`, etc.), join, retrain. Did AUC improve? Document in NOTES.
- **D: GPU acceleration.** If you have a GPU, set `tree_method="gpu_hist"` (XGB ≤1.7) or `device="cuda"` (XGB ≥2.0). Benchmark speedup.
- **E: Read** "XGBoost: A Scalable Tree Boosting System" (Chen & Guestrin 2016) Section 3 only (the regularized objective). Write 5 bullets on what's actually different about XGBoost vs vanilla GBM.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Forgetting `enable_categorical=True` and silently passing categoricals as `object` → XGBoost rejects or wastes them.
- **Pitfall 2:** Using `scale_pos_weight` AND `class_weight='balanced'` AND SMOTE. Pick one.
- **Pitfall 3:** Running SHAP on the full 60k test set without sampling → 30 min wait. Sample to 5–10k.
- **Pitfall 4:** Interpreting SHAP as "causal." It isn't. SHAP says "given this model's behavior, this feature contributed X to this prediction." Causal claims require a causal model.
- **Pitfall 5:** `early_stopping_rounds` against the test set instead of a validation set. This *does* peek at test data. In a strict pipeline, hold out a separate val set for early stopping.

---

## Quiz (tomorrow 13:00)

`code/day04/quiz_answers.md`.

1. In one sentence each: what is bagging, boosting, and stacking?
2. Why does XGBoost compute the second-order gradient (Hessian) of the loss, while sklearn's GradientBoosting only uses first-order? What does this buy us?
3. Explain `min_child_weight` in XGBoost — what does it actually constrain and how is it different from `min_samples_leaf` in RF?
4. SHAP values sum to: `prediction - expected_value`. Why does this property matter for interpretability?
5. When should you prefer LightGBM to XGBoost?
6. `subsample=0.8` in XGBoost — what is being subsampled and why does it help?
7. You add 50 new features and your model AUC goes up 0.001 on validation. Should you ship them? Why or why not?
8. The model is 0.76 AUC on Jan data, 0.71 on Apr data. List three plausible explanations.

---

→ Tomorrow at 09:00: open `DAILY/day-05.md`.
