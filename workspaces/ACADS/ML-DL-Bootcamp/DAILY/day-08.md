# Day 8 — Transfer Learning

**Industry context:** You will almost never train a vision model from scratch in industry. Transfer learning — take a pretrained ImageNet backbone, swap the head, fine-tune on your data — is *the* CV workflow. Today you do it properly, including the two-stage freeze/unfreeze recipe that separates Kaggle copy-pasters from production engineers.

**Objective by EOD:**
- Fine-tune a pretrained ResNet50 on Oxford Flowers 102 (102 classes, 1020 train / 1020 val / 6149 test).
- Use the two-stage protocol: freeze backbone → train head; unfreeze backbone → fine-tune with discriminative LRs.
- **Target: ≥92% test accuracy.** (Reachable with ResNet50; ≥95% with stretch.)

**Dataset:** Oxford Flowers 102 — `torchvision.datasets.Flowers102` (downloads automatically, ~330MB).

---

## 09:00–10:00 — Theory

**Read:**
1. PyTorch tutorial — *Transfer Learning for Computer Vision*: https://pytorch.org/tutorials/beginner/transfer_learning_tutorial.html. Read all the way to the bottom. You'll replicate the two strategies it lists.
2. Read **just sections 1, 4, 5** of *"How transferable are features in deep neural networks?"* (Yosinski et al. 2014): https://arxiv.org/abs/1411.1792. Take-away: earlier layers transfer better than later layers; transfer ability drops with semantic distance between source and target.
3. timm model gallery — skim https://huggingface.co/timm/models (just see what's in the modern menu).

**Two recipes to memorize:**

**Recipe A — Feature extractor.** Freeze all conv layers, train only the new classification head. Fast, small risk of overfitting, used when target dataset is small (≲ 1k images) and similar to source.

**Recipe B — Fine-tuning.** Train head first (a few epochs), then unfreeze backbone with a 10x lower LR. Used when target dataset is medium-large (≳ 5k) or domain-distant from ImageNet.

Today you do both, compare them honestly.

---

## 10:00–13:00 — Build A: feature-extractor baseline

Create `code/day08/train.py`:

```python
"""Day 8 — Transfer learning on Oxford Flowers 102 with ResNet50."""
from __future__ import annotations
import logging
from pathlib import Path

import torch
import torch.nn as nn
import torch.nn.functional as F
import torchvision.models as tvm
import wandb
from torch.utils.data import DataLoader
from torchvision import transforms as T
from torchvision.datasets import Flowers102

ART = Path("artifacts/day08"); ART.mkdir(parents=True, exist_ok=True)
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day08")

IMNET_MEAN = (0.485, 0.456, 0.406)
IMNET_STD = (0.229, 0.224, 0.225)


def get_loaders(batch_size: int = 32, workers: int = 4):
    train_tf = T.Compose([
        T.Resize(256), T.RandomResizedCrop(224, scale=(0.7, 1.0)),
        T.RandomHorizontalFlip(),
        T.TrivialAugmentWide(),
        T.ToTensor(),
        T.Normalize(IMNET_MEAN, IMNET_STD),
    ])
    eval_tf = T.Compose([
        T.Resize(256), T.CenterCrop(224),
        T.ToTensor(), T.Normalize(IMNET_MEAN, IMNET_STD),
    ])
    train_ds = Flowers102("data/raw/day08", split="train", download=True, transform=train_tf)
    val_ds = Flowers102("data/raw/day08", split="val", download=True, transform=eval_tf)
    test_ds = Flowers102("data/raw/day08", split="test", download=True, transform=eval_tf)

    return (DataLoader(train_ds, batch_size, shuffle=True, num_workers=workers, pin_memory=True),
            DataLoader(val_ds, batch_size, shuffle=False, num_workers=workers, pin_memory=True),
            DataLoader(test_ds, batch_size, shuffle=False, num_workers=workers, pin_memory=True))


def build_resnet50(num_classes: int = 102, freeze_backbone: bool = True) -> nn.Module:
    weights = tvm.ResNet50_Weights.IMAGENET1K_V2     # use the better V2 weights
    model = tvm.resnet50(weights=weights)
    if freeze_backbone:
        for p in model.parameters():
            p.requires_grad = False
    model.fc = nn.Linear(model.fc.in_features, num_classes)
    return model


@torch.no_grad()
def evaluate(model, loader):
    model.eval()
    correct, total = 0, 0
    for x, y in loader:
        x, y = x.to(DEVICE), y.to(DEVICE)
        correct += (model(x).argmax(1) == y).sum().item()
        total += y.size(0)
    return correct / total


def train_loop(model, loaders, epochs, optimizer, scheduler, scaler, run_name: str):
    train_loader, val_loader, test_loader = loaders
    best = 0.0
    for ep in range(1, epochs + 1):
        model.train()
        for x, y in train_loader:
            x, y = x.to(DEVICE, non_blocking=True), y.to(DEVICE, non_blocking=True)
            optimizer.zero_grad(set_to_none=True)
            if scaler is not None:
                with torch.cuda.amp.autocast():
                    loss = F.cross_entropy(model(x), y)
                scaler.scale(loss).backward(); scaler.step(optimizer); scaler.update()
            else:
                loss = F.cross_entropy(model(x), y); loss.backward(); optimizer.step()
        if scheduler is not None: scheduler.step()
        va = evaluate(model, val_loader)
        wandb.log({f"{run_name}/val_acc": va, f"{run_name}/lr": optimizer.param_groups[0]["lr"]})
        log.info(f"[{run_name}] epoch {ep:02d} val_acc={va:.4f}")
        if va > best:
            best = va
            torch.save(model.state_dict(), ART / f"{run_name}_best.pt")
    test_acc = evaluate_best(model, test_loader, run_name)
    wandb.log({f"{run_name}/test_acc": test_acc, f"{run_name}/best_val_acc": best})
    return best, test_acc


def evaluate_best(model, test_loader, run_name):
    model.load_state_dict(torch.load(ART / f"{run_name}_best.pt", map_location=DEVICE))
    return evaluate(model, test_loader)


def main():
    torch.manual_seed(42)
    loaders = get_loaders()
    scaler = torch.cuda.amp.GradScaler() if DEVICE == "cuda" else None

    run = wandb.init(project="bootcamp", name="day08-flowers-transfer",
                     config={"backbone": "resnet50_v2", "img_size": 224})

    # --- Stage A: frozen backbone, head only ---
    model = build_resnet50(freeze_backbone=True).to(DEVICE)
    head_params = [p for p in model.parameters() if p.requires_grad]
    optim_a = torch.optim.AdamW(head_params, lr=1e-3, weight_decay=1e-4)
    sched_a = torch.optim.lr_scheduler.CosineAnnealingLR(optim_a, T_max=5)
    best_a, test_a = train_loop(model, loaders, epochs=5,
                                optimizer=optim_a, scheduler=sched_a, scaler=scaler,
                                run_name="stage_a_frozen")
    log.info(f"Stage A (frozen): val_best={best_a:.4f}  test={test_a:.4f}")

    # --- Stage B: unfreeze backbone, discriminative LRs ---
    for p in model.parameters():
        p.requires_grad = True
    backbone_params = [p for n, p in model.named_parameters() if not n.startswith("fc.")]
    head_params     = [p for n, p in model.named_parameters() if n.startswith("fc.")]
    optim_b = torch.optim.AdamW([
        {"params": backbone_params, "lr": 1e-5},
        {"params": head_params,     "lr": 1e-4},
    ], weight_decay=1e-4)
    sched_b = torch.optim.lr_scheduler.CosineAnnealingLR(optim_b, T_max=10)
    best_b, test_b = train_loop(model, loaders, epochs=10,
                                optimizer=optim_b, scheduler=sched_b, scaler=scaler,
                                run_name="stage_b_finetune")
    log.info(f"Stage B (finetune): val_best={best_b:.4f}  test={test_b:.4f}")

    wandb.summary["final_test_acc"] = test_b
    run.finish()


if __name__ == "__main__":
    main()
```

Run on GPU (Kaggle Notebook recommended):

```bash
uv run python code/day08/train.py
```

**Time budget:** ~25–40 min on Kaggle T4. ~12 min on a 3090.

**Acceptance criteria:**
- Stage A test accuracy printed (~85–90%).
- Stage B test accuracy ≥ 92%. The bar.
- W&B run shows BOTH stages with clear improvement curve.
- Both checkpoints saved.

---

## 13:00–14:00 — Lunch + Day-7 Quiz

---

## 14:00–17:00 — Build B: error analysis + predict on your own image

### 1. Error analysis (15 min)

`code/day08/errors.py`. Load test set predictions, find top-25 misclassifications by confidence-in-wrong-class. Use the Flowers102 class names (`["pink primrose", "hard-leaved pocket orchid", ...]` — get from `Flowers102.classes` if available, or from the dataset README).

For each, show: image, true label, predicted label, predicted prob. Save as `errors_grid.png` (5x5).

### 2. Single-image inference script (45 min)

`code/day08/predict_one.py`:

```python
"""Predict species for a single image. Usage: python predict_one.py path/to/img.jpg"""
import sys
from pathlib import Path

import torch
import torch.nn.functional as F
import torchvision.models as tvm
from PIL import Image
from torchvision import transforms as T

# These are the 102 flower class names. Look them up from:
# https://www.robots.ox.ac.uk/~vgg/data/flowers/102/categories.html
FLOWER_CLASSES = [...]    # Fill this in. 102 entries.

DEVICE = "cuda" if torch.cuda.is_available() else "cpu"

model = tvm.resnet50()
model.fc = torch.nn.Linear(model.fc.in_features, 102)
ckpt = torch.load("artifacts/day08/stage_b_finetune_best.pt", map_location=DEVICE)
model.load_state_dict(ckpt); model.to(DEVICE).eval()

tf = T.Compose([T.Resize(256), T.CenterCrop(224), T.ToTensor(),
                T.Normalize((0.485, 0.456, 0.406), (0.229, 0.224, 0.225))])

img = Image.open(sys.argv[1]).convert("RGB")
x = tf(img).unsqueeze(0).to(DEVICE)
with torch.no_grad():
    probs = F.softmax(model(x), dim=1)[0]
top5 = torch.topk(probs, k=5)
print(f"Top-5 predictions for {sys.argv[1]}:")
for prob, idx in zip(top5.values, top5.indices):
    print(f"  {FLOWER_CLASSES[idx.item()]:35s}  {prob.item()*100:5.2f}%")
```

Test it on **3 real images** — download three flower images from a Google image search, save to `data/raw/day08/test_imgs/`, run the script on each. Note whether predictions are sensible.

### 3. Save a TorchScript export for Day 12 (1 hour)

```python
# In a separate file: code/day08/export.py
import torch, torchvision.models as tvm

model = tvm.resnet50()
model.fc = torch.nn.Linear(model.fc.in_features, 102)
model.load_state_dict(torch.load("artifacts/day08/stage_b_finetune_best.pt", map_location="cpu"))
model.eval()

example = torch.randn(1, 3, 224, 224)
scripted = torch.jit.trace(model, example)
scripted.save("artifacts/day08/resnet50_flowers.ts")
print("TorchScript saved.")
```

Verify: load and predict to confirm identical output.

```python
scripted = torch.jit.load("artifacts/day08/resnet50_flowers.ts")
print(scripted(example)[0, :5])
```

This file is what you'll deploy on Day 12.

**Acceptance criteria for Build B:**
- `errors_grid.png` saved.
- `predict_one.py` works on at least 3 random downloaded flower images.
- `resnet50_flowers.ts` saved and verified.

---

## 17:00–18:00 — Code Review

```bash
git add code/day08/ artifacts/day08/
git commit -m "day-8: ResNet50 transfer learning on Flowers102"
```

Prompt:

```
Review code/day08/. You are a CV team lead.

1. Two-stage vs single-stage fine-tuning — when does single-stage (just unfreeze
   everything, train end-to-end) work as well? When does it fail catastrophically?
2. Discriminative LRs (backbone 1e-5, head 1e-4) — what's the principle, and how
   would I generalize this to 4+ groups (e.g., per-layer LR)?
3. Image resolution — I use 224. Flowers benefit from higher resolution. What's
   the tradeoff, and what's the right path (multi-crop? larger input? EfficientNetV2?)?
4. Augmentation — TrivialAugment is generic. Should I add domain-specific augs
   (color jitter on hue? flower-friendly cutouts)?
5. The 1020 train / 6149 test split is unusual (more test than train). What
   does this imply about generalization claims?
6. TorchScript trace vs script — I traced. When does trace silently break?

End: 3 changes to push test accuracy from ~92 → ~96.
```

Save to `reviews/day08-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: timm backbones.** Replace ResNet50 with `timm.create_model("efficientnet_b3.ra2_in1k", pretrained=True, num_classes=102)`. Re-run Stage B. Typical gain: +1–3% with similar compute.
- **B: Test-time augmentation.** Re-implement TTA from Day 7. Apply to Stage B test eval.
- **C: Gradient checkpointing** — if you ran out of memory, add `torch.utils.checkpoint`. Else, skip.
- **D: Read** "Sharpness-Aware Minimization (SAM)" abstract + figure 1 — a regularizer used in modern CV (https://arxiv.org/abs/2010.01412). 10 minutes.
- **E: Build `MODEL_CARD.md`** in `artifacts/day08/`: dataset, train/val/test counts, augmentations, optimizer, LR schedule, final metric, known failure modes (e.g., flowers under unusual lighting).

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Forgetting to set `requires_grad = False` and assuming the backbone is frozen because you didn't pass its params to the optimizer. The optimizer doesn't update params not in its list, but the model still computes gradients (wasting memory). Setting `requires_grad = False` is correct + efficient.
- **Pitfall 2:** Normalizing with CIFAR mean/std instead of ImageNet mean/std for a pretrained-on-ImageNet backbone. The model's first layer expects ImageNet stats.
- **Pitfall 3:** Replacing `model.fc` but forgetting that some torchvision models have `model.classifier` instead. Always print `model` before replacing.
- **Pitfall 4:** Unfreezing the backbone with the same LR as the head → catastrophic forgetting, val accuracy drops by 10+%.
- **Pitfall 5:** Calling `model.train()` on a model whose BN layers shouldn't track new statistics (e.g., very small batch fine-tuning). Use `BatchNorm.eval()` selectively, or freeze BN running stats.

---

## Quiz (tomorrow 13:00)

`code/day08/quiz_answers.md`.

1. Why is the typical fine-tuning LR 10–100x smaller than from-scratch training LR?
2. Explain "catastrophic forgetting" in transfer learning in 2 sentences.
3. What's the difference between freezing parameters (`requires_grad=False`) and just not passing them to the optimizer?
4. What is `ResNet50_Weights.IMAGENET1K_V2` and why is it better than `V1`?
5. You see val_acc go up then drop sharply at epoch 8 of fine-tuning. What's the most likely cause?
6. When should you use `RandomResizedCrop(224)` vs `Resize(256)+CenterCrop(224)`?
7. Discriminative LR = different LR per layer group. Why does this work better than a single LR?
8. Why might "TorchScript trace" silently produce a wrong model, and how do you check?

---

→ Tomorrow at 09:00: open `DAILY/day-09.md`. **Brace yourself — tomorrow you build a transformer from scratch.**
