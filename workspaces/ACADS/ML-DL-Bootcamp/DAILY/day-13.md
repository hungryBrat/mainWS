# Day 13 — MLOps: Monitoring, Logging, and Drift

**Industry context:** "Why is my model worse this week than last week?" is the most expensive ML question. Without monitoring, you don't know it's broken until the business notices revenue dropping. Today you instrument the Day-12 service with what real ML teams use: structured logs, Prometheus metrics, a Grafana dashboard, and a drift sentinel.

**Objective by EOD:**
- Add structured (JSON) logging to the Day-12 service.
- Expose Prometheus metrics at `/metrics`.
- Spin up Prometheus + Grafana locally via docker-compose.
- Build a 4-panel Grafana dashboard.
- Add a simple input-distribution drift check.

---

## 09:00–10:00 — Theory

**Read:**
1. Chip Huyen *DMLS* — **Chapter 8 (Data Distribution Shifts and Monitoring) — sections 8.1, 8.2, 8.4**. ~30 min.
2. Skim the Prometheus *Concepts* page: https://prometheus.io/docs/concepts/data_model/ (just data model + metric types).
3. Read the `prometheus_client` Python README: https://github.com/prometheus/client_python (first 60 lines).

**Three ideas:**
- **Four golden signals** (from Google SRE book): latency, traffic, errors, saturation. Every service should expose all four.
- **Three drift types**: covariate (input distribution changes), label (label distribution changes), concept (P(y|x) changes). Monitoring strategies differ.
- **Population Stability Index (PSI)** is the simple, widely-used drift detector for tabular features. `PSI(p, q) = Σ (p_i - q_i) * log(p_i / q_i)`. PSI < 0.1 = stable; 0.1–0.25 = slight; > 0.25 = significant drift.

---

## 10:00–13:00 — Build A: instrument the service

### 1. Prometheus metrics

Add to `code/day12/app/metrics.py`:

```python
"""Prometheus instrumentation."""
from prometheus_client import Counter, Histogram, Gauge, generate_latest, CONTENT_TYPE_LATEST

REQUEST_COUNT = Counter(
    "predict_requests_total", "Total /predict requests", ["status"]
)
REQUEST_LATENCY = Histogram(
    "predict_latency_seconds", "End-to-end predict latency in seconds",
    buckets=(0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0),
)
INFERENCE_LATENCY = Histogram(
    "model_inference_seconds", "Model inference latency (excluding I/O)",
    buckets=(0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5),
)
TOP_PRED_CONFIDENCE = Histogram(
    "predict_top_confidence", "Confidence of top-1 prediction",
    buckets=(0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 0.95, 0.99),
)
PREDICTION_CLASS = Counter(
    "predict_class_total", "Predictions by top-1 class", ["predicted_class"]
)
MODEL_LOADED = Gauge("model_loaded", "1 if model loaded, else 0")


def metrics_text() -> bytes:
    return generate_latest()
```

### 2. Hook into `main.py`

Modify `app/main.py`:

```python
import time
from fastapi import Response
from app.metrics import (
    REQUEST_COUNT, REQUEST_LATENCY, INFERENCE_LATENCY,
    TOP_PRED_CONFIDENCE, PREDICTION_CLASS, MODEL_LOADED,
    metrics_text, CONTENT_TYPE_LATEST,
)

# inside lifespan:
MODEL_LOADED.set(1)
# at teardown:
MODEL_LOADED.set(0)


@app.get("/metrics")
def metrics():
    return Response(content=metrics_text(), media_type=CONTENT_TYPE_LATEST)


@app.post("/predict", response_model=PredictResponse)
async def predict(file: UploadFile = File(...)):
    t_total_start = time.perf_counter()
    try:
        if file.content_type not in ALLOWED_TYPES:
            REQUEST_COUNT.labels(status="415").inc()
            raise HTTPException(415, f"Unsupported type: {file.content_type}")
        data = await file.read()
        if len(data) > MAX_BYTES:
            REQUEST_COUNT.labels(status="413").inc()
            raise HTTPException(413, "Payload too large")
        if len(data) < 100:
            REQUEST_COUNT.labels(status="400").inc()
            raise HTTPException(400, "Image too small")

        with INFERENCE_LATENCY.time():
            preds = _state["clf"].predict(data, top_k=5)

        TOP_PRED_CONFIDENCE.observe(preds[0]["prob"])
        PREDICTION_CLASS.labels(predicted_class=preds[0]["class"]).inc()
        REQUEST_COUNT.labels(status="200").inc()
        dt_ms = (time.perf_counter() - t_total_start) * 1000

        log.info("predict.ok", filename=file.filename, top1=preds[0]["class"],
                 top1_prob=preds[0]["prob"], latency_ms=round(dt_ms, 2))

        return PredictResponse(
            filename=file.filename or "unknown",
            inference_ms=round(dt_ms, 2),
            predictions=[{"class": p["class"], "prob": p["prob"]} for p in preds],
        )
    finally:
        REQUEST_LATENCY.observe(time.perf_counter() - t_total_start)
```

### 3. Structured logging config

Add `app/logging_config.py`:

```python
import logging
import structlog

def configure_logging():
    structlog.configure(
        processors=[
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.add_log_level,
            structlog.processors.JSONRenderer(),
        ],
        wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory(),
    )
```

Call `configure_logging()` at top of `main.py`. All your log lines now emit as JSON.

### 4. Smoke test

```bash
cd code/day12
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Open `/metrics` in your browser — you should see Prometheus text exposition output. Send a few `/predict` requests; refresh `/metrics`; counters should increment.

**Acceptance criteria for Build A:**
- `/metrics` returns text in Prometheus format.
- After 5 predict requests, `predict_requests_total{status="200"} ≥ 5`.
- Logs are valid JSON lines (`docker logs flower | jq '.'` works).

---

## 13:00–14:00 — Lunch + Day-12 Quiz

---

## 14:00–17:00 — Build B: Prometheus + Grafana stack

### 1. docker-compose

`code/day12/docker-compose.yaml`:

```yaml
version: "3.9"

services:
  api:
    build: .
    image: flower-classifier:0.2.0
    ports: ["8000:8000"]
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://localhost:8000/health"]
      interval: 30s
      retries: 3

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    ports: ["9090:9090"]
    depends_on: [api]

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_AUTH_ANONYMOUS_ENABLED=true
    volumes:
      - grafana-data:/var/lib/grafana
    ports: ["3000:3000"]
    depends_on: [prometheus]

volumes:
  grafana-data:
```

### 2. Prometheus config

`code/day12/prometheus.yml`:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "flower-classifier"
    static_configs:
      - targets: ["api:8000"]
```

### 3. Launch

```bash
cd code/day12
docker compose up --build
```

In a second terminal, hammer the service so there's data to plot:

```bash
for i in $(seq 1 200); do
  curl -s -X POST -F "file=@sample.jpg" http://localhost:8000/predict > /dev/null
  sleep 0.2
done
```

Open:

- http://localhost:9090 — Prometheus. Try a query: `rate(predict_requests_total[1m])`.
- http://localhost:3000 — Grafana. Login admin/admin. Add Prometheus data source: URL = `http://prometheus:9090`.

### 4. Build a dashboard (4 panels)

Create a new dashboard, name it "Flower Classifier — overview". Add panels:

1. **Requests / sec.** Query: `rate(predict_requests_total[1m])`. Stacked by `status`. Time series.
2. **Latency p50 / p95 / p99 (ms).** Three queries on the same panel:
   - `histogram_quantile(0.50, sum(rate(predict_latency_seconds_bucket[5m])) by (le)) * 1000`
   - `histogram_quantile(0.95, sum(rate(predict_latency_seconds_bucket[5m])) by (le)) * 1000`
   - `histogram_quantile(0.99, sum(rate(predict_latency_seconds_bucket[5m])) by (le)) * 1000`
3. **Top-1 predicted classes.** Query: `topk(10, predict_class_total)`. Bar gauge.
4. **Confidence distribution.** Query: `histogram_quantile(0.5, sum(rate(predict_top_confidence_bucket[5m])) by (le))`. Repeat for 0.1 and 0.9 → 3 lines on time series.

Save the dashboard JSON to `code/day12/grafana_dashboard.json` (Dashboard → ⚙ → Export → JSON).

**Acceptance criteria for Build B:**
- `docker compose up` brings up 3 containers, all healthy.
- Prometheus successfully scrapes API at `/metrics`.
- Grafana dashboard with 4 working panels.
- `grafana_dashboard.json` committed.

---

## 17:00–18:00 — Drift Check + Code Review

### Drift check (sketch)

Create `code/day13/drift.py`:

```python
"""Compare predicted-class distribution from the last 24h to the training prior."""
import json
import numpy as np
from prometheus_client.parser import text_string_to_metric_families
import httpx

PROM = "http://localhost:9090"

def psi(expected: dict, observed: dict, eps: float = 1e-6) -> float:
    classes = sorted(set(expected) | set(observed))
    e = np.array([expected.get(c, eps) for c in classes])
    o = np.array([observed.get(c, eps) for c in classes])
    e = e / e.sum(); o = o / o.sum()
    return float(np.sum((e - o) * np.log((e + eps) / (o + eps))))


def query_prom(q: str) -> dict:
    r = httpx.get(f"{PROM}/api/v1/query", params={"query": q}, timeout=10)
    r.raise_for_status()
    return r.json()["data"]["result"]


def main():
    # Training prior: load from your Day-8 training set class counts
    with open("artifacts/day13/training_class_prior.json") as f:
        train_prior = json.load(f)

    # Live: from Prometheus
    raw = query_prom("predict_class_total")
    live = {item["metric"]["predicted_class"]: float(item["value"][1]) for item in raw}

    score = psi(train_prior, live)
    print(f"PSI = {score:.4f}")
    if score < 0.1: print("Stable.")
    elif score < 0.25: print("Slight drift — monitor.")
    else: print("ALERT — significant drift in predicted-class distribution.")
    return score

if __name__ == "__main__":
    main()
```

Stub `training_class_prior.json` (counts from Flowers102 train set — 10 per class, so uniform):

```bash
mkdir -p artifacts/day13
python -c "import json; json.dump({str(i): 10 for i in range(102)}, open('artifacts/day13/training_class_prior.json', 'w'))"
```

Run after hammering the API:

```bash
uv run python code/day13/drift.py
```

You should see PSI > 0.25 (because you've been sending the same single image 200 times!). That's expected and proves drift detection works.

### Code review

```bash
git add code/day12/ code/day13/ artifacts/day13/
git commit -m "day-13: Prometheus + Grafana + structured logs + drift check"
```

Prompt:

```
Review code/day12 (Day 13 changes) + code/day13/. You are a Senior ML Platform engineer.

1. The four golden signals — which ones do I have, which am I missing? (saturation?)
2. histogram_quantile() over Prometheus histograms is approximate. Why, and how
   would I get exact percentiles in production?
3. Cardinality risk — my `predict_class_total` has `predicted_class` as a label
   with 102 distinct values. Is that fine? When would it explode the time-series count?
4. Drift via PSI on predicted-class distribution — is that detecting input drift
   or model drift? What's the difference?
5. Structured logs are JSON. Where would I ship them in real prod? (Loki? Cloudwatch? Splunk?)
6. The drift script polls Prometheus on demand. What's the production equivalent?
   (alerting rules in prometheus.yml).
7. Grafana dashboard is exported as JSON — how do I version-control it properly?

End: outline the smallest set of alerts I'd configure if this were my first
production deployment.
```

Save to `reviews/day13-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Add alerting rule.** Create `alert_rules.yml` for Prometheus that fires when `predict_latency_seconds:histogram_quantile(0.95) > 1.5` for 5 minutes. Reference it from `prometheus.yml`.
- **B: Add a request-ID middleware.** Each request gets a UUID; the ID flows through logs and is returned as a response header. Invaluable for debugging.
- **C: Read** Chip Huyen DMLS Ch. 9 (Continual Learning) sections 9.1 + 9.2. Write 5 bullets on retraining triggers.
- **D: Tag your prediction with model version.** Set `MODEL_VERSION = "v0.2.0-flowers"` env var, expose in `/health`, include in every log line. This becomes critical when you ship more than one model.
- **E: A `/predict_async` endpoint** that returns a job ID and writes to an in-memory queue. Add `/jobs/{id}` to fetch the result. (Toy version of the async-inference pattern.)

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Using a Counter where a Gauge is needed (or vice versa). Counter = strictly increasing (total requests). Gauge = current value (queue depth, in-memory items).
- **Pitfall 2:** High-cardinality labels (user IDs, request IDs as labels) → Prometheus storage explosion. Labels for *categorical buckets*, not unique IDs.
- **Pitfall 3:** Forgetting to call `REQUEST_LATENCY.observe(...)` in the `finally` block → failed requests don't contribute to latency stats → false sense of speed.
- **Pitfall 4:** Confusing data drift, label drift, and concept drift. Drift in predicted-class distribution is *not necessarily* a concept drift signal — your inputs might have shifted, or the production population genuinely changed.
- **Pitfall 5:** Putting a Prometheus client inside async loops without thread safety. The official `prometheus_client` is thread-safe; check if you swap in something else.

---

## Quiz (tomorrow 13:00)

`code/day13/quiz_answers.md`.

1. What's the difference between a Counter and a Histogram in Prometheus?
2. Explain `rate()` vs `increase()` — when to use each.
3. What's `histogram_quantile` doing under the hood — is it interpolating?
4. Three reasons your model's accuracy might drop in production without anyone touching the model.
5. PSI < 0.1 = stable, > 0.25 = drift. Why use log ratios — what's wrong with just `|p - q|`?
6. Continual training (retrain weekly) vs trigger-based retraining — when do you pick each?
7. A/B testing two models in production — what's the minimum traffic split, and what's the primary metric you'd guardrail?
8. Define "shadow mode" deployment.

---

→ Tomorrow at 09:00: open `DAILY/day-14.md`. **Capstone day.**
