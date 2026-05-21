# Day 0 — Environment Setup

Do this **once**, before Day 1. Budget: 90 minutes. If anything blocks for more than 15 minutes, ping Claude with the exact error.

## Step 1 — Install `uv` (5 min)

`uv` replaces pip, venv, virtualenv, pyenv, and poetry. It's 10–100x faster than pip and reproducible.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# Restart shell or:
source $HOME/.local/bin/env

# Verify
uv --version    # should print 0.5.x or newer
```

## Step 2 — Create the project (5 min)

```bash
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
uv init --python 3.11 --no-readme
uv venv
source .venv/bin/activate
```

## Step 3 — Install all dependencies in one go (10 min)

```bash
uv add \
  "numpy>=1.26" "pandas>=2.2" "polars>=1.0" "pyarrow>=15" \
  "matplotlib>=3.8" "seaborn>=0.13" "plotly>=5.20" \
  "scikit-learn>=1.5" "imbalanced-learn>=0.12" \
  "xgboost>=2.1" "lightgbm>=4.3" "shap>=0.45" \
  "optuna>=3.6" "wandb>=0.17" \
  "torch>=2.5" "torchvision>=0.20" "torchaudio>=2.5" \
  "transformers>=4.40" "datasets>=2.20" "accelerate>=0.30" "peft>=0.11" "evaluate>=0.4" \
  "sentence-transformers>=3.0" "faiss-cpu>=1.8" \
  "anthropic>=0.34" \
  "fastapi>=0.110" "uvicorn[standard]>=0.30" "pydantic>=2.7" \
  "prometheus_client>=0.20" "structlog>=24.1" \
  "jupyterlab>=4.2" "ipywidgets>=8.1" \
  "pytest>=8.2" "ruff>=0.5" "black>=24.4" \
  "tqdm>=4.66" "rich>=13.7" "python-dotenv>=1.0" "httpx>=0.27" \
  "kaggle>=1.6"
```

If you have an NVIDIA GPU, additionally:

```bash
# Verify CUDA-enabled torch
python -c "import torch; print(torch.cuda.is_available(), torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
# If False but you have an NVIDIA GPU, install the CUDA build:
uv pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

## Step 4 — Configure API keys (10 min)

Create `.env` in the workspace root (already gitignored — see below):

```bash
cat > /home/agush/mainWS/workspaces/ML-DL-Bootcamp/.env <<'EOF'
ANTHROPIC_API_KEY=sk-ant-...
WANDB_API_KEY=...
KAGGLE_USERNAME=...
KAGGLE_KEY=...
EOF
```

- **Anthropic key:** https://console.anthropic.com/settings/keys → create one with $20 credit. Used Day 11.
- **W&B key:** https://wandb.ai/authorize. Free tier. Used Day 5, Day 7, Day 8, Day 10.
- **Kaggle key:** https://www.kaggle.com/settings → "Create New API Token" downloads `kaggle.json`. Used Day 1, Day 3, Day 4. Also place it at `~/.kaggle/kaggle.json` and `chmod 600`.

```bash
mkdir -p ~/.kaggle && mv ~/Downloads/kaggle.json ~/.kaggle/ && chmod 600 ~/.kaggle/kaggle.json
```

## Step 5 — Create `.gitignore` (2 min)

```bash
cat > /home/agush/mainWS/workspaces/ML-DL-Bootcamp/.gitignore <<'EOF'
.venv/
__pycache__/
.ipynb_checkpoints/
*.pyc
.env
.DS_Store
wandb/
mlruns/
data/raw/
data/processed/
*.parquet
*.csv.gz
checkpoints/
*.pth
*.pt
*.bin
.pytest_cache/
.ruff_cache/
node_modules/
EOF
```

## Step 6 — Initialize git + repo structure (5 min)

```bash
cd /home/agush/mainWS/workspaces/ML-DL-Bootcamp
git init -b main
mkdir -p code/{day01,day02,day03,day04,day05,day06,day07,day08,day09,day10,day11,day12,day13,day14}
mkdir -p data/raw data/processed checkpoints
touch journal.md
git add .
git commit -m "day-0: bootcamp scaffold"
```

Push to GitHub (create a fresh public repo `ml-bootcamp` on github.com first):

```bash
git remote add origin git@github.com:<your-handle>/ml-bootcamp.git
git push -u origin main
```

## Step 7 — VSCode setup (10 min)

Install these extensions (skip if already installed):

- `ms-python.python`
- `ms-toolsai.jupyter`
- `charliermarsh.ruff`
- `ms-azuretools.vscode-docker`

Set Python interpreter: `Ctrl+Shift+P → Python: Select Interpreter → .venv/bin/python`.

Add to `.vscode/settings.json`:

```json
{
  "python.defaultInterpreterPath": ".venv/bin/python",
  "editor.formatOnSave": true,
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.codeActionsOnSave": { "source.fixAll.ruff": "explicit" }
  }
}
```

## Step 8 — Smoke test (10 min)

Create `code/day00_smoke.py`:

```python
import numpy as np
import pandas as pd
import sklearn
import torch
import xgboost as xgb
from transformers import AutoTokenizer

print(f"numpy={np.__version__}")
print(f"pandas={pd.__version__}")
print(f"sklearn={sklearn.__version__}")
print(f"torch={torch.__version__}, cuda={torch.cuda.is_available()}")
print(f"xgboost={xgb.__version__}")

tok = AutoTokenizer.from_pretrained("distilbert-base-uncased")
ids = tok("hello world", return_tensors="pt")
print(f"tokenizer OK: input_ids shape = {ids['input_ids'].shape}")

x = torch.randn(2, 3)
print(f"torch tensor: {x.sum().item():.3f}")
```

Run:

```bash
uv run python code/day00_smoke.py
```

All five imports must succeed. If any fail, fix before Day 1.

## Step 9 — Set up W&B (5 min)

```bash
uv run wandb login   # paste key when prompted
```

Verify:

```bash
uv run python -c "import wandb; r = wandb.init(project='bootcamp', name='setup-smoke', mode='online'); wandb.log({'test': 1}); r.finish()"
```

Visit https://wandb.ai/<your-username>/bootcamp and confirm the run appears. You should see a single value plotted.

## Step 10 — Kaggle smoke test (3 min)

```bash
uv run kaggle competitions list -s "titanic"
```

Should return a table including the Titanic competition. If you get a 401, recheck the kaggle.json placement and permissions.

## Step 11 — Initial commit (2 min)

```bash
git add .
git commit -m "day-0: smoke tests pass"
git push
```

## ✅ Done

If all 11 steps passed, you are ready for Day 1. Close everything, sleep well, and open `DAILY/day-01.md` at 09:00 tomorrow.

**Time check:** Should have taken ≤90 min. Anything longer is wasted setup. If a step is blocking, ping Claude with the literal error message — don't grind.
