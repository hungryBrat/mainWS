# Day 6 — PyTorch From Scratch (MNIST)

**Industry context:** Every modern ML interviewer will ask you to walk through a PyTorch training loop. Not from `Trainer.fit()` — from `model.train(); optimizer.zero_grad(); loss.backward(); optimizer.step()`. Today you write that loop by hand. No `pytorch-lightning`, no `accelerate`, no `Trainer`. Once you've done it once, you understand every deep learning library that builds on top.

**Objective by EOD:**
- Build a 3-layer MLP and a small CNN on MNIST from scratch.
- Hand-write the training loop, eval loop, and checkpointing.
- Implement a custom `Dataset` (even though MNIST has a torchvision one — you need to know how).
- Hit **≥98% test accuracy** with the CNN.

---

## 09:00–10:00 — Theory

**Read / watch (this is the only video this week):**

1. PyTorch tutorial — *Training a Classifier*: https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html. Read all 5 cells. Don't run yet; you'll write your own.
2. PyTorch *Autograd mechanics* (just §1–3): https://pytorch.org/docs/stable/notes/autograd.html.
3. *A Recipe for Training Neural Networks* by Andrej Karpathy: https://karpathy.github.io/2019/04/25/recipe/. Read the whole thing. **This is the most important 20 minutes of the bootcamp.** It will save you from 100 bad models over the next decade.

**Five things to internalize from Karpathy's recipe:**
- Always start by *manually inspecting the data*.
- Set up an end-to-end skeleton (load → forward → loss → backward → step) before any improvements.
- Get the loss curve looking right on a tiny subset first.
- One change at a time.
- Don't trust any number until you've sanity-checked the inputs to it.

---

## 10:00–13:00 — Build A: MLP from scratch

Create `code/day06/mlp.py`. **No shortcuts. No torchvision Datasets.** Write a custom one.

```python
"""Day 6 — MLP on MNIST, hand-written training loop, custom Dataset."""
from __future__ import annotations
import logging
from pathlib import Path

import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import Dataset, DataLoader
from torchvision.datasets import MNIST
from torchvision.transforms import ToTensor

ART = Path("artifacts/day06"); ART.mkdir(parents=True, exist_ok=True)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
log = logging.getLogger("day06")
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"
log.info(f"Device: {DEVICE}")


class MNISTDataset(Dataset):
    """Wrap torchvision MNIST so we own the indexing logic — practice for custom datasets."""

    def __init__(self, root: str, train: bool):
        self.base = MNIST(root, train=train, download=True, transform=ToTensor())

    def __len__(self) -> int:
        return len(self.base)

    def __getitem__(self, idx: int) -> tuple[torch.Tensor, int]:
        img, label = self.base[idx]            # img is (1, 28, 28) float in [0,1]
        return img.view(-1), label             # flatten to (784,)


class MLP(nn.Module):
    def __init__(self, in_dim: int = 784, hidden: int = 256, out: int = 10, dropout: float = 0.2):
        super().__init__()
        self.fc1 = nn.Linear(in_dim, hidden)
        self.fc2 = nn.Linear(hidden, hidden)
        self.fc3 = nn.Linear(hidden, out)
        self.drop = nn.Dropout(dropout)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = F.relu(self.fc1(x))
        x = self.drop(x)
        x = F.relu(self.fc2(x))
        x = self.drop(x)
        return self.fc3(x)                     # raw logits, no softmax


@torch.no_grad()
def evaluate(model: nn.Module, loader: DataLoader) -> tuple[float, float]:
    model.eval()
    total, correct, loss_sum = 0, 0, 0.0
    for x, y in loader:
        x, y = x.to(DEVICE), y.to(DEVICE)
        logits = model(x)
        loss_sum += F.cross_entropy(logits, y, reduction="sum").item()
        pred = logits.argmax(dim=1)
        correct += (pred == y).sum().item()
        total += y.size(0)
    return loss_sum / total, correct / total


def train_one_epoch(model, loader, optimizer):
    model.train()
    loss_sum, total = 0.0, 0
    for x, y in loader:
        x, y = x.to(DEVICE), y.to(DEVICE)
        optimizer.zero_grad()
        logits = model(x)
        loss = F.cross_entropy(logits, y)
        loss.backward()
        optimizer.step()
        loss_sum += loss.item() * y.size(0)
        total += y.size(0)
    return loss_sum / total


def main():
    torch.manual_seed(42)
    train_ds = MNISTDataset("data/raw/day06", train=True)
    test_ds = MNISTDataset("data/raw/day06", train=False)
    train_loader = DataLoader(train_ds, batch_size=128, shuffle=True, num_workers=2)
    test_loader = DataLoader(test_ds, batch_size=512, shuffle=False, num_workers=2)
    log.info(f"Train batches: {len(train_loader)}  Test batches: {len(test_loader)}")

    model = MLP().to(DEVICE)
    optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)

    best_acc = 0.0
    for epoch in range(1, 11):
        tr_loss = train_one_epoch(model, train_loader, optimizer)
        va_loss, va_acc = evaluate(model, test_loader)
        log.info(f"Epoch {epoch:02d}: train_loss={tr_loss:.4f}  test_loss={va_loss:.4f}  test_acc={va_acc:.4f}")
        if va_acc > best_acc:
            best_acc = va_acc
            torch.save(model.state_dict(), ART / "mlp.pt")
    log.info(f"Best MLP test accuracy: {best_acc:.4f}")


if __name__ == "__main__":
    main()
```

Run: `uv run python code/day06/mlp.py`

Should hit ~97.5–98% test accuracy in 10 epochs. CPU ≈ 3 min. GPU ≈ 30 s.

---

## 13:00–14:00 — Lunch + Day-5 Quiz

---

## 14:00–17:00 — Build B: CNN + tighter training loop

Create `code/day06/cnn.py`. Same skeleton, but:

1. Replace MLP with a small CNN.
2. Don't flatten in the Dataset — let the model take `(1, 28, 28)` tensors. Make a `MNISTImageDataset` that doesn't reshape.
3. Add a learning-rate scheduler.
4. Add checkpointing of `(model, optimizer, epoch, best_acc)` for proper resume.

Architecture spec:

```python
class CNN(nn.Module):
    def __init__(self, num_classes: int = 10, dropout: float = 0.3):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 32, kernel_size=3, padding=1), nn.BatchNorm2d(32), nn.ReLU(),
            nn.Conv2d(32, 32, kernel_size=3, padding=1), nn.BatchNorm2d(32), nn.ReLU(),
            nn.MaxPool2d(2),                              # 28 → 14
            nn.Conv2d(32, 64, kernel_size=3, padding=1), nn.BatchNorm2d(64), nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, padding=1), nn.BatchNorm2d(64), nn.ReLU(),
            nn.MaxPool2d(2),                              # 14 → 7
        )
        self.head = nn.Sequential(
            nn.Flatten(),
            nn.Linear(64 * 7 * 7, 128), nn.ReLU(), nn.Dropout(dropout),
            nn.Linear(128, num_classes),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.head(self.features(x))
```

In `main()`:

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=2e-3, weight_decay=1e-4)
scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=10)

for epoch in range(1, 11):
    tr_loss = train_one_epoch(model, train_loader, optimizer)
    va_loss, va_acc = evaluate(model, test_loader)
    scheduler.step()
    # checkpoint logic ...
```

Checkpoint format:

```python
ckpt = {"epoch": epoch, "model": model.state_dict(),
        "optimizer": optimizer.state_dict(),
        "scheduler": scheduler.state_dict(),
        "best_acc": best_acc}
torch.save(ckpt, ART / "cnn_best.pt")
```

**Acceptance criteria:**
- Test accuracy ≥ 98%. (Trivially achievable; if not, you have a bug — likely forgot `.train()/.eval()` mode.)
- Loss curve printed each epoch and visually decreasing.
- Checkpoint can be re-loaded — write a 10-line `load_and_eval.py` proving it.

### The 10-line `load_and_eval.py`

```python
"""Smoke test: load checkpoint and reproduce test accuracy."""
import torch
from cnn import CNN, MNISTImageDataset, evaluate
from torch.utils.data import DataLoader

DEVICE = "cuda" if torch.cuda.is_available() else "cpu"
model = CNN().to(DEVICE)
ckpt = torch.load("artifacts/day06/cnn_best.pt", map_location=DEVICE)
model.load_state_dict(ckpt["model"])
loader = DataLoader(MNISTImageDataset("data/raw/day06", train=False), batch_size=512)
print(f"Loaded epoch {ckpt['epoch']}  best_acc={ckpt['best_acc']:.4f}")
print(f"Reproduced test accuracy: {evaluate(model, loader)[1]:.4f}")
```

---

## 17:00–18:00 — Code Review

```bash
git add code/day06/ artifacts/day06/
git commit -m "day-6: PyTorch MLP + CNN on MNIST"
```

Prompt:

```
Review code/day06/. You are a senior DL engineer.

1. Training loop — is it idiomatic? Compare to what Lightning would generate.
2. Did I correctly toggle model.train() / model.eval()? What breaks if I don't?
   (Hint: BatchNorm, Dropout.)
3. Loss reduction — I averaged per-batch, then divided by total. Is this correct
   even when the last batch is smaller? Show me the math.
4. AdamW vs Adam — when does the decoupled weight decay actually matter?
5. CosineAnnealingLR — is T_max=epochs correct, or should it count steps?
6. Checkpoint completeness — am I saving everything needed to resume? What's missing?
7. The Dataset/__getitem__ flattens in MLP but not CNN — would you put that logic in
   the model instead? Why or why not?

End: give me 3 numerical sanity checks I should always run before claiming
"my model trained correctly."
```

Save to `reviews/day06-review.md`.

---

## 18:00–19:00 — Dinner

---

## 19:00–21:00 — Hardening (pick two)

- **A: Mixed precision.** Add `torch.cuda.amp.autocast` + `GradScaler` if you have a GPU. Time the speedup.
- **B: TensorBoard or W&B.** Wire your CNN loop to log per-epoch metrics and a few sample predictions. Compare with W&B from Day 5 — which feels better for DL?
- **C: Confusion matrix.** Plot the 10×10 confusion matrix on test set. Which digits get confused most? (Usually 4↔9, 3↔5, 7↔1.)
- **D: Read** the Adam paper Section 3 (Algorithm 1) — https://arxiv.org/pdf/1412.6980 — and write 5 bullets on why the bias correction terms (`1 - β_1^t`) exist.
- **E: Sanity check by overfitting.** Use only 64 training examples. Your model should hit 100% training accuracy and ~poor test. This is Karpathy's "verify the recipe works" check.

---

## 21:00–21:30 — Journal

---

## Common Pitfalls

- **Pitfall 1:** Forgetting `optimizer.zero_grad()` → gradients accumulate from previous batch, your loss explodes.
- **Pitfall 2:** Forgetting `model.eval()` at validation → BatchNorm uses batch stats instead of running mean/var → val accuracy is artificially low (sometimes high too — anyway, wrong).
- **Pitfall 3:** Computing accuracy with `(pred == y).mean()` but `pred` and `y` are different dtypes → silent broadcasting error giving wrong answer.
- **Pitfall 4:** Calling `loss.item()` on every batch in a hot loop forces GPU↔CPU sync → cripples throughput. Accumulate `loss.detach().sum()` and `.item()` once per epoch.
- **Pitfall 5:** `torch.save(model)` (the whole module) creates pickle-fragile checkpoints. Always save `.state_dict()` — your saved file works after refactors.

---

## Quiz (tomorrow 13:00)

`code/day06/quiz_answers.md`.

1. What is `autograd` actually tracking when you do `y = x ** 2`?
2. Why does `model.eval()` change behavior for BatchNorm and Dropout but not Conv2d?
3. What's the difference between `F.cross_entropy(logits, y)` and a softmax + `F.nll_loss(log(p), y)`? Why do we use the former?
4. `optimizer.zero_grad(set_to_none=True)` vs default — what's the difference, and why is `True` slightly better?
5. You see `loss = nan` after 3 epochs. List four debugging steps in order.
6. What's the difference between `nn.BatchNorm2d` and `nn.LayerNorm`? When to use each?
7. Why do CNNs work better than MLPs on images, in two sentences? (Translation invariance, parameter sharing.)
8. What does `torch.no_grad()` do that `model.eval()` does not?

---

→ Tomorrow at 09:00: open `DAILY/day-07.md`.
