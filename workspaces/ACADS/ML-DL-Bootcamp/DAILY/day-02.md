# Day 2 — Scikit-Learn Pipelines & Regression

**Industry context:** Whatever else you use, `scikit-learn` is the lingua franca. Every ML interview will assume you know `Pipeline`, `ColumnTransformer`, `GridSearchCV`, and how to serialize a fitted pipeline. Build a production-grade tabular pipeline today.

**Objective by EOD:**
- Build an sklearn `Pipeline` that handles numeric + categorical features with `ColumnTransformer`.
- Use proper CV (no data leakage).
- Tune one hyperparameter with `GridSearchCV` and one with `RandomizedSearchCV`.
- Serialize the whole pipeline to `.joblib` and reload it for inference.
- **Target:** RMSE on `log(target)` ≤ 0.55 on a held-out test set.

**Dataset:** California Housing (built into sklearn — no download).

```python
from sklearn.datasets import fetch_california_housing
data = fetch_california_housing(as_frame=True)
X, y = data.data, data.target
# 8 features, 20640 rows, target = median house value in $100k
```

---

## 09:00–10:00 — Theory & Intuition

**Read:**
1. Géron HOML, **Ch. 2 — Prepare the Data + Select and Train a Model + Fine-Tune Your Model**, pages 63–95. This is *the* end-to-end workflow.
2. sklearn docs — **Pipelines and composite estimators**: https://scikit-learn.org/stable/modules/compose.html (just §6.1.1 to §6.1.4).
3. sklearn docs — **Cross-validation: evaluating estimator performance**: https://scikit-learn.org/stable/modules/cross_validation.html (top section + §3.1.2.1.1 K-fold).

**Skip** everything about visualizations and ensemble specifics — those come Day 4.

---

## 10:00–13:00 — Build A: end-to-end pipeline

Create `code/day02/train.py`. **No notebooks today** — production code lives in `.py`.

### Skeleton you must fill in

```python
"""Day 2 — California Housing regression with a production-style sklearn pipeline."""
from __future__ import annotations

import argparse
import logging
from pathlib import Path

import joblib
import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.datasets import fetch_california_housing
from sklearn.ensemble import RandomForestRegressor
from sklearn.impute import SimpleImputer
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error, mean_absolute_error
from sklearn.model_selection import train_test_split, KFold, cross_val_score, RandomizedSearchCV
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

ARTIFACT_DIR = Path("artifacts/day02")
ARTIFACT_DIR.mkdir(parents=True, exist_ok=True)

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day02")


def load_data() -> tuple[pd.DataFrame, pd.Series]:
    """Return X, y for California Housing. y is log1p of median house value."""
    raise NotImplementedError


def build_preprocessor(X: pd.DataFrame) -> ColumnTransformer:
    """Numeric features → impute median + scale. (No categoricals in this dataset.)"""
    raise NotImplementedError


def evaluate(pipe: Pipeline, X_test: pd.DataFrame, y_test_log: pd.Series) -> dict:
    """Return dict with rmse_log, rmse_orig, mae_orig."""
    raise NotImplementedError


def main() -> None:
    X, y_log = load_data()
    X_train, X_test, y_train, y_test = train_test_split(X, y_log, test_size=0.2, random_state=42)

    pre = build_preprocessor(X_train)

    # --- Baseline: ridge ---
    ridge_pipe = Pipeline([("pre", pre), ("model", Ridge(alpha=1.0, random_state=42))])
    cv = KFold(n_splits=5, shuffle=True, random_state=42)
    scores = cross_val_score(ridge_pipe, X_train, y_train, cv=cv,
                             scoring="neg_root_mean_squared_error", n_jobs=-1)
    log.info(f"Ridge CV RMSE (log target) = {-scores.mean():.4f} ± {scores.std():.4f}")

    # --- Random forest with RandomizedSearchCV ---
    rf_pipe = Pipeline([("pre", pre), ("model", RandomForestRegressor(random_state=42, n_jobs=-1))])
    param_dist = {
        "model__n_estimators": [100, 200, 400],
        "model__max_depth": [None, 8, 16, 24],
        "model__min_samples_leaf": [1, 2, 4],
        "model__max_features": ["sqrt", 0.5, 0.8],
    }
    search = RandomizedSearchCV(
        rf_pipe, param_distributions=param_dist, n_iter=15, cv=cv,
        scoring="neg_root_mean_squared_error", n_jobs=-1, random_state=42, verbose=1
    )
    search.fit(X_train, y_train)
    log.info(f"Best params: {search.best_params_}")
    log.info(f"Best CV RMSE (log target) = {-search.best_score_:.4f}")

    final_metrics = evaluate(search.best_estimator_, X_test, y_test)
    log.info(f"Test metrics: {final_metrics}")

    # Persist the *fitted Pipeline* — preprocessor + model in one artifact.
    joblib.dump(search.best_estimator_, ARTIFACT_DIR / "pipeline.joblib")
    log.info(f"Saved pipeline → {ARTIFACT_DIR/'pipeline.joblib'}")


if __name__ == "__main__":
    main()
```

Now fill in the three stub functions. Hints (not full answers):

- `load_data`: `fetch_california_housing(as_frame=True)`. Return `X` (DataFrame) and `np.log1p(y)` (Series). Set name on the Series to `"log_median_house_value"`.
- `build_preprocessor`: There are no categoricals in this dataset (verify with `X.dtypes`). So `ColumnTransformer` has one branch: numeric → `Pipeline([("imp", SimpleImputer(strategy="median")), ("sc", StandardScaler())])`. Use `make_column_selector(dtype_include=np.number)` to pass columns dynamically — this is the future-proof pattern.
- `evaluate`: Predict on `X_test`, exponentiate back with `np.expm1`. Compute RMSE on both the log scale and original scale, plus MAE on original.

Run:

```bash
uv run python code/day02/train.py
```

**Acceptance criteria:**
- Ridge baseline CV RMSE (log) prints out (~0.30–0.40 range).
- RandomizedSearchCV completes 15 trials.
- Final test RMSE (log target) ≤ 0.55 — the bar.
- `artifacts/day02/pipeline.joblib` exists.

If you miss the bar, the most likely cause is forgetting to fit on training only and leaking the scaler to the test set (Pipeline prevents this automatically — that's the lesson).

---

## 13:00–14:00 — Lunch + Day-1 Quiz

Run yesterday's quiz. Type answers in `code/day01/quiz_answers.md`. After eating, paste this into Claude:

```
Grade my Day 1 quiz at code/day01/quiz_answers.md against the questions in
DAILY/day-01.md. For each: correct (✓), partially correct (~), or wrong (✗).
Give the canonical answer where I missed. Score me out of 8.
```

Below 6/8? Spend 20 min tonight (during hardening) re-reading the pandas docs section you fumbled.

---

## 14:00–17:00 — Build B: inference script + tests

Create `code/day02/predict.py`:

```python
"""Load saved pipeline and predict on user-supplied rows (JSON in, JSON out)."""
import json
import sys
from pathlib import Path

import joblib
import numpy as np
import pandas as pd

PIPE = joblib.load("artifacts/day02/pipeline.joblib")
FEATURES = ["MedInc", "HouseAge", "AveRooms", "AveBedrms",
            "Population", "AveOccup", "Latitude", "Longitude"]

def predict(rows: list[dict]) -> list[float]:
    df = pd.DataFrame(rows)[FEATURES]
    log_pred = PIPE.predict(df)
    return np.expm1(log_pred).tolist()

if __name__ == "__main__":
    payload = json.loads(sys.stdin.read())
    print(json.dumps({"predictions_100k_usd": predict(payload)}, indent=2))
```

Test:

```bash
echo '[{"MedInc": 5.2, "HouseAge": 30, "AveRooms": 5.5, "AveBedrms": 1.0, "Population": 1200, "AveOccup": 2.8, "Latitude": 34.05, "Longitude": -118.25}]' | uv run python code/day02/predict.py
```

Now create `code/day02/test_pipeline.py`:

```python
import joblib
import numpy as np
import pandas as pd
import pytest
from sklearn.pipeline import Pipeline

PIPE_PATH = "artifacts/day02/pipeline.joblib"

@pytest.fixture(scope="module")
def pipe() -> Pipeline:
    return joblib.load(PIPE_PATH)

def make_row(**overrides):
    base = dict(MedInc=5.0, HouseAge=20, AveRooms=5.0, AveBedrms=1.0,
                Population=1000, AveOccup=3.0, Latitude=34.0, Longitude=-118.0)
    base.update(overrides)
    return pd.DataFrame([base])

def test_pipeline_predicts_positive(pipe):
    pred = np.expm1(pipe.predict(make_row()))[0]
    assert pred > 0

def test_higher_income_higher_prediction(pipe):
    low = pipe.predict(make_row(MedInc=1.0))[0]
    high = pipe.predict(make_row(MedInc=15.0))[0]
    assert high > low, "Higher MedInc must predict higher house value"

def test_pipeline_handles_missing(pipe):
    """SimpleImputer should fill in NaNs at inference too."""
    row = make_row(MedInc=np.nan)
    pred = pipe.predict(row)[0]
    assert not np.isnan(pred)
```

Run: `uv run pytest code/day02/ -v`. All three tests must pass.

**Acceptance criteria for Build B:**
- `predict.py` accepts JSON via stdin and emits predictions.
- 3 pytest tests pass.

---

## 17:00–18:00 — Code Review with Claude

Commit:

```bash
git add code/day02/ artifacts/day02/
git commit -m "day-2: california housing sklearn pipeline + tests"
```

Paste this prompt to Claude:

```
Review my Day 2 code at code/day02/. Check for:
1. Data leakage — am I fitting any preprocessor on the whole dataset before splitting?
2. The use of Pipeline vs manual preprocessing — am I getting the leakage-prevention benefit?
3. Whether ColumnTransformer with make_column_selector is appropriate here (there
   are no categoricals — am I over-engineering or future-proofing wisely?)
4. Hyperparameter search: is n_iter=15 enough? What's my CV variance?
5. Test coverage — what edge case did I miss?
6. The predict.py script — what would break if a real user sent a row with extra columns or missing required ones?

Then quiz me with 3 questions about Pipelines and ColumnTransformer that test
understanding, not memorization.
```

Save to `reviews/day02-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Polynomial features.** Add `PolynomialFeatures(degree=2, interaction_only=True)` between scaler and ridge. Re-CV. Does it help? Why might it hurt RF?
- **B: HistGradientBoostingRegressor.** Add a third model — `HistGradientBoostingRegressor`. Compare against RF. Often this is the answer in real tabular work.
- **C: Feature engineering.** Add `bedrooms_per_room = AveBedrms/AveRooms` and `rooms_per_household = AveRooms/AveOccup` using `FunctionTransformer`. Re-CV and report delta.
- **D: Learning curves.** Use `sklearn.model_selection.learning_curve` and plot train vs CV RMSE as a function of training set size. Are you under- or overfitting?
- **E: GridSearchCV vs RandomizedSearchCV writeup.** In `code/day02/NOTES.md` write 5 bullets on when to use each.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Calling `StandardScaler().fit_transform(X)` on the whole dataset before split. **Always** wrap in a Pipeline that's fit only on the training fold inside CV.
- **Pitfall 2:** Forgetting `random_state` somewhere — your results won't be reproducible.
- **Pitfall 3:** Using `scoring="neg_mean_squared_error"` and reporting MSE — that's not RMSE. Use `neg_root_mean_squared_error` or take a sqrt yourself.
- **Pitfall 4:** Hardcoding feature names in `predict.py`. Better to read them from the saved pipeline's `feature_names_in_` attribute.
- **Pitfall 5:** Saving the model with `pickle` directly. Use `joblib` — it's optimized for numpy arrays and handles sklearn correctly.

---

## Quiz (run tomorrow at 13:00)

Type answers in `code/day02/quiz_answers.md`, then have Claude grade.

1. What does `ColumnTransformer` do that you can't easily do with two `Pipeline`s in series?
2. Inside `Pipeline([("a", StandardScaler()), ("b", Ridge())])` — when you call `.fit(X, y)`, in what order are the steps called and what does `.transform` vs `.fit_transform` mean?
3. Why is `RandomizedSearchCV` usually better than `GridSearchCV` for ≥4 hyperparameters?
4. What's the difference between `cross_val_score`, `cross_validate`, and `cross_val_predict`?
5. You fit a pipeline with `KFold(shuffle=True)`. The next day you re-run and get different numbers. What's wrong?
6. Your test RMSE is much worse than your CV RMSE. Give three plausible causes.
7. In what situation should you use `TimeSeriesSplit` instead of `KFold`?
8. Explain "regularization strength" for Ridge — what does increasing `alpha` do to coefficients and to bias/variance?

---

→ Tomorrow at 09:00: open `DAILY/day-03.md`.
