# Day 1 — Data Wrangling Mastery

**Industry context:** ML engineers spend 60–80% of their time on data, not models. The first 6 months of any ML job are dominated by `pandas`, `polars`, SQL, and joining messy tables. Today you build the muscle memory.

**Objective by EOD:**
- Load a 1.5M-row dataset without crashing your laptop.
- Produce a clean, reproducible EDA notebook with 12 figure cells.
- Know when to reach for `polars` vs `pandas`.
- Detect and handle outliers, missing values, and time features.

**Dataset (locked):** Kaggle — *NYC Taxi Trip Duration*. Train file has 1.46M rows.

```bash
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
mkdir -p data/raw/day01 && cd data/raw/day01
uv run kaggle competitions download -c nyc-taxi-trip-duration
unzip -o nyc-taxi-trip-duration.zip
unzip -o train.zip
ls -lh train.csv     # expect ~190MB
```

If kaggle CLI errors with "you must accept this competition's rules": visit https://www.kaggle.com/c/nyc-taxi-trip-duration/rules and click Accept.

---

## 09:00–10:00 — Theory & Intuition

**Read** (in order, ~50 min):

1. Géron *Hands-On Machine Learning*, **Ch. 2 — Look at the Big Picture / Get the Data / Discover and Visualize**, pages 35–63 (3rd ed.). Skip the housing dataset code — you'll do California Housing tomorrow. Today, focus on the workflow shape: business problem → data → EDA → cleaning.
2. pandas docs — **"10 minutes to pandas"**: https://pandas.pydata.org/docs/user_guide/10min.html. Read every example, run none — you'll write your own.
3. polars docs — **"Coming from pandas"**: https://docs.pola.rs/user-guide/migration/pandas/. Just skim the table.

**Watch (skip if short on time):**
- Wes McKinney short talk: https://www.youtube.com/watch?v=DW87xs1cw84 (15 min, 1.5x).

---

## 10:00–13:00 — Build A: pandas EDA notebook

Create `code/day01/eda_pandas.ipynb`. Launch Jupyter:

```bash
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
uv run jupyter lab
```

Build these **12 numbered cells**. Each must produce a visible output. No copying from Kaggle kernels — write it yourself.

1. **Load + memory footprint.** Load `data/raw/day01/train.csv` into `df`. Print `df.info(memory_usage="deep")` and the raw shape. Convert `pickup_datetime` and `dropoff_datetime` to `datetime64[ns]`. Verify memory dropped by ≥ 20%.
2. **Trip duration sanity.** Compute `(dropoff_datetime - pickup_datetime).dt.total_seconds()` and compare to the `trip_duration` column. They should match exactly. Print the count of mismatches.
3. **Outlier detection.** Plot `trip_duration` histogram with log-scale y-axis. Identify the long tail. Compute the 99.5th percentile. Filter `df` to `(trip_duration >= 60) & (trip_duration <= df.trip_duration.quantile(0.995))`. Report rows removed.
4. **Geographic sanity.** Plot 2D histogram (`hexbin`) of `pickup_longitude` vs `pickup_latitude`. Zoom to NYC: longitude `[-74.05, -73.75]`, latitude `[40.6, 40.9]`. Drop rows outside this box; report count dropped.
5. **Distance feature.** Add a `haversine_km` column using the Haversine formula on pickup/dropoff coords. Write the formula yourself (look it up once, then type it). Verify median trip distance is between 1.5 and 3.0 km.
6. **Speed feature.** Add `avg_speed_kmh = haversine_km / (trip_duration / 3600)`. Plot speed distribution. Anything above 150 km/h is impossible in Manhattan — drop those rows.
7. **Time features.** Extract `hour`, `dayofweek`, `month`, and `is_weekend` from `pickup_datetime`. Plot a heatmap of mean `trip_duration` by `hour × dayofweek` using `sns.heatmap`. You should see rush-hour patterns.
8. **Categorical: vendor and passenger count.** Bar plot of mean `trip_duration` by `vendor_id` and by `passenger_count`. Note any surprises (e.g., is 7-passenger trips longer? why?).
9. **Missing values.** `df.isna().sum()`. There should be near-zero NaNs here, but build the habit — write the cell anyway and confirm.
10. **Pairwise correlations.** Compute `df.select_dtypes(include='number').corr()` and plot heatmap. Identify which feature has highest correlation with `trip_duration`.
11. **Target transform.** Plot `trip_duration` vs `np.log1p(trip_duration)`. The log version should look closer to normal. Note this — you'll use `log1p(target)` for regression tomorrow.
12. **Save the cleaned dataframe.** Save as `data/processed/day01_taxi_clean.parquet`. Verify by reloading and checking row count.

**Acceptance criteria:**
- ≥ 12 cells, each with an output figure or printed value.
- No `for` loops over rows (Pandas-level only). If you wrote `for i in range(len(df))`, you failed and must redo.
- `data/processed/day01_taxi_clean.parquet` exists.

---

## 13:00–14:00 — Lunch (no quiz today, Day 1)

Eat. Walk. Do not open the laptop.

---

## 14:00–17:00 — Build B: polars rewrite & benchmark

Create `code/day01/eda_polars.py` (a script, not a notebook — you'll learn the difference).

Rewrite **cells 1–7 only** of the pandas notebook in polars. Use `pl.scan_csv(...).filter(...).with_columns(...).collect()` — i.e., the lazy API. Then add a benchmark at the bottom:

```python
import time
import pandas as pd
import polars as pl

PATH = "data/raw/day01/train.csv"

t0 = time.perf_counter()
df_pd = pd.read_csv(PATH, parse_dates=["pickup_datetime", "dropoff_datetime"])
t_pd_load = time.perf_counter() - t0

t0 = time.perf_counter()
df_pl = pl.read_csv(PATH, try_parse_dates=True)
t_pl_load = time.perf_counter() - t0

print(f"pandas load: {t_pd_load:.2f}s")
print(f"polars load: {t_pl_load:.2f}s")
print(f"speedup: {t_pd_load / t_pl_load:.2f}x")
```

Then time a more complex operation: group-by hour, agg mean duration. Time both. Save the comparison as a comment block at the top of the file:

```python
# Benchmark (laptop CPU):
#   pandas load: X.Xs / polars load: X.Xs / speedup: X.Xx
#   pandas groupby: X.Xs / polars groupby: X.Xs / speedup: X.Xx
```

**Acceptance criteria:**
- The polars script reproduces the same `(filtered_row_count, mean_haversine_km)` tuple as the pandas notebook within float tolerance.
- Speedup is ≥ 2x for load and ≥ 3x for groupby (on CPU; will be higher on more cores).

---

## 17:00–18:00 — Code Review with Claude

Commit your work first:

```bash
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
git add code/day01/ data/processed/day01_taxi_clean.parquet
git commit -m "day-1: pandas EDA + polars rewrite"
```

Open a fresh Claude conversation in this workspace and paste the exact prompt below:

```
Review my Day 1 code at code/day01/.

Check the pandas notebook for:
1. Any row-level for-loops or .apply() that should be vectorized.
2. Memory inefficiency — am I copying frames unnecessarily?
3. The 12-cell coverage — does cell N actually do what the daily file specifies?
4. Are filter thresholds documented (why 99.5th percentile, why ≥60s)?
5. Is the parquet save schema clean (no object dtypes, indexes reset)?

For the polars script:
6. Did I use lazy mode (.scan_csv) or eager (.read_csv) and is the choice justified?
7. Is my benchmark fair — am I comparing apples to apples (same operation, same data)?

Be ruthless. Senior data engineer voice. Give me 5 priority fixes ranked by impact.
```

Save the response as `reviews/day01-review.md` and **apply the top-3 fixes before dinner**.

---

## 18:00–19:00 — Dinner

Step away.

---

## 19:00–21:00 — Hardening

Pick **two** of the following. Not three. Not all five.

- **Stretch A:** Add cell 13 — a parquet round-trip benchmark (`to_parquet` vs `to_csv` for size and write/read speed). Add insights to the notebook.
- **Stretch B:** Write `code/day01/test_clean.py` with three pytest tests: (a) no negative durations, (b) no NaN in critical columns, (c) latitude/longitude in NYC bounds. Run `uv run pytest code/day01/`.
- **Stretch C:** Compute the great-circle bearing and add it as a feature. Plot bearing histogram.
- **Stretch D:** Try `df.style.format(...)` on a small summary table — preparing for nicer report-style outputs.
- **Stretch E:** Skim https://wesmckinney.com/book/ Ch. 4 on the pandas datatype model (Categorical, nullable Int).

---

## 21:00–21:30 — Journal

Append the entry to `journal.md`. Use the template in `README.md`.

---

## Common Pitfalls (re-read tomorrow morning)

- **Pitfall 1:** Reading the same CSV multiple times. Read once, save parquet, reuse.
- **Pitfall 2:** Forgetting `parse_dates=...` and operating on strings.
- **Pitfall 3:** Using `df.apply(lambda r: ...)` row-wise — always vectorize.
- **Pitfall 4:** Filtering with a chained boolean without parens: `df[df.x > 0 & df.y < 10]` will fail due to operator precedence. Use `df[(df.x > 0) & (df.y < 10)]`.
- **Pitfall 5:** Setting on a chained slice: `df[df.x > 0]['y'] = 1` silently fails. Use `df.loc[df.x > 0, 'y'] = 1`.

---

## Quiz (you'll run tomorrow 13:00, no peeking)

Type your answers in `code/day01/quiz_answers.md`, then ask Claude to grade.

1. What's the difference between `df.iloc[0]` and `df.loc[0]`?
2. Why does `df.x.apply(np.sqrt)` defeat the purpose of NumPy?
3. What does `df.dtypes` return, and why care about `category` vs `object`?
4. When does `pd.merge(how='left')` produce more rows than the left frame had?
5. Name one operation polars does meaningfully faster than pandas and explain why (hint: column-major / Arrow / parallelization).
6. What's the cost of `df.copy(deep=True)` on a 1M-row frame and when do you actually need it?
7. Why is `parquet` better than `csv` for ML pipelines? Give two reasons.
8. What does `pd.cut` do, and when would you use it on a numeric feature for an ML model?

---

→ Tomorrow at 09:00: open `DAILY/day-02.md`.
