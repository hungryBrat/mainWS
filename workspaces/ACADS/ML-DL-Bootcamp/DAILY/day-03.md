# Day 3 — Classification & Imbalanced Data

**Industry context:** Fraud, churn, anomaly detection, medical diagnosis — almost every binary classification problem in production is imbalanced. Accuracy is meaningless; you need PR-AUC, calibrated probabilities, and a tuned threshold tied to business cost. Today is the bread and butter of applied classification.

**Objective by EOD:**
- Build a fraud classifier from a 99.83%-imbalanced dataset.
- Compare baselines: logistic regression, class-weighted, SMOTE-oversampled.
- Tune the decision threshold to a business cost trade-off.
- Calibrate the probabilities and verify with a reliability diagram.
- **Target:** PR-AUC ≥ 0.78 on test set.

**Dataset (locked):** Kaggle — *Credit Card Fraud Detection* (Worldline / ULB, 284k transactions, 492 frauds → 0.172% positive class).

```bash
mkdir -p data/raw/day03 && cd data/raw/day03
uv run kaggle datasets download -d mlg-ulb/creditcardfraud
unzip -o creditcardfraud.zip
ls -lh creditcard.csv   # ~144MB
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
```

If kaggle 403s, accept the dataset rules at https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud.

---

## 09:00–10:00 — Theory

**Read:**
1. Géron HOML, **Ch. 3 — Classification**, pages 99–135 (3rd ed.). The MNIST examples translate directly.
2. sklearn — **Imbalanced classes**: https://scikit-learn.org/stable/modules/svm.html#unbalanced-problems and https://scikit-learn.org/stable/modules/model_evaluation.html#precision-recall (read just the PR-AUC section).
3. Skim — *Calibration of probabilities*: https://scikit-learn.org/stable/modules/calibration.html (top half).

**Mental model to internalize:** the optimal classification *decision* depends on the cost matrix of FP vs FN. The model gives you probabilities; *you* set the threshold. Default 0.5 is a hidden assumption that costs are equal — they rarely are.

---

## 10:00–13:00 — Build A: baseline + class weights

Create `code/day03/train.py`. The dataset has columns `Time, V1...V28, Amount, Class`. `V1..V28` are PCA-anonymized features; `Class` is 1 = fraud.

```python
"""Day 3 — Fraud detection with imbalanced classes."""
from __future__ import annotations

import logging
from pathlib import Path

import joblib
import numpy as np
import pandas as pd
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import (
    average_precision_score, precision_recall_curve, roc_auc_score,
    classification_report, confusion_matrix, brier_score_loss,
)
from sklearn.model_selection import StratifiedKFold, cross_val_score, train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.calibration import CalibratedClassifierCV, calibration_curve

ART = Path("artifacts/day03"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day03")
RNG = 42


def load() -> tuple[pd.DataFrame, pd.Series]:
    df = pd.read_csv("data/raw/day03/creditcard.csv")
    y = df["Class"].astype(int)
    X = df.drop(columns=["Class"])
    log.info(f"Loaded {len(df):,} rows, {y.sum()} positives ({y.mean()*100:.3f}%)")
    return X, y


def baseline(X_tr, y_tr, X_te, y_te) -> dict:
    """Plain logistic regression. Default threshold."""
    pipe = Pipeline([("sc", StandardScaler()),
                     ("lr", LogisticRegression(max_iter=2000, random_state=RNG))])
    pipe.fit(X_tr, y_tr)
    p = pipe.predict_proba(X_te)[:, 1]
    return {"name": "plain_lr",
            "pr_auc": average_precision_score(y_te, p),
            "roc_auc": roc_auc_score(y_te, p),
            "pipe": pipe}


def class_weighted(X_tr, y_tr, X_te, y_te) -> dict:
    pipe = Pipeline([("sc", StandardScaler()),
                     ("lr", LogisticRegression(max_iter=2000, random_state=RNG,
                                               class_weight="balanced"))])
    pipe.fit(X_tr, y_tr)
    p = pipe.predict_proba(X_te)[:, 1]
    return {"name": "balanced_lr",
            "pr_auc": average_precision_score(y_te, p),
            "roc_auc": roc_auc_score(y_te, p),
            "pipe": pipe}


def main() -> None:
    X, y = load()
    X_tr, X_te, y_tr, y_te = train_test_split(
        X, y, test_size=0.25, random_state=RNG, stratify=y)

    results = []
    for fn in [baseline, class_weighted]:
        r = fn(X_tr, y_tr, X_te, y_te)
        log.info(f"{r['name']}: PR-AUC={r['pr_auc']:.4f} ROC-AUC={r['roc_auc']:.4f}")
        results.append(r)
        joblib.dump(r["pipe"], ART / f"{r['name']}.joblib")

    # Persist test predictions for downstream threshold tuning + calibration scripts
    best = max(results, key=lambda r: r["pr_auc"])
    probs = best["pipe"].predict_proba(X_te)[:, 1]
    np.savez(ART / "test_probs.npz", y=y_te.values, p=probs, name=best["name"])
    log.info(f"Best so far: {best['name']} — wrote test_probs.npz")


if __name__ == "__main__":
    main()
```

Run: `uv run python code/day03/train.py`.

Expected output: plain_lr PR-AUC ~0.72, balanced_lr PR-AUC ~0.73. Both ROC-AUC ~0.97 — note how ROC-AUC is misleadingly high on imbalanced data. PR-AUC is the right metric.

---

## 13:00–14:00 — Lunch + Day-2 Quiz

Run the Day-2 quiz. Grade with Claude same as yesterday.

---

## 14:00–17:00 — Build B: SMOTE + threshold tuning + calibration

Create `code/day03/improve.py`:

### Part 1: SMOTE inside a Pipeline (the right way)

Use `imblearn.pipeline.Pipeline` (not sklearn's) so SMOTE is applied **only during fit, only on training folds**:

```python
from imblearn.over_sampling import SMOTE
from imblearn.pipeline import Pipeline as ImbPipeline
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import average_precision_score
from sklearn.model_selection import StratifiedKFold, cross_val_score

smote_pipe = ImbPipeline([
    ("sc", StandardScaler()),
    ("smote", SMOTE(sampling_strategy=0.1, random_state=42)),  # bring positives to 10% of negs
    ("lr", LogisticRegression(max_iter=2000, random_state=42)),
])
cv = StratifiedKFold(5, shuffle=True, random_state=42)
scores = cross_val_score(smote_pipe, X_tr, y_tr, cv=cv, scoring="average_precision", n_jobs=-1)
print(f"SMOTE-LR CV PR-AUC: {scores.mean():.4f} ± {scores.std():.4f}")
```

**Important:** `sampling_strategy=1.0` (full balance) usually hurts. Try `[0.05, 0.1, 0.25]` and pick best CV.

### Part 2: Threshold tuning

Load `test_probs.npz` from Build A. Vary threshold over a fine grid, compute precision/recall/F1, plot all three vs threshold. Then:

```python
# Business cost: FN costs $100 (missed fraud), FP costs $5 (annoyed customer)
cost_fn, cost_fp = 100, 5
thresholds = np.linspace(0.001, 0.999, 500)
costs = []
for t in thresholds:
    yhat = (p >= t).astype(int)
    fn = ((y == 1) & (yhat == 0)).sum()
    fp = ((y == 0) & (yhat == 1)).sum()
    costs.append(fn * cost_fn + fp * cost_fp)
best_t = thresholds[np.argmin(costs)]
print(f"Cost-minimizing threshold: {best_t:.3f}")
```

Plot cost vs threshold. The minimum should be **well below 0.5** for this imbalance + cost ratio.

### Part 3: Calibration

```python
from sklearn.calibration import CalibratedClassifierCV, calibration_curve
from sklearn.metrics import brier_score_loss
import matplotlib.pyplot as plt

# Wrap the SMOTE pipe in CalibratedClassifierCV (sigmoid = Platt scaling)
calibrated = CalibratedClassifierCV(smote_pipe, method="sigmoid", cv=3)
calibrated.fit(X_tr, y_tr)
p_cal = calibrated.predict_proba(X_te)[:, 1]

# Reliability diagram: bin predicted probs, compare to observed positive rate
prob_true, prob_pred = calibration_curve(y_te, p_cal, n_bins=10, strategy="quantile")
plt.plot(prob_pred, prob_true, "o-", label="Calibrated")
plt.plot([0,1], [0,1], "k--", label="Perfect")
plt.xlabel("Mean predicted prob"); plt.ylabel("Fraction positive"); plt.legend()
plt.savefig("artifacts/day03/reliability.png", dpi=120, bbox_inches="tight")

print(f"Brier (uncal): {brier_score_loss(y_te, p):.5f}")
print(f"Brier (cal):   {brier_score_loss(y_te, p_cal):.5f}")
```

**Acceptance criteria for Build B:**
- SMOTE-LR test PR-AUC ≥ 0.78.
- Threshold tuning plot + business cost minimum identified and printed.
- Reliability diagram saved to `artifacts/day03/reliability.png`.
- Calibrated Brier score lower than uncalibrated.

If you can't hit PR-AUC 0.78 with LR alone, also try:

```python
from sklearn.ensemble import HistGradientBoostingClassifier
hgb = HistGradientBoostingClassifier(class_weight="balanced", random_state=42)
hgb.fit(X_tr, y_tr)
```

HGB usually gets PR-AUC ~0.85 on this dataset out of the box. Use it if needed — note in your code WHY.

---

## 17:00–18:00 — Code Review with Claude

```bash
git add code/day03/ artifacts/day03/
git commit -m "day-3: fraud detection — SMOTE, threshold tuning, calibration"
```

Prompt:

```
Review code/day03/. Be a senior fraud-team ML engineer.

1. Is my data split appropriate (stratified, no leakage, correct fraction)?
2. Did I apply SMOTE correctly — only inside CV/training, never on test?
3. Did I use the right metric (PR-AUC) consistently and is my threshold-tuning logic sound?
4. The business cost (FN=$100, FP=$5) is illustrative — what cost ratio does the
   industry typically use for card fraud and how would I source real numbers?
5. Calibration — when is sigmoid better than isotonic and vice versa?
6. What would break if the deployment population had a different fraud rate than training?

Score my Day 3 effort out of 10 and give me the two biggest improvements.
```

Save to `reviews/day03-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: PR curve plot.** Use `precision_recall_curve` and matplotlib to plot the PR curve with the chosen threshold marked. Save as `artifacts/day03/pr_curve.png`.
- **B: Cross-validate the threshold itself.** Threshold chosen on test set = leakage in real-world deployment. Pick the threshold using `StratifiedKFold` inside training data, then evaluate once on test. Compare with what you got naïvely.
- **C: Read** the "Practical Lessons from Predicting Clicks on Ads at Facebook" abstract + section 4 (https://research.facebook.com/file/273183074306353/Practical-Lessons-from-Predicting-Clicks-on-Ads-at-Facebook.pdf). It's the canonical industry paper on calibrated probabilities + threshold strategy.
- **D: Try isotonic calibration** instead of sigmoid. Compare reliability diagrams side by side.
- **E: Group-aware split.** The dataset has a `Time` column — split by time (earliest 75% train, latest 25% test) and re-evaluate. Does performance drop? It should — this is realistic fraud-temporal-drift.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Reporting accuracy on a 0.172%-positive dataset. A model that predicts "never fraud" gets 99.83% accuracy and is worthless.
- **Pitfall 2:** Applying SMOTE before train_test_split → leaks synthetic positives into your test set.
- **Pitfall 3:** Using sklearn's Pipeline with SMOTE inside — it tries to oversample at predict time, which is wrong. Use `imblearn.pipeline.Pipeline`.
- **Pitfall 4:** Calibrating on the same data you trained on. Use `CalibratedClassifierCV(..., cv=...)` to do it properly via held-out folds.
- **Pitfall 5:** Picking the threshold on the test set. Use a validation set or CV; the test set is for final reporting only.

---

## Quiz (tomorrow 13:00)

Write answers to `code/day03/quiz_answers.md`.

1. Define precision, recall, and F1 in your own words. Which is more important for card fraud?
2. Explain why ROC-AUC ≈ 0.97 on this dataset is *not* impressive.
3. What's the relationship between PR-AUC and average precision (`average_precision_score`)?
4. When SMOTE creates a synthetic point between two minority class points, what assumption is it making? When does that assumption break?
5. You set `class_weight="balanced"`. What does sklearn actually compute behind the scenes? (Hint: inverse frequency.)
6. After Platt scaling, your predicted probabilities are better calibrated but PR-AUC is slightly *lower*. Why is this possible?
7. Your model has PR-AUC 0.8 in training, 0.7 in production. List four plausible causes.
8. The product team says "we need 95% recall on fraud." Walk through how you choose the threshold and report the corresponding precision.

---

→ Tomorrow at 09:00: open `DAILY/day-04.md`.
