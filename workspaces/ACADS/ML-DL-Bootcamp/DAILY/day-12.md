# Day 12 — Model Serving (FastAPI + Docker)

**Industry context:** A model that isn't served can't make money. Every ML engineer at every shop ends up touching deployment. FastAPI is the python-native standard for API serving — type-safe, async, OpenAPI-doc'd. Docker is universal. Today you ship the Day-8 flower classifier as a real REST service that anyone with `docker run` can use.

**Objective by EOD:**
- Wrap the Day-8 flower classifier (TorchScript export) in a FastAPI service.
- Accept image uploads, return top-5 predictions JSON.
- Add request validation, timing, basic error handling.
- Containerize with Docker.
- Pass a 50-request load test (avg latency < 500ms on CPU).

---

## 09:00–10:00 — Theory

**Read:**
1. FastAPI **First steps**: https://fastapi.tiangolo.com/tutorial/first-steps/ (15 min).
2. FastAPI **Request files**: https://fastapi.tiangolo.com/tutorial/request-files/ (5 min).
3. FastAPI **Background tasks** + **Dependency injection**: https://fastapi.tiangolo.com/tutorial/dependencies/ (~15 min).
4. Chip Huyen *Designing ML Systems* — **Chapter 7 (Model Deployment) — sections 7.1–7.3 only**.

**Three ideas:**
- **Sync vs async endpoints.** FastAPI handlers can be `def` (sync, runs in threadpool) or `async def` (event loop). For PyTorch inference (CPU-blocking), `def` is correct — async doesn't make PyTorch any faster, but lets the server still handle other requests via threadpool.
- **Pydantic models.** Define request/response schemas with type hints; FastAPI auto-validates and auto-documents. Don't return raw dicts from a prod API — return Pydantic models.
- **Lifespan events.** Load model once at startup, not per request. Use FastAPI `lifespan` context manager.

---

## 10:00–13:00 — Build A: FastAPI service

Create `code/day12/`:

### File 1: `app/model.py`

```python
"""Load the Day-8 TorchScript model and expose predict()."""
from __future__ import annotations
from io import BytesIO
from pathlib import Path

import torch
import torch.nn.functional as F
from PIL import Image
from torchvision import transforms as T

MODEL_PATH = Path(__file__).parent.parent / "artifacts" / "resnet50_flowers.ts"
FLOWER_CLASSES = [...]   # paste your 102-class list from Day 8 here

_tf = T.Compose([
    T.Resize(256), T.CenterCrop(224),
    T.ToTensor(),
    T.Normalize((0.485, 0.456, 0.406), (0.229, 0.224, 0.225)),
])


class FlowerClassifier:
    def __init__(self, model_path: Path = MODEL_PATH, device: str | None = None):
        self.device = device or ("cuda" if torch.cuda.is_available() else "cpu")
        self.model = torch.jit.load(str(model_path), map_location=self.device).eval()
        # warm up
        with torch.no_grad():
            self.model(torch.zeros(1, 3, 224, 224, device=self.device))

    def predict(self, image_bytes: bytes, top_k: int = 5) -> list[dict]:
        img = Image.open(BytesIO(image_bytes)).convert("RGB")
        x = _tf(img).unsqueeze(0).to(self.device)
        with torch.no_grad():
            probs = F.softmax(self.model(x), dim=1)[0]
        values, indices = torch.topk(probs, k=top_k)
        return [
            {"class": FLOWER_CLASSES[i.item()], "prob": float(v.item())}
            for v, i in zip(values, indices)
        ]
```

Copy your Day-8 TorchScript export to `code/day12/artifacts/resnet50_flowers.ts`.

### File 2: `app/schemas.py`

```python
from pydantic import BaseModel, Field

class Prediction(BaseModel):
    class_: str = Field(..., alias="class")
    prob: float = Field(..., ge=0.0, le=1.0)

    class Config:
        allow_population_by_field_name = True

class PredictResponse(BaseModel):
    filename: str
    inference_ms: float
    predictions: list[Prediction]

class HealthResponse(BaseModel):
    status: str
    device: str
    model: str
```

### File 3: `app/main.py`

```python
"""FastAPI service for the flower classifier."""
from __future__ import annotations
import logging
import time
from contextlib import asynccontextmanager

import structlog
from fastapi import FastAPI, File, HTTPException, UploadFile
from fastapi.responses import JSONResponse

from app.model import FlowerClassifier, MODEL_PATH
from app.schemas import HealthResponse, PredictResponse

logging.basicConfig(level=logging.INFO)
log = structlog.get_logger()

_state: dict = {}


@asynccontextmanager
async def lifespan(app: FastAPI):
    log.info("model.load.start", path=str(MODEL_PATH))
    _state["clf"] = FlowerClassifier()
    log.info("model.load.done", device=_state["clf"].device)
    yield
    _state.clear()


app = FastAPI(title="Flower Classifier", version="0.1.0", lifespan=lifespan)


@app.get("/health", response_model=HealthResponse)
def health():
    clf = _state.get("clf")
    if clf is None:
        raise HTTPException(503, "model not loaded yet")
    return HealthResponse(status="ok", device=clf.device, model="resnet50-flowers")


MAX_BYTES = 10 * 1024 * 1024  # 10MB
ALLOWED_TYPES = {"image/jpeg", "image/png", "image/webp"}


@app.post("/predict", response_model=PredictResponse)
async def predict(file: UploadFile = File(...)):
    if file.content_type not in ALLOWED_TYPES:
        raise HTTPException(415, f"Unsupported type: {file.content_type}")
    data = await file.read()
    if len(data) > MAX_BYTES:
        raise HTTPException(413, f"Payload too large: {len(data)} bytes")
    if len(data) < 100:
        raise HTTPException(400, "Image data too small")
    try:
        t0 = time.perf_counter()
        preds = _state["clf"].predict(data, top_k=5)
        dt_ms = (time.perf_counter() - t0) * 1000
    except Exception as e:
        log.exception("predict.error", filename=file.filename)
        raise HTTPException(500, f"inference failed: {e!s}")
    return PredictResponse(
        filename=file.filename or "unknown",
        inference_ms=round(dt_ms, 2),
        predictions=[{"class": p["class"], "prob": p["prob"]} for p in preds],
    )
```

### File 4: `pyproject.toml` / requirements

```bash
cd code/day12
cat > requirements.txt <<'EOF'
fastapi>=0.110
uvicorn[standard]>=0.30
pydantic>=2.7
structlog>=24.1
torch>=2.5
torchvision>=0.20
Pillow>=10.0
python-multipart>=0.0.7
EOF
```

### Run locally

```bash
cd code/day12
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Open http://localhost:8000/docs — FastAPI gives you a free interactive Swagger UI. Test the `/predict` endpoint by uploading a flower image.

```bash
curl -X POST -F "file=@../day08/sample_flower.jpg" http://localhost:8000/predict | jq
```

**Acceptance criteria:**
- `/health` returns `{"status": "ok", "device": "...", "model": "resnet50-flowers"}`.
- `/predict` accepts an image file and returns top-5 predictions in JSON.
- Pydantic validation rejects non-image content-types with 415.
- Files > 10MB rejected with 413.
- OpenAPI docs render at `/docs`.

---

## 13:00–14:00 — Lunch + Day-11 Quiz

---

## 14:00–17:00 — Build B: Docker + load test

### 1. Dockerfile

`code/day12/Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# System deps for Pillow + curl for healthcheck
RUN apt-get update && apt-get install -y --no-install-recommends \
      libjpeg-dev zlib1g-dev curl && \
    rm -rf /var/lib/apt/lists/*

# Install dependencies first (better layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app + artifacts
COPY app/ ./app/
COPY artifacts/ ./artifacts/

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=20s --retries=3 \
    CMD curl -fsS http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

`code/day12/.dockerignore`:

```
__pycache__
*.pyc
.venv
.pytest_cache
.git
*.md
```

### 2. Build and run

```bash
cd code/day12
docker build -t flower-classifier:0.1.0 .
docker run --rm -p 8000:8000 --name flower flower-classifier:0.1.0
```

In another terminal:

```bash
curl -X POST -F "file=@/path/to/flower.jpg" http://localhost:8000/predict | jq
```

Image size check:

```bash
docker images flower-classifier
```

Aim for under 1.5GB (PyTorch is large; that's the floor).

### 3. Load test

Install Locust:

```bash
uv add locust
```

`code/day12/load_test.py`:

```python
import os
from pathlib import Path
from locust import HttpUser, task, between

IMG = Path(os.environ.get("LOAD_TEST_IMG", "sample.jpg")).read_bytes()

class FlowerUser(HttpUser):
    wait_time = between(0.1, 0.5)

    @task
    def predict(self):
        files = {"file": ("flower.jpg", IMG, "image/jpeg")}
        self.client.post("/predict", files=files)
```

Run:

```bash
LOAD_TEST_IMG=/path/to/your/flower.jpg \
  uv run locust -f code/day12/load_test.py --host http://localhost:8000 \
  --users 10 --spawn-rate 2 --run-time 1m --headless
```

This sends ~10 concurrent users for 60 seconds. Locust prints a final summary.

**Acceptance criteria:**
- Total requests ≥ 50.
- 0 failures.
- p95 latency < 1500 ms (CPU is slow; if GPU enabled, < 300 ms).
- Median latency < 500 ms (CPU) or < 100 ms (GPU).

If failures: check container logs (`docker logs flower`). Common: OOM (raise instance memory), threadpool exhausted (raise `--workers 4`).

### 4. Tests

`code/day12/tests/test_api.py`:

```python
from pathlib import Path
import pytest
from fastapi.testclient import TestClient
from app.main import app

@pytest.fixture(scope="module")
def client():
    with TestClient(app) as c:
        yield c

def test_health(client):
    r = client.get("/health")
    assert r.status_code == 200
    assert r.json()["status"] == "ok"

def test_predict_accepts_jpeg(client):
    img = Path("sample.jpg").read_bytes()
    r = client.post("/predict", files={"file": ("flower.jpg", img, "image/jpeg")})
    assert r.status_code == 200
    body = r.json()
    assert body["filename"] == "flower.jpg"
    assert 0 < body["inference_ms"] < 5000
    assert len(body["predictions"]) == 5
    assert all(0 <= p["prob"] <= 1 for p in body["predictions"])

def test_predict_rejects_text(client):
    r = client.post("/predict", files={"file": ("f.txt", b"hello", "text/plain")})
    assert r.status_code == 415

def test_predict_rejects_large_payload(client):
    huge = b"x" * (11 * 1024 * 1024)
    r = client.post("/predict", files={"file": ("big.jpg", huge, "image/jpeg")})
    assert r.status_code == 413
```

Run: `uv run pytest code/day12/ -v`. All 4 must pass.

---

## 17:00–18:00 — Code Review

```bash
git add code/day12/
git commit -m "day-12: FastAPI flower classifier service + Docker + load test"
```

Prompt:

```
Review code/day12/. You are an SRE who's seen 100 ML services go to prod.

1. Sync vs async — I used `async def predict` but inference itself is blocking.
   Did I get the trade-off right? Should I use `def` instead?
2. The model is loaded in a global `_state` dict — is this thread-safe with
   uvicorn workers=2? What changes with workers=8?
3. Error handling — I catch all exceptions on inference. Is that hiding bugs?
   What should I let bubble up vs return 500?
4. Pydantic validation — anything I missed (file extension, image dimensions
   sanity, content-type spoofing)?
5. Dockerfile — is the layer order right for build cache reuse?
6. Image size — what's the minimum I could achieve while staying CPU-portable?
   (Hint: distroless? slim → alpine? CPU-only torch wheel?)
7. Load test result — what does p95 < 1500ms on CPU tell us about
   batch-vs-single inference for this workload?

End: 3 things I should add before this is "real" production.
```

Save to `reviews/day12-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Batch inference endpoint.** Add `/predict_batch` that accepts multiple files in a single request, runs them as a batch through the model. Compare throughput vs single-shot.
- **B: Slim the image.** Switch to `python:3.11-slim` (already done) + install `torch` with `--index-url https://download.pytorch.org/whl/cpu` to skip CUDA libs (saves ~700MB). Rebuild and confirm size.
- **C: Add request ID + structured logs.** Use `structlog` with a request-ID middleware. Each log line should include the request ID. This is invaluable in production.
- **D: docker-compose.yaml** for local stack (just the API for now; expand tomorrow). Even a one-service compose file is good practice.
- **E: Read** Chip Huyen DMLS Ch. 7.3 ("Model Compression") and write 3 bullets on how it'd apply to your flower model (quantization, pruning, distillation).

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Loading the model inside the request handler → loads on every request → 10x latency. Always load at startup via `lifespan`.
- **Pitfall 2:** Using `Pydantic v1` syntax in a `pydantic>=2` project. `Config:` blocks, `class Config:` → `model_config = ConfigDict(...)`. We used v1 syntax above for clarity; v2 prefers `ConfigDict`.
- **Pitfall 3:** Forgetting `python-multipart` → `UploadFile` quietly stops working.
- **Pitfall 4:** Running uvicorn with `--reload` in production. Only for dev.
- **Pitfall 5:** Not setting `WORKERS` env / `--workers`. Default of 1 means single-threaded under load. For CPU inference, set `workers ≈ CPU cores`.

---

## Quiz (tomorrow 13:00)

`code/day12/quiz_answers.md`.

1. Why `async def` for the predict endpoint even though inference is CPU-bound?
2. What does `Pydantic` actually do at runtime — is it validating types? Coercing? Both?
3. Explain the FastAPI lifespan context manager and what code goes inside vs outside.
4. What's the difference between `uvicorn --workers 4` and `gunicorn -w 4 -k uvicorn.workers.UvicornWorker`?
5. Your latency p99 is 5x p50 (e.g., 50ms median, 250ms p99). Two plausible causes?
6. TorchScript vs ONNX — when do you use each for deployment?
7. Why is image classification a great first ML service to deploy but text generation is harder?
8. You're asked to deploy this to AWS — what's the simplest path (in your opinion)? (Hint: ECS Fargate with the same Dockerfile.)

---

→ Tomorrow at 09:00: open `DAILY/day-13.md`. **MLOps + monitoring.**
