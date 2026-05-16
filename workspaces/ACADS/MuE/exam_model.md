# ECE F244 — Exam Model (Comprehensive, 11 May 2026)

10 long-form problems matching the empirical Part A + Part B template (2013–2023 PYQs). Every problem is followed by a full step-by-step solution. Default device parameters apply unless overridden:

```
VDD = 3.3 V
NMOS: μnCox = 140 μA/V²,  VTN = 0.7 V,  λ = 0.1 V⁻¹
PMOS: μpCox = 40  μA/V²,  VTP = −0.7 V, λ = 0.1 V⁻¹
BJT:  β = 100, VA = 100 V, VBE,on = 0.6 V, VT = kT/q = 25 mV
```

Tier coverage (per `STUDY_GUIDE.md`): **1, 2, 3, 4, 5, 6, 7** — all hit.

---

## Problem 1 — Self-Biased CS Amplifier (Part A Q1, Tier 2+3)

### Problem
A common-source NMOS amplifier uses a self-bias scheme. The source is bypassed for AC. Find $I_{DQ}$, $V_{GSQ}$, $V_{DSQ}$, then $g_m$, $r_o$, $A_v$, $R_{in}$, $R_{out}$. Verify the device is in saturation.

```
              VDD = 3.3 V
                 │
            ┌────┴────┐
           [R1=2 MΩ] [RD=10 kΩ]
            │         │
            ├────┐    ├──── Vout (─Cout─► load Rout)
            │    │    │
   Cin──────┤   Vin   │
            │    │    │
           [R2=1MΩ] [M1]   W/L = 100,  default NMOS
            │         │
            │        [RS=1 kΩ]──┬── CS (bypass, large)
            │         │         │
            └─────────┴─────────┘
                  GND
```

### Solution

**Step 1 — Gate DC voltage.**

Since gate current is zero,

$$V_G = V_{DD}\cdot\frac{R_2}{R_1+R_2} = 3.3\cdot\frac{1}{3} = 1.10\;\text{V}.$$

**Step 2 — DC current equation.**

Source voltage $V_S = I_D R_S$. So $V_{GS}=V_G - I_D R_S = 1.10 - I_D(\text{mA})\cdot 1.$

Assume saturation:

$$I_D = \tfrac{1}{2}\mu_n C_{ox}\,(W/L)(V_{GS}-V_T)^2 = \tfrac{1}{2}(140\mu)(100)(V_{GS}-0.7)^2 = 7\,\text{mA}\cdot(V_{GS}-0.7)^2.$$

**Step 3 — Solve quadratic.**

Let $x = I_D$ in mA. $V_{GS} = 1.10 - x$, so $V_{GS}-0.7 = 0.4-x$.

$$x = 7(0.4-x)^2 \;\Rightarrow\; 7x^2 - 6.6x + 1.12 = 0.$$

$$x = \frac{6.6\pm\sqrt{43.56-31.36}}{14} = \frac{6.6\pm 3.49}{14} \;\Rightarrow\; x = 0.222\text{ or }0.722.$$

Discard $x = 0.722$ (gives $V_{GS} = 0.378 < V_T$ — device off).

$$\boxed{I_{DQ} = 0.222\;\text{mA},\quad V_{GSQ} = 0.878\;\text{V},\quad V_{ov} = 0.178\;\text{V}}$$

**Step 4 — DC drain voltage and saturation check.**

$$V_{DS} = V_{DD} - I_D(R_D + R_S) = 3.3 - 0.222(11) = 0.860\;\text{V}.$$

Saturation requires $V_{DS} \ge V_{ov}$: $0.86 > 0.178$ ✓.

**Step 5 — Small-signal parameters.**

$$g_m = \frac{2 I_D}{V_{ov}} = \frac{2(0.222\text{m})}{0.178} = 2.49\;\text{mA/V}$$

$$r_o = \frac{1}{\lambda I_D} = \frac{1}{0.1 \cdot 0.222\text{m}} = 45.0\;\text{k}\Omega$$

**Step 6 — AC gain, input/output resistance (source bypassed).**

$$A_v = -g_m(R_D \,\|\, r_o) = -2.49\text{m}\cdot(10\text{k}\,\|\,45\text{k}) = -2.49\text{m}\cdot 8.18\text{k} = \boxed{-20.4\;\text{V/V}}$$

$$R_{in} = R_1 \,\|\, R_2 = 667\;\text{k}\Omega,\qquad R_{out} = R_D \,\|\, r_o = 8.18\;\text{k}\Omega.$$

---

## Problem 2 — Region of Operation + Body Effect (Part A Q2, Tier 1)

### Problem
An NMOS device has $V_{GS} = 1.0\,\text{V}$, $V_{DS} = 0.3\,\text{V}$, $V_{SB} = 1.0\,\text{V}$. Parameters: $V_{T0} = 0.6\,\text{V}$, $\gamma = 0.4\,\text{V}^{1/2}$, $2\phi_F = 0.7\,\text{V}$, $\mu_n C_{ox}(W/L) = 1\,\text{mA/V}^2$, $\lambda = 0.05\,\text{V}^{-1}$. (a) Determine the region of operation. (b) Compute $I_D$. (c) Compute $g_m$ and $g_{mb}$.

### Solution

**Step 1 — Effective threshold with body effect.**

$$V_T = V_{T0} + \gamma\!\left[\sqrt{2\phi_F + V_{SB}} - \sqrt{2\phi_F}\right]$$

$$V_T = 0.6 + 0.4\!\left[\sqrt{1.7}-\sqrt{0.7}\right] = 0.6 + 0.4(1.304 - 0.837) = 0.6 + 0.187 = 0.787\;\text{V}.$$

**Step 2 — Region check.**

$V_{ov} = V_{GS}-V_T = 1.0 - 0.787 = 0.213\,\text{V}$. Since $V_{DS} = 0.3 > V_{ov} = 0.213$, the device is in **saturation**.

**Step 3 — Drain current with channel-length modulation.**

$$I_D = \tfrac12\mu_nC_{ox}(W/L)V_{ov}^2(1+\lambda V_{DS}) = \tfrac12(1\text{m})(0.213)^2(1+0.05\cdot 0.3)$$

$$I_D = 0.5\text{m}\cdot 0.0454 \cdot 1.015 = 23.0\;\mu\text{A}.$$

**Step 4 — Small-signal $g_m$ and $g_{mb}$.**

$$g_m = \mu_n C_{ox}(W/L)V_{ov} = 1\text{m}\cdot 0.213 = 0.213\;\text{mA/V}.$$

$$\eta = \frac{g_{mb}}{g_m} = \frac{\gamma}{2\sqrt{2\phi_F+V_{SB}}} = \frac{0.4}{2\sqrt{1.7}} = 0.153.$$

$$g_{mb} = 0.153\cdot 0.213\text{m} = 32.6\;\mu\text{A/V}.$$

$\boxed{I_D = 23\,\mu\text{A},\;g_m = 213\,\mu\text{A/V},\;g_{mb} = 32.6\,\mu\text{A/V}, \text{ saturation.}}$

---

## Problem 3 — Cascode Current Mirror Sizing (Part A Q2, Tier 4)

### Problem
Design a cascode NMOS current mirror to deliver $I_{out} = I_{ref} = 200\,\mu\text{A}$ with $V_{ov} = 0.2\,\text{V}$ for each transistor. (a) Pick $W/L$. (b) Find minimum output voltage. (c) Compute output resistance.

```
          VDD                     VDD
           │                       │
          [Iref=200μA]           Vout o─── load
           │                       │
   ┌───────┴───────┐               │
   │      drain    │               │
   │      ┌─[M3]──┐│               ├─[M4] (cascode)
   │      │       ││               │
   │      gate────┘│               │
   │      drain    │               │
   │      ┌─[M1]──┐│               ├─[M2] (mirror)
   │      │       ││               │
   │      gate────┘│               │
   │      source   │               │
   │      GND      │               GND
```
(M1, M3 on the left are diode-connected; M1↔M2 share gate, M3↔M4 share gate.)

### Solution

**Step 1 — $W/L$ from Vov target.**

$$I_D = \tfrac12\mu_n C_{ox}(W/L)V_{ov}^2 \;\Rightarrow\; (W/L) = \frac{2I_D}{\mu_n C_{ox}V_{ov}^2}=\frac{2(200\mu)}{(140\mu)(0.04)} = 71.4 \approx 72.$$

All four NMOS use $W/L = 72$.

**Step 2 — DC node voltages on reference branch.**

$V_{GS1} = V_{GS3} = V_T+V_{ov} = 0.9\,\text{V}$. The gate of M3 (= drain of M3) sits at $V_{GS1}+V_{GS3} = 1.8\,\text{V}$.

**Step 3 — Output voltage compliance ($V_{out,min}$).**

For matching we want $V_{DS2} = V_{DS1}$, i.e. source of M4 at $V_{GS1} = 0.9\,\text{V}$.

Then M4 gate is at $1.8\,\text{V}$, so $V_{GS4} = 1.8 - 0.9 = 0.9\,\text{V}$ ✓ (matches M3 → mirror integrity).

For M4 saturation: $V_{DS4}\ge V_{ov}=0.2\,\text{V}$, so $V_{out,\min} = V_{S4} + V_{ov4} = 0.9 + 0.2 = \boxed{1.1\;\text{V}}$.

(A wide-swing/low-voltage cascode would push this to $2V_{ov} = 0.4\,\text{V}$ by re-routing M4's gate.)

**Step 4 — Output resistance.**

$$g_m = \frac{2 I_D}{V_{ov}} = \frac{2(200\mu)}{0.2} = 2.0\;\text{mA/V}, \quad r_o = \frac{1}{\lambda I_D}=\frac{1}{0.1\cdot 200\mu} = 50\;\text{k}\Omega.$$

$$R_{out}\approx g_{m4}\,r_{o4}\,r_{o2} = (2\text{m})(50\text{k})(50\text{k}) = \boxed{5\;\text{M}\Omega}.$$

(Compared to a basic mirror with $R_{out} = r_{o2} = 50\,\text{k}\Omega$, cascoding boosts $R_{out}$ by $g_m r_o = 100$.)

---

## Problem 4 — CS with Active Load and Cascode Comparison (Part B Q2, Tier 3)

### Problem
NMOS M1 (W/L = 50) is the input device of a CS stage; PMOS M2 (W/L = 200) acts as the active load supplying $I_D = 200\,\mu\text{A}$. (a) Find $A_v$ and $R_{out}$ of this single-stage amplifier. (b) Now replace M2 with a cascoded PMOS pair (M2, M3) and add an NMOS cascode (M4) above M1. Find the new $A_v$ and $R_{out}$.

```
(a) Simple active-load CS         (b) Telescopic cascode
        VDD                              VDD
         │                                │
      [M2 PMOS]──Vbias                  [M2]──Vbp1
         │                                │
         │                              [M3]──Vbp2
         ├── Vout                         │
         │                                ├── Vout
      [M1 NMOS]                         [M4]──Vbn2
         │ │                              │
        Vin GND                         [M1]──Vin
                                           │
                                          GND
```

### Solution — Part (a)

**Step 1 — Operating-point parameters.**

$$V_{ov,1} = \sqrt{\frac{2I_D}{\mu_nC_{ox}(W/L)_1}} = \sqrt{\frac{2(200\mu)}{(140\mu)(50)}} = \sqrt{0.0571} = 0.239\;\text{V}.$$

$$g_{m1} = \frac{2 I_D}{V_{ov1}} = \frac{2(200\mu)}{0.239} = 1.67\;\text{mA/V}, \quad r_{o1} = \frac{1}{0.1\cdot 200\mu}= 50\;\text{k}\Omega.$$

For PMOS M2 ($W/L = 200$):

$$V_{ov,2} = \sqrt{\frac{2(200\mu)}{(40\mu)(200)}} = \sqrt{0.05}=0.224\;\text{V},\quad g_{m2}=1.79\;\text{mA/V},\quad r_{o2}=50\;\text{k}\Omega.$$

**Step 2 — Gain.**

$$R_{out} = r_{o1}\,\|\,r_{o2} = 25\;\text{k}\Omega.$$

$$A_v = -g_{m1}(r_{o1}\|r_{o2}) = -1.67\text{m}\cdot 25\text{k} = \boxed{-41.8\;\text{V/V}}.$$

### Solution — Part (b) Cascode

**Step 3 — Resistance looking up (PMOS cascode).**

$$R_{up} \approx g_{m2}r_{o2}r_{o3} = (1.79\text{m})(50\text{k})(50\text{k}) = 4.48\;\text{M}\Omega.$$

**Step 4 — Resistance looking down (NMOS cascode through M4 and M1).**

Use $g_{m4}, r_{o4}$ same as M1 (same dimensions, same $I_D$):

$$R_{down}\approx g_{m4}r_{o4}r_{o1} = (1.67\text{m})(50\text{k})(50\text{k}) = 4.18\;\text{M}\Omega.$$

**Step 5 — Total output resistance and gain.**

$$R_{out} = R_{up}\,\|\,R_{down} = 4.48\text{M}\,\|\,4.18\text{M}=2.16\;\text{M}\Omega.$$

$$A_v = -g_{m1}\,R_{out} = -1.67\text{m}\cdot 2.16\text{M} = \boxed{-3608\;\text{V/V}}.$$

**Take-away.** Cascoding boosts gain by a factor of $\sim g_m r_o \approx 86$.

---

## Problem 5 — NMOS Differential Amplifier with Resistive Loads (Part B Q1, Tier 5)

### Problem
A symmetric NMOS differential pair has tail current $I_{SS} = 200\,\mu\text{A}$ supplied by an ideal current source whose output resistance is $R_{SS} = 50\,\text{k}\Omega$. Each input transistor has $W/L = 50$. Loads are $R_D = 10\,\text{k}\Omega$ each. Supplies are $V_{DD} = +1.65\,\text{V}, V_{SS} = -1.65\,\text{V}$.

Compute (a) DC drain voltages and quiescent values; (b) differential and single-ended gain; (c) common-mode gain; (d) CMRR (single-ended output); (e) ICMR (max and min); (f) output swing.

```
              VDD = +1.65 V
              ┌───────────┬──────────┐
              │           │          │
            [RD]        [RD]
              │           │
              ├── vO1     ├── vO2
              │           │
       vIN1──[M1]       [M2]──vIN2
              │           │
              └─────┬─────┘
                    │
              ┌──[Tail Iss]──   Rss = 50 kΩ
              │
              VSS = −1.65 V
```

### Solution

**Step 1 — DC.** Each side carries $I_D = I_{SS}/2 = 100\,\mu\text{A}$.

$$V_{ov} = \sqrt{\frac{2(100\mu)}{(140\mu)(50)}} = 0.169\,\text{V}, \quad V_{GS} = 0.7+0.169 = 0.869\,\text{V}.$$

$$g_m = \frac{2 I_D}{V_{ov}} = \frac{2(100\mu)}{0.169}=1.18\;\text{mA/V},\quad r_o = \frac{1}{\lambda I_D} = \frac{1}{0.1\cdot 100\mu} = 100\;\text{k}\Omega.$$

DC drain voltage at each output:

$$V_O = V_{DD} - I_D R_D = 1.65 - 100\mu\cdot 10\text{k} = 0.65\;\text{V}.$$

**Step 2 — Differential gain (half-circuit).**

Set $v_{id} = v_{G1} - v_{G2}$. The source becomes a virtual ground.

$$A_{dm,\text{diff}} = \frac{v_{O1}-v_{O2}}{v_{id}} = -g_m(R_D\,\|\,r_o) = -1.18\text{m}(10\text{k}\,\|\,100\text{k}) = -1.18\text{m}\cdot 9.09\text{k} = \boxed{-10.7\;\text{V/V}}.$$

Single-ended (output taken at $v_{O1}$):

$$A_{dm,\text{SE}} = -\tfrac12 g_m(R_D\|r_o) = -5.36\;\text{V/V}.$$

**Step 3 — Common-mode gain.**

CM half-circuit: each device sees $2R_{SS}$ in its source.

$$A_{cm,\text{SE}} = \frac{-g_m(R_D\|r_o)}{1 + g_m(2R_{SS})} \approx -\frac{R_D}{2R_{SS}} = -\frac{10\text{k}}{100\text{k}} = -0.10\;\text{V/V}.$$

(Differential output ideally yields $A_{cm,\text{diff}} = 0$ under perfect matching.)

**Step 4 — CMRR (single-ended).**

$$\text{CMRR} = \left|\frac{A_{dm,\text{SE}}}{A_{cm,\text{SE}}}\right| = \frac{5.36}{0.10}=53.6 \;\Rightarrow\; 20\log_{10}(53.6) = \boxed{34.6\;\text{dB}}.$$

(Equivalently, CMRR $\approx g_m R_{SS} = 1.18\text{m}\cdot 50\text{k} = 59$ — same order.)

**Step 5 — ICMR.**

*Lower limit* — tail current source must remain saturated. Tail device $V_{ov,\text{tail}} = 0.239\,\text{V}$ (use $W/L = 50$, $I = 200\mu\text{A}$):

$$V_{IC,\min} = V_{SS} + V_{ov,\text{tail}} + V_{GS1} = -1.65 + 0.239 + 0.869 = \boxed{-0.542\;\text{V}}.$$

*Upper limit* — input pair must remain saturated, $V_{D} \ge V_G - V_T$:

$$V_{IC,\max} = V_O + V_{TN} = 0.65 + 0.7 = \boxed{1.35\;\text{V}}.$$

So ICMR $\approx [-0.54,\,+1.35]\,\text{V}$.

**Step 6 — Output swing per side.**

DC drain at $0.65\,\text{V}$. Lower limit (sat of M1 at largest input swing) $\approx V_{IC} - V_T$. With $V_{IC} = 0$: $V_{O,\min}\approx -0.7\,\text{V}$ (electrically), but practical swing constrained by $V_D \ge V_G - V_T$ during signal excursion. Upper bound $V_{DD} = 1.65\,\text{V}$ when $I_D \to 0$. Symmetric usable swing $\approx \pm 0.5\,\text{V}$ around the quiescent $0.65\,\text{V}$.

---

## Problem 6 — Diff-Pair with Current-Mirror Load + Offset (Part B Q1, Tier 5)

### Problem
The same NMOS pair as Problem 5 (W/L = 50, $I_{SS}=200\,\mu\text{A}$, $R_{SS}=50\,\text{k}\Omega$) is now loaded by a PMOS current-mirror $M_3-M_4$ (W/L = 200, default PMOS). The output is taken single-ended at the drain of M2.

(a) Compute the differential gain. (b) Compute output swing limits. (c) If $\Delta V_T = 5\,\text{mV}$ between M1–M2 and $\Delta(W/L)/(W/L) = 1\%$ between mirror M3–M4, estimate the input-referred offset.

```
              VDD
            ┌──┬──┐
           [M3][M4]   PMOS mirror, gate of M3 = gate of M4 = drain M3
            │   │
            │   ├── vOUT (single-ended)
            │   │
           [M1][M2]   input pair
            │   │
            └─┬─┘
            [Iss]  ─── Rss
              │
             VSS
```

### Solution

**Step 1 — Operating-point.** Same as Problem 5: $g_{m1,2}=1.18\text{ mA/V}$, $r_{o1,2}=100\text{ k}\Omega$.

For PMOS load (W/L = 200, $I_D = 100\,\mu\text{A}$):

$$V_{ov,p} = \sqrt{\frac{2(100\mu)}{(40\mu)(200)}}= 0.158\,\text{V},\quad g_{mp}=1.27\,\text{mA/V},\quad r_{op}= 100\,\text{k}\Omega.$$

**Step 2 — Differential gain.**

A diff-pair with mirror load delivers the *full* $g_m$ to a single-ended node:

$$A_{dm} = g_{m1}\,(r_{o2}\,\|\,r_{o4}) = 1.18\text{m}\cdot(100\text{k}\,\|\,100\text{k})= 1.18\text{m}\cdot 50\text{k} = \boxed{59\;\text{V/V}}.$$

(Sign convention: positive at $v_{IN1}$ relative to $v_{IN2}$; the mirror flips one polarity so both signals add at the output.)

**Step 3 — Output swing.**

*Upper:* $v_{OUT,\max} = V_{DD} - |V_{ov4}| = 3.3 - 0.158 = 3.14\,\text{V}$.

*Lower:* M2 must remain saturated: $V_{OUT,\min} = V_{IC} - V_{TN}$. With $V_{IC} \approx V_{DD}/2 = 1.65\,\text{V}$, $V_{OUT,\min}\approx 0.95\,\text{V}$.

**Step 4 — Input-referred offset.**

$$V_{OS} \approx \Delta V_T + \frac{V_{ov}}{2}\!\left[\frac{\Delta(W/L)_{\text{mirror}}}{(W/L)} + \frac{\Delta V_T}{V_{ov}}\right] \approx \Delta V_{T,\text{pair}} + \frac{V_{ov,p}}{2}\cdot\frac{\Delta(W/L)}{(W/L)}.$$

With $\Delta V_T = 5\,\text{mV}$ and the mirror mismatch contribution

$$\frac{V_{ov,p}}{2}\cdot 0.01 = \frac{0.158}{2}\cdot 0.01 = 0.79\,\text{mV}.$$

$$\boxed{V_{OS} \approx 5 + 0.79 \approx 5.8\;\text{mV}.}$$

(Threshold mismatch dominates — typical for short channels.)

---

## Problem 7 — CS High-Frequency Response (Part B Q3, Tier 6)

### Problem
A CS amplifier has source resistance $R_{sig} = 10\,\text{k}\Omega$ driving M1. Drain load $R_D = 10\,\text{k}\Omega$. Bias: $I_D = 100\,\mu\text{A}$, $V_{ov} = 0.2\,\text{V}$ → $g_m = 1\,\text{mA/V}, r_o = 100\,\text{k}\Omega$. Capacitances: $C_{gs} = 50\,\text{fF}$, $C_{gd} = 5\,\text{fF}$, $C_{db} = 5\,\text{fF}$, plus a load capacitance $C_L = 100\,\text{fF}$.

(a) Find midband gain. (b) Use Miller approximation to estimate the dominant pole. (c) Use OCTC to refine. (d) Estimate UGB and the location of the RHP zero.

```
   Vsig ──[R_sig]── G ──[M1]── D ── Vout
                               │
                              [RD]                CL at output
                               │
                              VDD
```

### Solution

**Step 1 — Midband gain.**

$$|A_v| = g_m(R_D\|r_o) = 1\text{m}(10\text{k}\,\|\,100\text{k}) = 1\text{m}\cdot 9.09\text{k} = 9.09\;\text{V/V}.$$

**Step 2 — Miller multiplied input cap.**

Effective input capacitance at the gate node:

$$C_{in} = C_{gs} + C_{gd}(1+|A_v|) = 50 + 5(10.09) = 50 + 50.5 = 100.5\;\text{fF}.$$

Output node Miller'd cap:

$$C_{out,M} = C_{gd}\!\left(1+\frac{1}{|A_v|}\right) + C_{db} + C_L = 5(1.11) + 5 + 100 = 110.5\;\text{fF}.$$

**Step 3 — Dominant pole (Miller approximation).**

Resistance at gate $\approx R_{sig} = 10\,\text{k}\Omega$ (assuming gate-bias network high-Z).

$$\tau_{in} = R_{sig}\,C_{in} = 10\text{k}\cdot 100.5\text{f} = 1.005\,\text{ns}.$$

Resistance at drain $\approx R_D\|r_o = 9.09\,\text{k}\Omega$:

$$\tau_{out} = 9.09\text{k}\cdot 110.5\text{f} = 1.004\,\text{ns}.$$

**Step 4 — OCTC refinement.**

For $C_{gd}$ alone, the resistance seen with the other caps open is

$$R_{gd}^{OCTC} = R_{sig} + R_D' + g_m R_{sig}R_D' = 10\text{k}+9.09\text{k}+(1\text{m})(10\text{k})(9.09\text{k}) = 19.1\text{k}+90.9\text{k} = 110\text{k}\Omega.$$

$$\tau_H = R_{sig}C_{gs} + R_{gd}^{OCTC}\,C_{gd} + R_D'(C_{db}+C_L)$$

$$= 10\text{k}\cdot 50\text{f} + 110\text{k}\cdot 5\text{f} + 9.09\text{k}\cdot 105\text{f} = 0.5 + 0.55 + 0.954\;\text{ns} = 2.00\;\text{ns}.$$

$$f_H \approx \frac{1}{2\pi\tau_H} = \frac{1}{2\pi(2.00\text{n})} = \boxed{79.6\;\text{MHz}}.$$

**Step 5 — Unity-gain bandwidth.**

$$f_{UGB} \approx |A_v|\cdot f_H = 9.09\cdot 79.6\text{M} = \boxed{724\;\text{MHz}}.$$

(Or, equivalently, $f_T \approx g_m/[2\pi(C_{gs}+C_{gd})] = 1\text{m}/[2\pi(55\text{f})] = 2.89\,\text{GHz}$, the device's intrinsic limit.)

**Step 6 — RHP zero.**

The forward path through $C_{gd}$ creates

$$\omega_z = \frac{g_m}{C_{gd}} = \frac{1\text{m}}{5\text{f}} = 2\times 10^{11}\,\text{rad/s} \;\Rightarrow\; f_z = \frac{\omega_z}{2\pi} = \boxed{31.8\;\text{GHz}}.$$

Far above $f_{UGB}$ → no PM degradation in this single-stage case.

---

## Problem 8 — Two-Stage Op-Amp Frequency Response (Part B Q3, Tier 6)

### Problem
A two-stage CMOS op-amp has stage gains $A_1 = -100$ V/V and $A_2 = -50$ V/V. The output of stage 1 drives the gate of stage 2. Stage-1 output resistance $R_1 = 200\,\text{k}\Omega$, stage-2 output resistance $R_2 = 100\,\text{k}\Omega$. Without compensation, the two pole capacitances are $C_1 = 0.5\,\text{pF}$ (at node $V_1$) and $C_2 = 2\,\text{pF}$ (at output). A Miller compensation cap $C_C = 5\,\text{pF}$ is added between $V_1$ and $V_{out}$.

(a) Find DC gain. (b) Find dominant pole, non-dominant pole (with pole-splitting), UGB. (c) RHP zero location. (d) Phase margin if PM = 90° − atan(UGB/p2) − atan(UGB/z) with PM target of 60°.

```
   vIN ──[Stage 1]──┬──[Stage 2]── vOUT
                   V1                     ↑
                   │                      │
                   └─────[Cc = 5 pF]──────┘
                   │                      │
                  C1=0.5p                C2=2p
```

### Solution

**Step 1 — DC gain.**

$$A_0 = A_1\cdot A_2 = (-100)(-50) = 5000\;\text{V/V} \;\Rightarrow\; 74\,\text{dB}.$$

**Step 2 — Dominant pole (Miller pole-splitting).**

The Miller cap reflects to node $V_1$ as $C_C(1+|A_2|) = 5\text{p}\cdot 51 = 255\,\text{pF}$.

Total cap at $V_1$: $C_1 + C_C(1+|A_2|) \approx 255\,\text{pF}$.

$$f_{p1} = \frac{1}{2\pi R_1[C_1 + C_C(1+|A_2|)]} = \frac{1}{2\pi(200\text{k})(255\text{p})} = \frac{1}{2\pi\cdot 5.10\times 10^{-5}}=\boxed{3.12\;\text{kHz}}.$$

**Step 3 — Non-dominant pole (output node, after pole-splitting).**

The output sees $g_{m2}$ as effective conductance through the compensation network. With $A_2 = g_{m2}R_2 = 50$, $g_{m2} = 50/100\text{k}= 0.5\,\text{mA/V}$.

$$f_{p2}\approx \frac{g_{m2}}{2\pi(C_2 + C_C\cdot C_1/(C_C+C_1))} \approx \frac{g_{m2}}{2\pi C_2} \;\text{when } C_C\gg C_1, C_2$$

$$f_{p2} \approx \frac{0.5\text{m}}{2\pi(2\text{p})} = \frac{0.5\text{m}}{1.257\times 10^{-11}} = \boxed{39.8\;\text{MHz}}.$$

**Step 4 — Unity-gain bandwidth.**

$$f_{UGB} = A_0 \cdot f_{p1} = 5000 \cdot 3.12\text{k} = \boxed{15.6\;\text{MHz}}.$$

Equivalent expression (for op-amps): $f_{UGB} = g_{m1}/(2\pi C_C)$. Working backward, $g_{m1} = 2\pi(C_C)(f_{UGB})$. Checks out if $g_{m1} = 0.5\,\text{mA/V}$.

**Step 5 — RHP zero (forward path through Cc).**

$$f_z = \frac{g_{m2}}{2\pi C_C} = \frac{0.5\text{m}}{2\pi(5\text{p})} = \boxed{15.9\;\text{MHz}}.$$

Note $f_z \approx f_{UGB}$ → severe PM degradation (≈45° loss). Mitigation: nulling resistor $R_z = 1/g_{m2}$ in series with $C_C$ moves the zero to $\infty$ or LHP.

**Step 6 — Phase margin (with the RHP zero).**

$$\text{PM} = 90° - \tan^{-1}(f_{UGB}/f_{p2}) - \tan^{-1}(f_{UGB}/f_z)$$

$$= 90° - \tan^{-1}(15.6/39.8) - \tan^{-1}(15.6/15.9)$$

$$= 90° - 21.4° - 44.5° = \boxed{24.1°}.$$

Below 60° target → must add nulling resistor or increase $C_C$ until $f_{p2} \ge 2.2\,f_{UGB}$.

---

## Problem 9 — Series-Shunt Feedback (Voltage Amplifier) (Part B Q4, Tier 7)

### Problem
A two-stage MOSFET amplifier (CS → CD source-follower) has open-loop voltage gain $A = 1000$ V/V, input resistance $R_i = 1\,\text{M}\Omega$, output resistance $R_o = 1\,\text{k}\Omega$, and bandwidth $f_H = 1\,\text{MHz}$. Resistors $R_F = 9\,\text{k}\Omega$ and $R_E = 1\,\text{k}\Omega$ form a feedback divider from $v_{out}$ back to a series node at the source of the input transistor.

(a) Identify the feedback topology. (b) Find $\beta$. (c) Find $A_f, R_{if}, R_{of}, f_{Hf}$.

```
   vS ──[Rsig]──+ ─── A (gain block) ─── vO
                │                         │
                │                        [RF=9k]
              [RE=1k]                     │
                │                         │
                └─────────────────────────┴── ground
              (RE samples vO scaled by RE/(RE+RF))
```

### Solution

**Step 1 — Topology.**

Output is sampled in **shunt** (across the load → senses voltage). Feedback signal subtracts in **series** with the input (a voltage drop on $R_E$). → **Series-Shunt** (voltage amplifier with $A$ in V/V, $\beta$ in V/V).

**Step 2 — Feedback factor.**

$$\beta = \frac{R_E}{R_E + R_F} = \frac{1}{1+9} = \boxed{0.10\;\text{V/V}}.$$

**Step 3 — Loop gain and closed-loop gain.**

$$T = A\beta = 1000\cdot 0.10 = 100.$$

$$A_f = \frac{A}{1+A\beta} = \frac{1000}{101} = \boxed{9.90\;\text{V/V}.}$$

(Approximately $1/\beta = 10$ ✓.)

**Step 4 — Input resistance (series feedback boosts $R_i$).**

$$R_{if} = R_i(1+T) = 1\text{M}\cdot 101 = \boxed{101\;\text{M}\Omega}.$$

**Step 5 — Output resistance (shunt feedback reduces $R_o$).**

$$R_{of} = \frac{R_o}{1+T} = \frac{1\text{k}}{101} = \boxed{9.9\;\Omega.}$$

**Step 6 — Bandwidth (gain–BW product preserved).**

$$f_{Hf} = f_H(1+T) = 1\text{M}\cdot 101 = \boxed{101\;\text{MHz}.}$$

(Sanity: $A_f \cdot f_{Hf} = 9.9 \cdot 101\text{M} = 1000\text{ M} = A\cdot f_H$ ✓.)

---

## Problem 10 — Shunt-Shunt Feedback (Transimpedance) (Part B Q4, Tier 7)

### Problem
A CS amplifier with open-loop transresistance $R_M = -10^6\,\text{V/A}$, input resistance $R_i = 100\,\text{k}\Omega$, output resistance $R_o = 50\,\text{k}\Omega$, and bandwidth $f_H = 100\,\text{kHz}$ is wrapped with feedback resistor $R_F = 100\,\text{k}\Omega$ from drain to gate (single-ended). The signal source is a current $i_S$ in parallel with $R_S = 1\,\text{M}\Omega$.

(a) Identify the topology. (b) Compute $\beta$ (in A/V). (c) Closed-loop transresistance $R_{Mf}$, $R_{if}$, $R_{of}$, $f_{Hf}$.

```
                  ┌──[RF = 100k]──┐
                  │               │
   iS ┄┤RS=1M├────┤ G       D ────┤── vO
                  │               │
                  └─[A: CS amp]───┘
                      Rin = 100k
                      Rout = 50k
                      RM = -10^6 V/A (open-loop)
```

### Solution

**Step 1 — Topology.**

The feedback resistor $R_F$ samples the output **voltage** (shunt sensing) and feeds back a **current** at the input node (shunt mixing). → **Shunt-Shunt** (transresistance amp). $A$ has units of V/A; $\beta$ has units of A/V.

**Step 2 — Feedback factor.**

$$\beta = -\frac{1}{R_F} = -\frac{1}{100\text{k}} = -10\,\mu\text{A/V}.$$

(Sign chosen so that $T = A\beta > 0$.)

**Step 3 — Loop gain.**

$$T = A\beta = (-10^6)(-10^{-5}) = 10.$$

**Step 4 — Closed-loop transresistance.**

$$R_{Mf} = \frac{A}{1+T} = \frac{-10^6}{11} = \boxed{-9.09\times 10^4\;\text{V/A}.}$$

$\Rightarrow$ closed-loop converts 1 μA input into about 90 mV output.

**Step 5 — Input resistance (shunt feedback at input → divided).**

$$R_{if} = \frac{R_i}{1+T} = \frac{100\text{k}}{11} = \boxed{9.09\;\text{k}\Omega.}$$

(Bonus check: total node resistance, with $R_F$ in shunt across input, is $R_{if}\,\|\,R_F$ ≈ 8.3 kΩ.)

**Step 6 — Output resistance.**

$$R_{of} = \frac{R_o}{1+T} = \frac{50\text{k}}{11} = \boxed{4.55\;\text{k}\Omega.}$$

**Step 7 — Bandwidth.**

$$f_{Hf} = f_H(1+T) = 100\text{k}\cdot 11 = \boxed{1.1\;\text{MHz}.}$$

**Step 8 — Voltage gain seen by source.**

The current divider $R_S \| R_{if}$ shunts $i_S$. Useful voltage at $v_O$:

$$v_O = i_S\cdot\!\left(\frac{R_S R_{if}}{R_S+R_{if}}\right)\cdot \text{(transimpedance factor inside loop)}\;\;\rightarrow\;\;\text{exam usually stops at }R_{Mf}.$$

---

## Quick-Reference Map (use during the exam)

| If you see…                                                               | Tier | Solution path                                           |
| ------------------------------------------------------------------------- | ---- | ------------------------------------------------------- |
| Self-bias network → find IDQ                                              | 2    | Quadratic in $V_{GS}$, accept root with $V_{GS}>V_T$    |
| "Determine region"                                                        | 1    | Compute $V_{ov}$, compare $V_{DS}$ to $V_{ov}$          |
| "Size $W/L$" mirror                                                       | 4    | $I_D = (\mu C_{ox}/2)(W/L)V_{ov}^2$, pick $V_{ov}=0.2$  |
| "Maximize $R_{out}$"                                                      | 3,4  | Cascode ⇒ $R_{out} \approx g_m r_o\,r_o$                |
| Diff-pair w/ resistive $R_D$                                              | 5    | Half-circuit, $A_{dm}=-g_m R_D'$, CMRR $= g_m R_{SS}$   |
| Diff-pair w/ mirror load                                                  | 5    | $A_{dm}=g_m(r_{on}\|r_{op})$ to single-ended            |
| Capacitances given, find $f_H$                                            | 6    | OCTC: $\tau_H = \sum R_k C_k$ ; $R_{gd}\sim g_m R_S R_L'$ |
| 2-stage with $C_C$                                                        | 6    | $f_{p1} = 1/[2\pi R_1 C_C A_2]$; $f_z = g_m/(2\pi C_C)$ |
| Feedback: figure out 2×2 (input/output × series/shunt)                    | 7    | Topology → $\beta$ → $A_f, R_{if}, R_{of}, f_{Hf}$      |

Good luck. Show every formula plug-in step — partial credit is awarded for correct setup even with a numerical slip.
