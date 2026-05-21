# Day 7 — Production CNNs on CIFAR-10

**Industry context:** Real image classification is harder than MNIST. CIFAR-10 is the canonical "small but non-trivial" benchmark. Today you build a ResNet-style network with batch norm, data augmentation, a proper LR schedule, and W&B tracking — every piece of which appears in real production CV pipelines.

**Objective by EOD:**
- Build a ResNet-style CNN (~5M params) and train on CIFAR-10.
- Use data augmentation, BatchNorm, dropout, cosine LR.
- Log everything to W&B.
- **Target: ≥88% test accuracy.** (Stretch: ≥92%.)

**Dataset:** CIFAR-10 via `torchvision.datasets.CIFAR10` (auto-downloads, ~170MB).

---

## 09:00–10:00 — Theory

**Read:**
1. *Deep Residual Learning for Image Recognition* (He et al. 2015) — read **Abstract + Section 3.1, 3.2 + Figure 2 + Table 1**. Skip everything else. https://arxiv.org/abs/1512.03385.
2. *Batch Normalization* (Ioffe & Szegedy 2015) — read **Abstract + Section 3 (Algorithm 1) + first paragraph of Section 4**. https://arxiv.org/abs/1502.03167.
3. PyTorch *AutoAugment / Trivial Augment* docs: https://pytorch.org/vision/main/auto_examples/transforms/plot_transforms_illustrations.html (just the augmentations gallery — 5 min).

**Three ideas to walk away with:**
- A residual connection `y = F(x) + x` lets gradients flow even through hundreds of layers — that's how networks went from 19 layers (VGG) to 152+ (ResNet).
- BatchNorm shifts and scales pre-activations *per batch* during training; at inference it uses running statistics. This stabilizes optimization and accelerates training.
- Data augmentation is regularization. For CIFAR-10, random crop + horizontal flip alone get you most of the way; TrivialAugment is the modern default for the rest.

---

## 10:00–13:00 — Build A: data, augmentation, baseline

Create `code/day07/train.py`:

```python
"""Day 7 — ResNet-style CNN on CIFAR-10."""
from __future__ import annotations
import logging
from pathlib import Path

import torch
import torch.nn as nn
import torch.nn.functional as F
import wandb
from torch.utils.data import DataLoader
from torchvision import transforms as T
from torchvision.datasets import CIFAR10

ART = Path("artifacts/day07"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day07")
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"

CIFAR_MEAN = (0.4914, 0.4822, 0.4465)
CIFAR_STD = (0.2470, 0.2435, 0.2616)


def get_loaders(batch_size: int = 128, num_workers: int = 4):
    train_tf = T.Compose([
        T.RandomCrop(32, padding=4, padding_mode="reflect"),
        T.RandomHorizontalFlip(),
        T.TrivialAugmentWide(),                    # modern auto-augment
        T.ToTensor(),
        T.Normalize(CIFAR_MEAN, CIFAR_STD),
        T.RandomErasing(p=0.25),
    ])
    test_tf = T.Compose([T.ToTensor(), T.Normalize(CIFAR_MEAN, CIFAR_STD)])

    train_ds = CIFAR10("data/raw/day07", train=True, download=True, transform=train_tf)
    test_ds = CIFAR10("data/raw/day07", train=False, download=True, transform=test_tf)

    train_loader = DataLoader(train_ds, batch_size=batch_size, shuffle=True,
                              num_workers=num_workers, pin_memory=True, drop_last=True)
    test_loader = DataLoader(test_ds, batch_size=256, shuffle=False,
                             num_workers=num_workers, pin_memory=True)
    return train_loader, test_loader
```

### Architecture: a small ResNet (BasicBlock-style)

Append to `train.py`:

```python
class BasicBlock(nn.Module):
    expansion = 1
    def __init__(self, in_planes, planes, stride=1):
        super().__init__()
        self.conv1 = nn.Conv2d(in_planes, planes, 3, stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(planes)
        self.conv2 = nn.Conv2d(planes, planes, 3, stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(planes)
        self.shortcut = nn.Sequential()
        if stride != 1 or in_planes != planes * self.expansion:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_planes, planes * self.expansion, 1, stride=stride, bias=False),
                nn.BatchNorm2d(planes * self.expansion),
            )

    def forward(self, x):
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out += self.shortcut(x)
        return F.relu(out)


class ResNet(nn.Module):
    """Compact ResNet for 32x32 inputs, similar to ResNet-20."""

    def __init__(self, block=BasicBlock, num_blocks=(2, 2, 2, 2), num_classes=10):
        super().__init__()
        self.in_planes = 64
        self.conv1 = nn.Conv2d(3, 64, 3, stride=1, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.layer1 = self._make_layer(block, 64, num_blocks[0], stride=1)
        self.layer2 = self._make_layer(block, 128, num_blocks[1], stride=2)
        self.layer3 = self._make_layer(block, 256, num_blocks[2], stride=2)
        self.layer4 = self._make_layer(block, 512, num_blocks[3], stride=2)
        self.pool = nn.AdaptiveAvgPool2d(1)
        self.fc = nn.Linear(512 * block.expansion, num_classes)

    def _make_layer(self, block, planes, n_blocks, stride):
        strides = [stride] + [1] * (n_blocks - 1)
        layers = []
        for s in strides:
            layers.append(block(self.in_planes, planes, s))
            self.in_planes = planes * block.expansion
        return nn.Sequential(*layers)

    def forward(self, x):
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.layer1(out); out = self.layer2(out)
        out = self.layer3(out); out = self.layer4(out)
        out = self.pool(out).flatten(1)
        return self.fc(out)
```

### Training driver

```python
def evaluate(model, loader):
    model.eval()
    correct, total, loss_sum = 0, 0, 0.0
    with torch.no_grad():
        for x, y in loader:
            x, y = x.to(DEVICE, non_blocking=True), y.to(DEVICE, non_blocking=True)
            logits = model(x)
            loss_sum += F.cross_entropy(logits, y, reduction="sum").item()
            correct += (logits.argmax(1) == y).sum().item()
            total += y.size(0)
    return loss_sum / total, correct / total


def train_one_epoch(model, loader, optimizer, scheduler=None, scaler=None):
    model.train()
    running_loss, total = 0.0, 0
    for x, y in loader:
        x, y = x.to(DEVICE, non_blocking=True), y.to(DEVICE, non_blocking=True)
        optimizer.zero_grad(set_to_none=True)
        if scaler is not None:
            with torch.cuda.amp.autocast():
                logits = model(x); loss = F.cross_entropy(logits, y)
            scaler.scale(loss).backward()
            scaler.step(optimizer); scaler.update()
        else:
            logits = model(x); loss = F.cross_entropy(logits, y)
            loss.backward(); optimizer.step()
        if scheduler is not None: scheduler.step()
        running_loss += loss.item() * y.size(0); total += y.size(0)
    return running_loss / total


def main():
    torch.manual_seed(42)
    EPOCHS, LR, WD, BS = 40, 0.1, 5e-4, 128

    train_loader, test_loader = get_loaders(BS)
    model = ResNet().to(DEVICE)
    n_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
    log.info(f"Model params: {n_params/1e6:.2f}M")

    optimizer = torch.optim.SGD(model.parameters(), lr=LR, momentum=0.9,
                                weight_decay=WD, nesterov=True)
    scheduler = torch.optim.lr_scheduler.OneCycleLR(
        optimizer, max_lr=LR, epochs=EPOCHS, steps_per_epoch=len(train_loader))
    scaler = torch.cuda.amp.GradScaler() if DEVICE == "cuda" else None

    run = wandb.init(project="bootcamp", name="day07-resnet-cifar10",
                     config={"epochs": EPOCHS, "lr": LR, "wd": WD, "bs": BS,
                             "model": "ResNet-like-18", "params": n_params})

    best_acc = 0.0
    for epoch in range(1, EPOCHS + 1):
        tr_loss = train_one_epoch(model, train_loader, optimizer, scheduler, scaler)
        va_loss, va_acc = evaluate(model, test_loader)
        wandb.log({"train_loss": tr_loss, "test_loss": va_loss, "test_acc": va_acc,
                   "lr": scheduler.get_last_lr()[0], "epoch": epoch})
        log.info(f"E{epoch:02d}: tr={tr_loss:.3f}  te={va_loss:.3f}  acc={va_acc:.4f}")
        if va_acc > best_acc:
            best_acc = va_acc
            torch.save(model.state_dict(), ART / "resnet_best.pt")
    log.info(f"Best test acc: {best_acc:.4f}")
    wandb.summary["best_test_acc"] = best_acc
    run.finish()


if __name__ == "__main__":
    main()
```

Run: `uv run python code/day07/train.py`

**Time budget:**
- GPU (T4 or better): ~25 min for 40 epochs.
- CPU: skip — go to Kaggle or Colab. CPU will take 6+ hours and burn the day.

Use Kaggle Notebook (file → notebook → upload `train.py` → set Accelerator: GPU T4 x2 → run).

**Acceptance criteria:**
- Test accuracy ≥ 88% within 40 epochs.
- W&B run visible with loss + accuracy + LR curves.
- Best checkpoint saved.

---

## 13:00–14:00 — Lunch + Day-6 Quiz

---

## 14:00–17:00 — Build B: diagnostics, errors, TTA

While the model trains, build `code/day07/diagnose.py`:

### 1. Confusion matrix

```python
import torch, numpy as np
from torch.utils.data import DataLoader
from sklearn.metrics import confusion_matrix, classification_report
import seaborn as sns
import matplotlib.pyplot as plt
from train import ResNet, get_loaders, CIFAR_MEAN, CIFAR_STD, DEVICE

_, test_loader = get_loaders()
model = ResNet().to(DEVICE)
model.load_state_dict(torch.load("artifacts/day07/resnet_best.pt", map_location=DEVICE))
model.eval()

preds, labels = [], []
with torch.no_grad():
    for x, y in test_loader:
        x = x.to(DEVICE)
        preds.append(model(x).argmax(1).cpu().numpy())
        labels.append(y.numpy())
preds = np.concatenate(preds); labels = np.concatenate(labels)

classes = ["airplane", "automobile", "bird", "cat", "deer",
           "dog", "frog", "horse", "ship", "truck"]
cm = confusion_matrix(labels, preds)
sns.heatmap(cm, annot=True, fmt="d", xticklabels=classes, yticklabels=classes, cmap="Blues")
plt.tight_layout()
plt.savefig("artifacts/day07/confusion.png", dpi=120, bbox_inches="tight")
print(classification_report(labels, preds, target_names=classes))
```

CIFAR-10 confusable pairs: cat↔dog (largest), deer↔horse, ship↔airplane. Note which your model confuses most.

### 2. Worst predictions (error analysis)

Pull the 16 test images with highest confidence in the *wrong* class, plot them with true vs predicted labels. This is the single most useful 30 minutes of any DL project — you find labeling errors, weird inputs, and model biases.

### 3. Test-Time Augmentation (TTA)

```python
def predict_tta(model, x, num_aug=4):
    """Average predictions over horizontal flip + 4 random crops."""
    model.eval()
    preds = [F.softmax(model(x), dim=1)]
    preds.append(F.softmax(model(torch.flip(x, dims=[3])), dim=1))
    return torch.stack(preds).mean(0)
```

Re-evaluate with TTA. Typical gain: +0.3–0.7%. Free accuracy.

**Acceptance criteria:**
- Confusion matrix saved.
- 16-image error grid saved as `errors.png`.
- TTA accuracy reported alongside vanilla.

---

## 17:00–18:00 — Code Review

```bash
git add code/day07/ artifacts/day07/
git commit -m "day-7: CIFAR-10 ResNet, augmentation, TTA"
```

Prompt:

```
Review code/day07/. You are an applied research engineer for a vision team.

1. Augmentation pipeline — am I over- or under-augmenting? What's the standard
   "TrivialAugmentWide + RandomErasing" recipe's typical accuracy delta on CIFAR-10?
2. Optimizer choice — SGD+Nesterov vs AdamW for CNNs from scratch — defend the choice.
3. OneCycleLR — what makes it more effective than constant LR or step LR for short
   training runs (≤50 epochs)?
4. The architecture — is "ResNet-like-18" actually ResNet-18? What did I trim
   (initial 7x7 stem, maxpool) and why is that OK for 32x32 inputs?
5. AMP / GradScaler — am I using it correctly? What's the speedup I should expect?
6. Error analysis — beyond the confusion matrix, what would you ask me to plot
   to debug a "cat predicted as dog" failure?
7. TTA — should it go in the train script or only at inference? What's its cost?

End with the next 3 things to push test accuracy past 92%.
```

Save to `reviews/day07-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Week-1 Checkpoint + Hardening

Tomorrow is the start of week 2 — heavier topics. Before resting:

**Mandatory: Week 1 audit.**

- Run all four quizzes (Days 1, 2, 4, 5) again, **closed book**. Score yourself.
- Open W&B and verify Day 5 and Day 7 are both there.
- Push everything to GitHub:

```bash
git status
git log --oneline | head
git push
```

- Open the GitHub repo in your browser. Click around it as if you were a recruiter. The README is the welcome mat — does it exist? If not, write a short one tomorrow morning before Day 8 starts.

**Hardening (pick one only, you've earned a half-day off):**

- **A: Read** "Bag of Tricks for Image Classification with Convolutional Neural Networks" (He et al. 2018) — pick 2 tricks (mixup, label smoothing, cosine annealing variants) and add them. Aim for 92%+.
- **B: torch.compile.** Wrap your model in `model = torch.compile(model)` and benchmark speedup. (PyTorch 2.x.)
- **C: Save the model card.** Make `artifacts/day07/MODEL_CARD.md`: model name, dataset, train/test split, augmentations, optimizer, best metric, expected use, known failure modes. This is industry-standard practice.

---

## 21:00–21:30 — Journal + Week-1 Reflection

Write 1 paragraph at the bottom of `journal.md`:

- **Sharpest moment of week 1:** (one concept that made everything click)
- **Slowest day:** (which day cost you the most overtime — fix the cause)
- **What I'll do differently in week 2:** (one habit change)

---

## Common Pitfalls

- **Pitfall 1:** Forgetting to normalize images with CIFAR mean/std → model still trains but ~2% worse.
- **Pitfall 2:** Putting `BatchNorm` AFTER the bias term of Conv2d (it cancels out) → wasteful. Use `bias=False` on conv layers before BN.
- **Pitfall 3:** Augmentation on the *test* set → invalid evaluation. Test transforms must be deterministic (only resize/normalize/totensor).
- **Pitfall 4:** Calling `scheduler.step()` once per epoch when using `OneCycleLR` (which expects per-step). Read the scheduler docstring before using.
- **Pitfall 5:** `BatchNorm` + tiny batch sizes (≤8) → unstable statistics, model diverges. Use GroupNorm or a larger batch.

---

## Quiz (tomorrow 13:00)

`code/day07/quiz_answers.md`.

1. Why does adding a residual connection make very deep networks trainable?
2. BatchNorm has 4 learnable parameters per channel — name them and their roles.
3. CIFAR-10 ResNet uses `Conv2d(3, 64, 3)` as the stem, while ImageNet uses `Conv2d(3, 64, 7, stride=2)`. Why the difference?
4. In OneCycleLR, what does the "annihilation phase" do and why is it crucial?
5. You see training accuracy = 99% but test = 70%. List 4 fixes in order of impact.
6. Why does horizontal flip help for CIFAR-10 but is *harmful* for an OCR digit dataset?
7. Mixup combines two images at α ratio and their labels too. Why is this a regularizer?
8. Explain `AdaptiveAvgPool2d(1)` — what does it do, and why does it make a CNN input-size flexible?

---

→ **Week 2 starts tomorrow.** Open `DAILY/day-08.md` at 09:00 sharp. Get a real sleep.
