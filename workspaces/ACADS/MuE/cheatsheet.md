# ECE F244 — Concept Revision Cheat Sheet

Compre: 2026-05-11. Organized by the 7 tiers in [STUDY_GUIDE.md](STUDY_GUIDE.md).

---

## 0. Default Device Parameters

$$V_{DD} = V_{CC} = 3.3\ \text{V}$$

**NMOS:** $\mu_n C_{ox} = 140\ \mu\text{A/V}^2,\ V_{TN} = 0.7\ \text{V},\ \lambda = 0.1\ \text{V}^{-1},\ V_{ov} \approx 0.2\ \text{V}$
**PMOS:** $\mu_p C_{ox} = 40\ \mu\text{A/V}^2,\ V_{TP} = -0.7\ \text{V},\ \lambda = 0.1\ \text{V}^{-1},\ |V_{ov}| \approx 0.2\ \text{V}$
**BJT:** $\beta = 100,\ V_A = 100\ \text{V},\ V_{BE,on} = 0.6\ \text{V},\ V_T = kT/q = 25\ \text{mV}$

Bulk: NMOS $\to$ GND, PMOS $\to V_{DD}$ unless body effect is invoked.

---

## TIER 1 — MOSFET Device

### Region of operation (NMOS; flip signs for PMOS)

| Region | Condition |
|---|---|
| Cutoff | $V_{GS} < V_T$ |
| Triode (linear) | $V_{GS} > V_T$ and $V_{DS} < V_{GS}-V_T$ |
| Saturation | $V_{GS} > V_T$ and $V_{DS} \geq V_{GS}-V_T$ |
| Edge of saturation | $V_{DS} = V_{GS}-V_T = V_{ov}$ |

### Current equations

$$I_D^{\text{tri}} = \mu_n C_{ox}\frac{W}{L}\left[(V_{GS}-V_T)V_{DS} - \frac{V_{DS}^2}{2}\right]$$

$$I_D^{\text{sat}} = \frac{\mu_n C_{ox}}{2}\frac{W}{L}(V_{GS}-V_T)^2(1+\lambda V_{DS})$$

$$I_D^{\text{sub-th}} \propto \exp\!\left(\frac{V_{GS}}{n V_T}\right)$$

$V_{ov} \equiv V_{GS} - V_T$ (overdrive).

### Channel-Length Modulation (CLM)

$\lambda$ models slope of $I_D$ vs $V_{DS}$ in saturation.

$$r_o = \frac{1}{\lambda I_D} = \frac{V_A}{I_D},\qquad V_A = \frac{1}{\lambda}\ \text{(Early voltage)},\qquad V_A \propto L$$

### Body Effect

When $V_{SB} \neq 0$:

$$V_T = V_{T0} + \gamma\left(\sqrt{2\phi_F + V_{SB}} - \sqrt{2\phi_F}\right)$$

$$g_{mb} = \chi\, g_m,\qquad \chi \approx 0.1\text{–}0.3$$

### Small-signal parameters (saturation)

$$g_m = \frac{\partial I_D}{\partial V_{GS}} = \mu_n C_{ox}\frac{W}{L} V_{ov} = \frac{2 I_D}{V_{ov}} = \sqrt{2\mu_n C_{ox}\frac{W}{L} I_D}$$

$$r_o = \frac{1}{\lambda I_D} = \frac{V_A}{I_D},\qquad g_{mb} = \chi g_m$$

### Intrinsic gain

$$A_o = g_m r_o = \frac{2 V_A}{V_{ov}} = \frac{2}{\lambda V_{ov}}$$

Bigger $L$ (bigger $V_A$) and smaller $V_{ov}$ $\Rightarrow$ bigger $A_o$.

### Sizing $(W/L)$ from spec

$$\frac{W}{L} = \frac{2 I_D}{\mu C_{ox} V_{ov}^2},\qquad \frac{W}{L} = \frac{g_m^2}{2\mu C_{ox} I_D}$$

---

## TIER 2 — DC Biasing

Goal: pin down $(I_{DQ}, V_{GSQ}, V_{DSQ})$. Always check saturation.

### Self-bias (diode-connected MOSFET)

$V_G = V_D \Rightarrow$ always saturation if $I_D > 0$.

### Resistive divider + source resistor (most common in PYQs)

$$V_G = V_{DD}\,\frac{R_2}{R_1+R_2}$$

$$V_G = V_{GS} + I_D R_S,\qquad I_D = \frac{\mu_n C_{ox}}{2}\frac{W}{L}(V_{GS}-V_T)^2$$

Solve quadratic in $V_{GS}$, pick root with $V_{GS} > V_T$, then:

$$V_{DS} = V_{DD} - I_D(R_D + R_S)$$

### Load-line construction

DC load line: $I_D = \dfrac{V_{DD}-V_{DS}}{R_D+R_S}$. Q-point = intersection with device curve.
AC load line: passes through Q with slope $-1/(R_D \| R_L)$.

### Bias stability

Larger $R_S$, smaller $R_1\|R_2$ $\Rightarrow$ less sensitivity to $V_T$ spread.

---

## TIER 3 — Single-Stage Amplifiers

Workflow: (1) DC bias $\to$ $g_m, r_o$; (2) replace MOS with small-signal model; (3) apply formula.

### Common-Source (CS), resistive load

$$A_v = -g_m(R_D \| r_o \| R_L),\qquad R_{in} = \infty,\qquad R_{out} = R_D \| r_o$$

Inverting. Highest gain of basic stages.

### CS with source degeneration

$$A_v \approx -\frac{g_m R_D}{1 + g_m R_S}$$

$$R_{out,\text{drain}} = R_D \,\|\, \big[\,r_o + R_S + g_m r_o R_S\,\big]$$

Trades gain for linearity.

### Common-Drain (Source Follower)

$$A_v = \frac{g_m r_o}{1 + g_m r_o} \approx 1$$

$$R_{in} = \infty,\qquad R_{out} = \frac{1}{g_m}\,\Big\|\,r_o \approx \frac{1}{g_m}$$

### Common-Gate

$$A_v = g_m(R_D \| r_o),\qquad R_{in} = \frac{1}{g_m}$$

$$R_{out} = R_D \,\|\, \big[\,r_o(1 + g_m R_{src})\,\big]$$

Low input R, high BW (no Miller).

### Cascode (CS + CG stacked)

$$R_{out} = r_{o2}(1 + g_{m2} r_{o1}) + r_{o1} \approx g_{m2} r_{o1} r_{o2}$$

$$A_v = -g_{m1} R_{out}$$

With matched complementary cascode active load:

$$R_{out,\text{tot}} = (g_m r_o^2)_n \,\|\, (g_m r_o^2)_p$$

### Active-load CS

$$A_v = -g_m(r_{on} \| r_{op}) \approx -\frac{A_o}{2}\ \text{(matched)}$$

---

## TIER 4 — Current Mirrors

Function: copy reference current $I_{ref}$ into output legs.

### Basic NMOS mirror

$$\frac{I_O}{I_{REF}} = \frac{(W/L)_2}{(W/L)_1}\quad\text{(ignoring }\lambda\text{)}$$

$$\frac{I_O}{I_{REF}} = \frac{(W/L)_2}{(W/L)_1}\cdot\frac{1+\lambda V_{DS,2}}{1+\lambda V_{DS,1}}$$

$$R_{out} = r_{o2},\qquad V_{\min} = V_{ov,2}$$

### Cascode mirror

$$R_{out} \approx g_m r_o^{\,2},\qquad V_{\min} = 2V_{ov} + V_T$$

### Wide-swing / low-voltage cascode

$$R_{out} \approx g_m r_o^{\,2},\qquad V_{\min} = 2V_{ov}$$

### Wilson mirror

$$R_{out} \approx \frac{g_m r_o^{\,2}}{2}$$

### BJT mirror

$$\frac{I_O}{I_{REF}} = \frac{1}{1 + 2/\beta} \approx 1 - \frac{2}{\beta},\qquad R_{out} = r_o$$

---

## TIER 5 — Differential Amplifier

Matched $M_1, M_2$ with sources tied to tail $I_{SS}$ ($R_{ss} = r_{o,\text{tail}}$). Loads = $R_D$ or current-mirror.

### DC

$$I_{D1} = I_{D2} = \frac{I_{SS}}{2}$$

$$g_m = \sqrt{\mu_n C_{ox}\frac{W}{L}\,I_{SS}},\qquad V_{ov} = \sqrt{\frac{I_{SS}}{\mu_n C_{ox}(W/L)}}$$

### Differential-mode gain (single-ended out, Sedra)

$$A_{dm,\text{res}} = -g_m R_D$$

$$A_{dm,\text{active}} = -g_m(r_{on} \| r_{op})$$

Fully differential out: same magnitude (factor of 2 differs by convention; check the question's definition).

### Common-mode gain & CMRR

$$A_{cm} \approx -\frac{R_D}{2 R_{ss}}$$

$$\text{CMRR} = \left|\frac{A_{dm}}{A_{cm}}\right| \approx 2 g_m R_{ss}$$

$$\text{CMRR}_{\text{dB}} = 20\log_{10}\left|\frac{A_{dm}}{A_{cm}}\right|$$

### Input common-mode range (NMOS pair)

$$\text{ICMR}_{\max} = V_{DD} - |V_{ov,\text{load}}| \quad\text{(mirror load)}$$

$$\text{ICMR}_{\max} = V_{DD} - |V_{ov,\text{load}}| - |V_{TP}| + V_{TN} \quad\text{(resistive)}$$

$$\text{ICMR}_{\min} = V_{SS} + V_{ov,\text{tail}} + V_{TN} + V_{ov,1}$$

### Output swing per side

$$V_{\max} = V_{DD} - |V_{ov,\text{load}}|,\qquad V_{\min} = V_{CM,in} - V_{TN}$$

### Offset voltage

$$V_{os} = \Delta V_T + \frac{V_{ov}}{2}\!\left(\frac{\Delta R_D}{R_D} - \frac{\Delta(W/L)}{W/L}\right)$$

### Slew rate

$$\text{SR} = \frac{I_{SS}}{C_L}$$

---

## TIER 6 — Frequency Response

### High-frequency MOS model

Add three caps: $C_{gs}$ (gate-source, $\approx \tfrac{2}{3} W L C_{ox}$ in sat), $C_{gd}$ (Miller path), $C_{db}$.

### Transit frequency

$$f_T = \frac{g_m}{2\pi(C_{gs} + C_{gd})}$$

### Miller's theorem

For $Z$ bridging input-output of inverting amp with gain $A_v$ ($A_v < 0$):

$$Z_{in,\text{Mil}} = \frac{Z}{1 - A_v},\qquad Z_{out,\text{Mil}} = \frac{Z}{1 - 1/A_v}$$

$$C_{in,\text{Mil}} = C_{gd}(1 + |A_v|),\qquad C_{out,\text{Mil}} = C_{gd}\!\left(1 + \frac{1}{|A_v|}\right) \approx C_{gd}$$

### Open-Circuit Time Constants

For each $C_k$ find $R_k$ (other caps open, sources zeroed):

$$\tau_H = \sum_k C_k R_k,\qquad \omega_H \approx \frac{1}{\tau_H}$$

### Short-Circuit Time Constants (low end)

$$\omega_L \approx \sum_k \frac{1}{C_k R_k}\quad\text{(other caps shorted)}$$

### Bode rules

Pole: $-20$ dB/dec, $-45^\circ$ at corner. Zero: $+20$ dB/dec.
**RHP zero** subtracts phase (looks like a pole in phase) but adds magnitude like a zero.

### Dominant-pole approximation

$$|H(j\omega)| \approx \frac{|A_o|}{\sqrt{1 + (\omega/\omega_{p1})^2}}$$

$$\text{UGB} = |A_o|\,\omega_{p1},\qquad \text{GBW = const} \Rightarrow \text{trade gain for BW}$$

### Phase margin

$$\text{PM} = 180^\circ + \angle H(j\omega_u),\quad |H(j\omega_u)| = 1$$

PM $\geq 60^\circ$ safe; $45^\circ$ marginal; $<45^\circ$ ringy/unstable.

### Miller-compensated RHP zero (two-stage)

$$\omega_z = \frac{g_{m2}}{C_c}\quad(\text{RHP, hurts phase})$$

With nulling resistor $R_z$:

$$\omega_z = \frac{g_{m2}}{C_c(1 - g_{m2} R_z)},\qquad R_z = \frac{1}{g_{m2}}\Rightarrow\text{zero cancelled}$$

### Dominant pole by topology

| Stage | Dominant cap | Why |
|---|---|---|
| CS | $C_{gd}$ via Miller at input node | $C_{in,\text{Mil}}$ large |
| CD | $C_{gs}$ at input | No Miller, low gain $\Rightarrow$ wide BW |
| CG | Output node | No Miller |
| Cascode | Output node | Input Miller killed by low $Z$ at CG source |

---

## TIER 7 — Feedback Amplifiers

### Topology identification (2×2 rule)

| Sample (out) | Mix (in) | Topology | Amplifies | $R_{in,f}$ | $R_{out,f}$ |
|---|---|---|---|---|---|
| Voltage | Series | Series-Shunt | $V\to V$ | $R_{in}(1+T)$ | $R_{out}/(1+T)$ |
| Voltage | Shunt | Shunt-Shunt | $I\to V$ | $R_{in}/(1+T)$ | $R_{out}/(1+T)$ |
| Current | Series | Series-Series | $V\to I$ | $R_{in}(1+T)$ | $R_{out}(1+T)$ |
| Current | Shunt | Shunt-Series | $I\to I$ | $R_{in}/(1+T)$ | $R_{out}(1+T)$ |

**Sample test:** short the output load — if feedback dies, voltage-sampled. Open it — if dies, current-sampled.
**Mix test:** voltages subtracted in a loop $\Rightarrow$ series-in; currents summed at a node $\Rightarrow$ shunt-in.

### Closed-loop quantities

$$T = A\beta,\qquad A_f = \frac{A}{1+T}\xrightarrow{T\gg 1}\frac{1}{\beta}$$

### Resistance scaling

- Series mixing $\Rightarrow R_{in,f} = R_{in}(1+T)$
- Shunt mixing $\Rightarrow R_{in,f} = R_{in}/(1+T)$
- Shunt sampling (V-out) $\Rightarrow R_{out,f} = R_{out}/(1+T)$
- Series sampling (I-out) $\Rightarrow R_{out,f} = R_{out}(1+T)$

### Bandwidth & gain-BW

$$\omega_{Hf} = \omega_H(1+T),\qquad \omega_{Lf} = \frac{\omega_L}{1+T}$$

$$A_f \cdot \omega_{Hf} = A \cdot \omega_H\quad\text{(GBW preserved)}$$

### $\beta$ network units

| Topology | $\beta$ definition | Units |
|---|---|---|
| Series-Shunt | $V_f / V_o$ | dimensionless |
| Shunt-Shunt | $I_f / V_o$ | siemens (S) |
| Series-Series | $V_f / I_o$ | ohms ($\Omega$) |
| Shunt-Series | $I_f / I_o$ | dimensionless |

---

## BJT Quick Card

$$I_C = I_S \exp(V_{BE}/V_T)(1 + V_{CE}/V_A)$$

$$g_m = \frac{I_C}{V_T},\qquad r_\pi = \frac{\beta}{g_m} = \frac{\beta V_T}{I_C},\qquad r_o = \frac{V_A}{I_C}$$

$$A_{intr} = g_m r_o = \frac{V_A}{V_T}\approx 4000$$

**CE:** $A_v = -g_m(R_C\|r_o\|R_L),\ R_{in}=r_\pi,\ R_{out}=R_C\|r_o$
**CC (follower):** $A_v\approx 1,\ R_{in}=r_\pi+(\beta+1)R_E,\ R_{out}\approx 1/g_m$
**CB:** $A_v=g_m(R_C\|r_o),\ R_{in}=r_e=1/g_m=V_T/I_E$

All MOS topology rules carry over with $g_m = I_C/V_T$, no body effect.

---

## Two-Stage CMOS OpAmp

$$A_{dm,\text{tot}} = g_{m1}(r_{o1}\|r_{o3})\cdot g_{m6}(r_{o6}\|r_{o7})$$

$$\text{ICMR}_{\max} = V_{DD} - |V_{ov,p}| - |V_{TP}| + V_{TN}$$

$$\text{ICMR}_{\min} = V_{SS} + V_{ov,\text{tail}} + V_{TN} + V_{ov,1}$$

$$\text{SR} = \frac{I_{SS}}{C_c},\qquad \text{GBW} = \frac{g_{m1}}{C_c}$$

Output swing: $V_{SS}+V_{ov,7}\ \to\ V_{DD}-V_{ov,6}$. RHP zero at $g_{m6}/C_c$; null with $R_z = 1/g_{m6}$. PM $\geq 60^\circ \Rightarrow$ place 2nd pole at $\geq 2.2\,\text{GBW}$.

---

## VTC Sketch (CMOS inverter / CS amp)

5 regions: cutoff $\to$ NMOS-sat/PMOS-tri $\to$ both-sat (steep) $\to$ NMOS-tri/PMOS-sat $\to$ PMOS-cutoff. Switching threshold:

$$V_M = \frac{V_{DD} - |V_{TP}| + V_{TN}\sqrt{k_n/k_p}}{1 + \sqrt{k_n/k_p}},\qquad k_x = \mu_x C_{ox}\frac{W}{L}\bigg|_x$$

Symmetric inverter ($k_n = k_p$): $V_M = V_{DD}/2$, gain at $V_M$: $A_v = -(g_{mn}+g_{mp})/(g_{on}+g_{op})$.

---

## Checklist Per Question Type

**Part A Q1 (DC + small-signal):** region check $\to$ solve quadratic for $V_{GSQ}$ $\to$ $I_{DQ}$ $\to$ $g_m,r_o$ $\to$ small-signal model $\to$ apply CS/CD/CG/Cascode $\to$ report $R_{in}, R_{out}, A_v, A_i$ with units.

**Part A Q2 (mirror sizing or VTC):** $W/L$ ratio, headroom check ($V_{\min} = V_{ov}$ or $2V_{ov}$).

**Part B Q1 (Diff amp):** $I_D = I_{SS}/2 \to g_m \to A_{dm} = -g_m R_{\text{load}} \to$ ICMR $\to$ swing $\to$ CMRR with $R_{ss}=r_{o,\text{tail}} \to$ offset.

**Part B Q2 (CS bias + cascode):** load line, Q-point, then cascode small-signal: $R_{out}\approx g_{m2} r_{o1} r_{o2}$, $A_v = -g_{m1} R_{out}$.

**Part B Q3 (Frequency):** HF model with $C_{gs}, C_{gd}, C_{db}$ $\to$ Miller-reflect $C_{gd}$ $\to$ OCTC sum $\to$ $\omega_{p1}$, UGB $= A_o\omega_{p1}$ $\to$ Bode $\to$ PM.

**Part B Q4 (Feedback):** sample/mix test $\to$ topology $\to$ $\beta$ $\to$ $T = A\beta$ $\to$ $A_f, R_{in,f}, R_{out,f}, \omega_{Hf}$ via $(1+T)$ scaling.

---

## Units & Sanity Checks

- $g_m$ in mA/V (mS); $r_o$ in k$\Omega$–M$\Omega$.
- CS $|A_v|\sim 10\text{–}100$; cascode $\sim 10^3$; OpAmp open-loop $\sim 10^4\text{–}10^5$.
- If CS $|A_v| < 1$, recheck — probably forgot $r_o$ or used triode.
- If $V_{ov} < 0$ or $> 1$ V, recheck region/quadratic root.
- Always verify $V_{DS} \geq V_{ov}$ for every transistor before trusting saturation formulas.
