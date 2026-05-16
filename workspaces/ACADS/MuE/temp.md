Tier mapping: Tier 2 (DC biasing — voltage-divider) plus a load-line sketch. CS amplifier with no source degeneration.

Given: W/L = 25, R₁ ∥ R₂ = 100 kΩ, V_TN = 1 V, I_DQ = 2 mA, R_D = 2.5 kΩ. Defaults: μ_n C_ox = 140 μA/V². V_DD is read from Fig 2a — I'll take V_DD = 10 V (the only value that makes I_DQ R_D = 5 V fit; the default 3.3 V would violate KVL on the drain branch). If your figure shows a different V_DD, the method is identical — only the numerics shift.

Step 0 — Anchor V_GSQ from the drain-current equation
V_GSQ depends only on I_DQ and the device, so do this first; it's independent of V_DD.

$$I_D = \tfrac{1}{2}\mu_n C_{ox}\tfrac{W}{L}(V_{GS}-V_{TN})^2$$
$$V_{ov}^2 = \frac{2 I_D}{\mu_n C_{ox}(W/L)} = \frac{2(2\text{m})}{(140\mu)(25)} = \frac{4\text{m}}{3.5\text{m}} = 1.143$$
$$\boxed{V_{ov} \approx 1.07,V \quad\Rightarrow\quad V_{GSQ} = V_{TN}+V_{ov} \approx 2.07,V}$$

(a) Load line
KVL drain branch: V_DS = V_DD − I_D R_D ⇒ straight line, slope −1/R_D.

Axis intercept	Value
y-intercept ( I_D max ), set V_DS = 0	I_D,max = V_DD/R_D = 10/2.5k = 4 mA
x-intercept (V_DS max), set I_D = 0	V_DS,max = V_DD = 10 V
Mark the Q-point at (V_DSQ, I_DQ) = (5 V, 2 mA). The boundary V_DS = V_ov ≈ 1.07 V separates triode (left) from saturation (right) — Q sits well inside saturation.


I_D(mA)
4 |\
  | \
  |  \
2 |---*Q          <-- (5V, 2mA)
  |    \
  |     \
0 |______\______ V_DS(V)
  0  1.07  5   10
       ^     ^
       Vov   VDSQ
(b) Q-point in the middle of saturation
Load line gives V_DSQ = V_DD − I_DQ R_D = 10 − (2 m)(2.5 k) = 5 V.

Strict "middle of saturation" check: midpoint between V_DSsat = V_ov ≈ 1.07 V and V_DD = 10 V is (1.07 + 10)/2 ≈ 5.5 V. V_DSQ = 5 V is essentially the middle (within V_ov/2).

Quantity	Value
V_GSQ	2.07 V
V_DSQ	5 V
(If your instructor reads "middle of saturation" strictly — equal swing room — then V_DSQ = (V_DD+V_ov)/2 forces V_DD = 2I_DR_D + V_ov ≈ 11.07 V; the rest scales the same way.)

(c) R₁ and R₂
Source is grounded (no R_S in Fig 2a), so V_G = V_GSQ = 2.07 V.

Voltage-divider equations:
$$V_G = V_{DD}\cdot\frac{R_2}{R_1+R_2} \qquad R_1!\parallel! R_2 = 100\text{ k}\Omega$$

Let α = V_G/V_DD = 2.07/10 = 0.207. Then R_2 = α(R_1+R_2) and R_1 = (1−α)(R_1+R_2), so:

$$R_1!\parallel! R_2 = \alpha(1-\alpha)(R_1+R_2) = 100\text{ k}$$
$$R_1+R_2 = \frac{100\text{ k}}{0.207 \times 0.793} = \frac{100\text{ k}}{0.1641} \approx 609\text{ k}\Omega$$

Resistor	Value
R₂ = α(R₁+R₂)	0.207 × 609 k ≈ 126 kΩ
R₁ = (1−α)(R₁+R₂)	0.793 × 609 k ≈ 483 kΩ
Sanity: R₁ ∥ R₂ = (483)(126)/(609) ≈ 100 kΩ ✓; V_G = 10 × 126/609 ≈ 2.07 V ✓.

Why this is "guaranteed marks"
The trick is recognizing V_GSQ is decoupled from the bias network — the I_D equation alone gives it. After that, parts (a)–(c) are bookkeeping: load-line intercepts, then α-split + parallel-resistance simultaneous equations. Total time target: 8–10 minutes.

See STUDY_GUIDE.md Tier 2 (voltage-divider bias) and Tier 1 (saturation I_D inversion).