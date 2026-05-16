# ECE F243 — 12 Anchor Formulas

## 1. FT of u(t)

$$\mathcal{F}\{u(t)\} = \pi\delta(\omega) + \frac{1}{j\omega}$$

---

## 2. FT of two-sided exponential

$$\mathcal{F}\{e^{-a|t|}\} = \frac{2a}{a^2 + \omega^2}$$

---

## 3. FT of one-sided exponential

$$\mathcal{F}\{e^{-at}u(t)\} = \frac{1}{a + j\omega}, \quad a > 0$$

---

## 4. Laplace — causal exponential

$$\mathcal{L}\{e^{-at}u(t)\} = \frac{1}{s+a}, \quad \text{ROC: } \mathrm{Re}(s) > -a$$

---

## 5. Laplace — anticausal exponential

$$\mathcal{L}\{-e^{-at}u(-t)\} = \frac{1}{s+a}, \quad \text{ROC: } \mathrm{Re}(s) < -a$$

---

## 6. Z-transform — causal

$$\mathcal{Z}\{a^n\, u[n]\} = \frac{z}{z-a}, \quad \text{ROC: } |z| > |a|$$

---

## 7. Z-transform — anticausal

$$\mathcal{Z}\{-a^n\, u[-n-1]\} = \frac{z}{z-a}, \quad \text{ROC: } |z| < |a|$$

---

## 8. Stability — Laplace

$$\text{Stable} \iff \text{ROC includes the } j\omega\text{-axis}$$

---

## 9. Stability — Z-transform

$$\text{Stable} \iff \text{ROC includes the unit circle } |z| = 1$$

---

## 10. Fourier Series through LTI system

$$y_k = H(jk\omega_0)\cdot a_k, \qquad \omega_0 = \frac{2\pi}{T}$$

---

## 11. Sampling trap — cos squared

$$\cos^2(2\pi f_0 t) = \frac{1}{2} + \frac{1}{2}\cos(4\pi f_0 t) \implies f_{s,\min} = 4f_0$$

---

## 12. Two-sided split before Laplace

$$e^{-a|t|} = e^{-at}u(t) + e^{at}u(-t) \implies \text{ROC is a vertical strip}$$
