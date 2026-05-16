# ECE F244 Microelectronic Circuits — Compre Study Guide

**Exam:** 11 May 2026 · 3 hrs · Part A Closed Book (40 marks, 60 min) + Part B Open Book (70 marks, 2 hrs)
**Source PYQs ingested:** 2011-12, 2012-13, 2015-16, 2017-18, 2021-22, 2022-23

> **Status:** Tiers 1–6 complete. Tier 7 (Feedback) pending.

---

## Master Parameters (Memorise — Part A is closed book)

$$
V_{DD} = V_{CC} = 3.3\text{ V}
$$

| Device | $\mu C_{ox}$ | $V_T$ | $\lambda$ | $V_{ov}$ |
|---|---|---|---|---|
| NMOS | $140\,\mu\text{A/V}^2$ | $+0.7$ V | $0.1$ V$^{-1}$ | $0.2$ V |
| PMOS | $40\,\mu\text{A/V}^2$ | $-0.7$ V | $0.1$ V$^{-1}$ | $0.2$ V |
| BJT | $\beta = 100$, $V_{CE,sat}=0.2$ V, $V_A=100$ V, $V_T=kT/q=25$ mV, $V_{BE,on}=0.6$ V, $\alpha\approx 1$ | | | |

## Master Equations

$$
\boxed{I_D = \tfrac{1}{2}\mu C_{ox}\,\tfrac{W}{L}\,(V_{GS}-V_T)^2(1+\lambda V_{DS})} \quad\text{(saturation)}
$$
$$
I_D = \mu C_{ox}\,\tfrac{W}{L}\!\left[(V_{GS}-V_T)V_{DS}-\tfrac{V_{DS}^2}{2}\right] \quad\text{(triode)}
$$
$$
g_m = \frac{2 I_D}{V_{ov}} = \sqrt{2\mu C_{ox}\tfrac{W}{L}I_D},\quad
r_o = \frac{V_A}{I_D} = \frac{1}{\lambda I_D},\quad
A_0 = g_m r_o = \frac{2 V_A}{V_{ov}}
$$

---

# TIER 1 — MOSFET Device Problems

> Building blocks. Every Part A question opens here. ~8–10 marks across small parts.

---

## Type 1A — Operating Region Determination

**Theory.** Determine ON/OFF first ($V_{GS}>V_T$ for NMOS), then compare $|V_{DS}|$ vs $|V_{ov}|$. Saturation if $|V_{DS}|\ge|V_{ov}|$, else triode. PMOS analysis uses absolute values (source is the higher-potential terminal).

**Algorithm.**
1. Identify type, locate $S$ and $D$ (S = lower for NMOS, higher for PMOS).
2. Compute $V_{GS}$, confirm device is ON.
3. $V_{ov} = V_{GS}-V_T$ (use signed values; for PMOS take magnitudes).
4. Compare $|V_{DS}|$ to $|V_{ov}|$ → pick saturation or triode formula.

### Worked Example — PYQ 2022-23 Part A Q2(B)
> *`PYQs/Compre Qp.pdf` p.1 — "For a p-channel long MOSFET..."*

PMOS, $V_T=-1$ V, $W=20\,\mu$m, $L=2\,\mu$m, $\mu_p=200$ cm²/V·s, $C_{ox}=3.5\times 10^{-7}$ F/cm². Source/body grounded.

$$
\mu_p C_{ox} = 200 \times 3.5\times 10^{-7} = 70\,\mu\text{A/V}^2,\quad W/L = 10
$$

**(a) $V_G=-4$ V, $V_D=-1$ V:**
$$
V_{GS}=-4,\ V_{ov}=-3\text{ V } (|V_{ov}|=3),\ V_{DS}=-1\ \Rightarrow |V_{DS}|<|V_{ov}|\Rightarrow\text{TRIODE}
$$
$$
I_D = \mu_p C_{ox}\tfrac{W}{L}\!\left[|V_{ov}||V_{DS}|-\tfrac{V_{DS}^2}{2}\right] = 70\mu\cdot 10\cdot[3\cdot 1 - 0.5] = \boxed{1.75\text{ mA}}
$$

**(b) $V_G=-3$ V, $V_D=-3$ V:**
$$
|V_{ov}|=2,\ |V_{DS}|=3\ge |V_{ov}|\Rightarrow\text{SATURATION}
$$
$$
I_D=\tfrac{1}{2}\mu_p C_{ox}\tfrac{W}{L}V_{ov}^2 = 35\mu\cdot 10\cdot 4 = \boxed{1.4\text{ mA}}
$$

---

## Type 1B — Channel-Length Modulation, $\Delta I_D$, $r_o$

**Theory.** Saturation current rises slightly with $V_{DS}$ via $(1+\lambda V_{DS})$. Slope of this rise gives the device output resistance.

**Formulas.**
$$
\Delta I_D = I_D\cdot\lambda\cdot\Delta V_{DS},\quad r_o = \frac{1}{\lambda I_D}=\frac{V_A}{I_D}
$$

### Worked Example — PYQ 2015-16 Part A Q5
> *`PYQs/2016 mue compre.pdf` p.2*

$I_D=1$ mA at $V_{DS}=0.5$ V, $\lambda=0.1$ V$^{-1}$, $V_{DS}\to 1$ V.

$$
\Delta I_D = 1\text{ mA}\cdot 0.1\cdot 0.5 = \boxed{50\,\mu\text{A}},\quad
r_o = \frac{1}{0.1\cdot 1\text{ mA}}=\boxed{10\text{ k}\Omega}
$$

---

## Type 1C — Body Effect (Threshold Shift)

**Theory.** When $V_{SB}\ne 0$, the bulk acts as a back-gate and raises $V_T$.

**Formula.**
$$
V_{TH} = V_{TH0} + \gamma\!\left(\sqrt{2\phi_F+V_{SB}}-\sqrt{2\phi_F}\right)
$$

### Worked Example — PYQ 2015-16 Part A Q4
> *`PYQs/2016 mue compre.pdf` p.2*

$V_S=0.5$ V, $V_{TH0}=0.6$ V, $\gamma=0.4$ V$^{1/2}$, $\phi_F=0.4$ V (so $2\phi_F=0.8$). Then $V_G=V_D=1.4$ V, $\mu_n C_{ox}=100\,\mu$A/V², $W/L=50$, $\lambda=0$.

$$
V_{SB}=0.5\Rightarrow V_{TH}=0.6+0.4\!\left(\sqrt{1.3}-\sqrt{0.8}\right)=0.6+0.4(1.140-0.894)=\boxed{0.70\text{ V}}
$$
$$
V_{GS}=0.9,\ V_{ov}=0.2,\ V_{DS}=0.9\ge V_{ov}\Rightarrow\text{SAT}
$$
$$
I_D = 50\mu\cdot 50\cdot 0.04 = \boxed{100\,\mu\text{A}}
$$

---

## Type 1D — Intrinsic Gain $g_m r_o$

**Theory.** Maximum single-stage gain. Depends only on $V_{ov}$ and $V_A$ (not $W/L$ or $I_D$).

**Formulas.**
$$
g_m r_o = \frac{2 V_A}{V_{ov}}
$$

### Worked Example — PYQ 2015-16 Part A Q2
> *`PYQs/2016 mue compre.pdf` p.1*

$W/L=100$, $I_D=0.5\,\mu$A, defaults ($\mu_n C_{ox}=140\,\mu$A/V², $V_A=100$ V).
$$
g_m=\sqrt{2\cdot 140\mu\cdot 100\cdot 0.5\mu}=\sqrt{1.4\times 10^{-8}}\approx 118.3\,\mu\text{A/V}
$$
$$
r_o=\tfrac{100}{0.5\,\mu\text{A}}=200\text{ M}\Omega,\quad g_m r_o\approx \boxed{2.37\times 10^4\ (\approx 87\text{ dB})}
$$

---

## Type 1E — Sizing for Target $I_D$ or Edge of Saturation

**Formula.** From $I_D=\tfrac{1}{2}\mu C_{ox}\tfrac{W}{L}V_{ov}^2$, solve for $W/L$. Edge of saturation: $V_{DS}=V_{ov}$.

### Worked Example — PYQ 2015-16 Part A Q6
> *`PYQs/2016 mue compre.pdf` p.2*

$V_{DD}=1.8$ V, $R_D=5$ k$\Omega$, $V_G=1$ V, $W/L=2/0.18$, $\mu_n C_{ox}=100\,\mu$A/V², $V_T=0.4$ V.

**(i) $I_D$.** $V_{ov}=0.6$ V, assume sat:
$$
I_D=50\mu\cdot 11.11\cdot 0.36=200\,\mu\text{A};\ V_{DS}=1.8-1.0=0.8\ge V_{ov}\,\checkmark\Rightarrow \boxed{200\,\mu\text{A}}
$$

**(ii) $(W/L)_\text{max}$ at edge of saturation.** $V_{DS}=V_{ov}=0.6$:
$$
I_D=\tfrac{1.8-0.6}{5\text{k}}=240\,\mu\text{A}=50\mu\cdot(W/L)\cdot 0.36\Rightarrow \boxed{(W/L)_\text{max}=13.33}
$$

---

# TIER 2 — DC Biasing

> First half of every Part A Q1, plus Part B Q2(a) load-line problems. ~8–12 marks.

---

## Type 2A — Self-Bias CS (Drain-to-Gate $R_F$)

**Theory.** Large $R_F$ (≥100 k$\Omega$) gate-to-drain forces $V_G=V_D$ at DC (no gate current), so $V_{GS}=V_{DS}$ — automatic saturation.

**Setup.**
$$
V_{GS}=V_{DS}=V_{DD}-I_D R_D
$$
Substitute into $I_D=\tfrac{1}{2}\mu C_{ox}\tfrac{W}{L}(V_{GS}-V_T)^2$ → quadratic in $I_D$.

### Worked Example — PYQ 2022-23 Part A Q1
> *`PYQs/Compre Qp.pdf` p.1*

$R_F=100$ k, $R_D=R_L=5$ k, $K_n'=20\,\mu$A/V², $W/L=20$, $V_A=100$ V, $V_T=2$ V, $V_{DD}=12$ V.

$$
k=\tfrac{1}{2}K_n'(W/L)=200\,\mu\text{A/V}^2
$$
$$
I_D = k(V_{GS}-V_T)^2 = 200\mu(12-5000I_D-2)^2
$$
$$
\Rightarrow 5000 I_D^2 - 21 I_D + 0.02 = 0
$$
$$
I_D = \frac{21\pm\sqrt{441-400}}{10000}=\frac{21\pm 6.40}{10000}
$$
- $I_D=2.74$ mA → $V_{GS}=-1.7$ V (reject, $V_{GS}<V_T$)
- $I_D=1.46$ mA → $V_{GS}=4.7$ V ✓

$$
\boxed{I_{DQ}=1.46\text{ mA},\ V_{GSQ}=V_{DSQ}=4.7\text{ V},\ V_{ov}=2.7\text{ V}}
$$
$$
g_m=\frac{2\cdot 1.46m}{2.7}=1.08\text{ mA/V},\quad r_o=\frac{100}{1.46m}=68.5\text{ k}\Omega
$$

---

## Type 2B — Voltage-Divider Bias

**Theory.** $R_1, R_2$ form gate divider; $V_G = V_{DD}\cdot R_2/(R_1+R_2)$. The Thevenin gate resistance $R_{th}=R_1\|R_2$ is usually given.

**Solve for $R_1, R_2$ given $V_G$ and $R_{th}$:**
$$
R_1 = R_{th}\cdot\frac{V_{DD}}{V_G},\quad R_2 = R_{th}\cdot\frac{V_{DD}}{V_{DD}-V_G}
$$

### Worked Example — PYQ 2022-23 Part B Q2(A)
> *`PYQs/Compre Qp.pdf` p.2*

$W/L=25$, $R_1\|R_2=100$ k, $V_{TN}=1$ V, $I_{DQ}=2$ mA, $R_D=2.5$ k, source-degenerated NMOS CS amp. Q-point at middle of saturation.

$$
2\text{ mA}=70\mu\cdot 25\cdot V_{ov}^2 \Rightarrow V_{ov}^2=1.143\Rightarrow V_{ov}=1.07\text{ V}
$$
$$
V_{GSQ}=V_T+V_{ov}=2.07\text{ V}
$$
Assuming $V_S\approx 0$ (or expressed in terms of $R_S$ if given): $V_G=V_{GSQ}=2.07$ V. With $V_{DD}=12$ V (read from figure):
$$
R_1 = 100\text{k}\cdot\tfrac{12}{2.07}\approx \boxed{580\text{ k}\Omega},\quad R_2=100\text{k}\cdot\tfrac{12}{9.93}\approx \boxed{121\text{ k}\Omega}
$$

---

## Type 2C — Load Line

**Equation.** $I_D = (V_{DD}-V_{DS})/(R_D+R_S)$; line from $(0, V_{DD}/(R_D+R_S))$ to $(V_{DD}, 0)$. Saturation boundary at $V_{DS}=V_{ov}$.

### Worked Example — Same as 2B
$V_{DD}=12$ V (assumed), $R_D=2.5$ k, $R_S=0$:
$$
I_{D,\max}=\frac{12}{2.5\text{k}}=4.8\text{ mA},\quad V_{DS,\max}=12\text{ V}
$$
Q at $(V_{DSQ}\approx 6.5\text{ V},\ 2\text{ mA})$, sat boundary at $V_{DS}=1.07$ V.

---

# TIER 3 — Single-Stage Small-Signal Amplifiers

> Workhorse for Part A Q1 and Part B Q2(b). ~15–25 marks.

---

## Type 3A — Common Source (CS)

**Theory.** Inverting voltage gain. $R_\text{in}=\infty$, $R_\text{out}=R_D\|r_o$.

$$
A_v = -g_m(R_D\|r_o\|R_L),\quad R_\text{out}=R_D\|r_o
$$
For self-bias (Miller): $R_\text{in}=R_F/(1+|A_v|)$.

### Worked Example — PYQ 2022-23 Part A Q1 (continued)
With $g_m=1.08$ mA/V, $r_o=68.5$ k, $R_D=R_L=5$ k, $R_F=100$ k:
$$
R_D\|r_o\|R_L = 2.5\text{k}\|68.5\text{k}=2.41\text{ k}\Omega
$$
$$
A_v=-1.08m\cdot 2.41\text{k} = \boxed{-2.6\text{ V/V}}
$$
$$
R_\text{in}=\frac{R_F}{1+|A_v|}=\frac{100\text{k}}{3.6}=\boxed{27.8\text{ k}\Omega},\quad R_\text{out}=R_D\|r_o\approx \boxed{4.66\text{ k}\Omega}
$$
$$
A_i=A_v\cdot\frac{R_\text{in}}{R_L}=-2.6\cdot\frac{27.8}{5}=\boxed{-14.5\text{ A/A}}
$$

### Worked Example — PYQ 2015-16 Part A Q7
> *`PYQs/2016 mue compre.pdf` p.3*

$V_{DD}=1.8$ V, $R_D=1$ k, $V_{DS}=0.8$ V, $V_T=0.5$ V, $W/L=10/0.18$, $\mu_n C_{ox}=100\,\mu$A/V², $\lambda=0$.

$$
I_D=\frac{1.8-0.8}{1\text{k}}=1\text{ mA};\ 1m=50\mu\cdot 55.56\cdot V_{ov}^2\Rightarrow V_{ov}=0.6
$$
$$
g_m=\frac{2\cdot 1m}{0.6}=3.33\text{ mA/V},\quad A_v=-g_m R_D=\boxed{-3.33\text{ V/V}}
$$

---

## Type 3B — CS with Source Degeneration

**Theory.** $R_S$ in source provides series-series local feedback: gain shrinks by $(1+g_m R_S)$, $R_\text{out}$ rises.

$$
A_v = -\frac{g_m R_D}{1+g_m R_S}\quad(r_o\to\infty)
$$
$$
R_\text{out}=R_D\,\|\,[r_o(1+g_m R_S)+R_S]\approx R_D\,\|\,(g_m r_o R_S)
$$
If $R_S$ is a diode-connected MOSFET $M_d$: $R_S=1/g_{m,d}$.

### Worked Example — PYQ 2021-22 Part A Q3 (b–d)
> *`PYQs/MuE 2022 Compre.pdf` p.2*

CSA with PMOS active-load mirror, then add diode-connected NMOS $M_d$ as source degeneration. $W_N=10\,\mu$m, $W_P=20\,\mu$m, $L=1\,\mu$m, $V_{ov}=0.2$ V, $I_{REF}=20\,\mu$A, current ratio = 2 → $I_D=40\,\mu$A.

$$
g_{m,M1}=g_{m,Md}=\frac{2\cdot 40\mu}{0.2}=0.4\text{ mA/V}
$$
$$
R_S=\frac{1}{g_{m,Md}}=2.5\text{ k}\Omega,\quad r_{o,N}=r_{o,P}=\frac{100}{40\mu}=2.5\text{ M}\Omega
$$
$$
A_v=-\frac{g_m\cdot(r_{o,N}\|r_{o,P})}{1+g_m R_S}=-\frac{0.4m\cdot 1.25\text{M}}{1+0.4m\cdot 2.5\text{k}}=-\frac{500}{2}=\boxed{-250\text{ V/V}}
$$

---

## Type 3C — Source Follower (CD)

**Theory.** Gate input, source output. Voltage gain $\approx 1$, low $R_\text{out}\approx 1/g_m$. Buffer.

$$
A_v=\frac{g_m R_S}{1+g_m R_S},\quad R_\text{out}=\tfrac{1}{g_m}\,\|\,r_o\,\|\,R_S
$$

### Worked Example — PYQ 2015-16 Part A Q10
> *`PYQs/2016 mue compre.pdf` p.4*

Target $A_v=0.5$, $R_S=50\,\Omega$, $I_D=5.56$ mA, $\mu_n C_{ox}=100\,\mu$A/V², $V_T=0.5$ V.

$$
0.5=\frac{50\,g_m}{1+50\,g_m}\Rightarrow g_m=\frac{1}{R_S}=\boxed{20\text{ mA/V}}
$$
$$
g_m^2=2\mu_n C_{ox}\,\tfrac{W}{L}\,I_D\Rightarrow \tfrac{W}{L}=\frac{(20m)^2}{2\cdot 100\mu\cdot 5.56m}=\boxed{360}
$$

---

## Type 3D — Common Gate (CG)

**Theory.** Source input, drain output, gate AC ground. Non-inverting; $R_\text{in}=1/g_m$.

$$
A_v=g_m(R_D\|r_o),\quad R_\text{in}=\frac{1}{g_m},\quad \frac{dI_\text{out}}{dV_\text{in}}=g_m
$$

### Worked Example — PYQ 2015-16 Part A Q1
> *`PYQs/2016 mue compre.pdf` p.1*

Cascode stack: $M_2$ CS bottom (gate=$V_\text{in}$), $M_1$ CG top ($V_b$ at gate). $I_\text{out}$ from $M_1$ drain. At constant $V_\text{out}$:
$$
\boxed{\frac{dI_\text{out}}{dV_\text{in}}=g_{m,M2}=g_m}
$$

---

## Type 3E — Cascode

**Theory.** CS+CG stack: output resistance is multiplied by intrinsic gain.

$$
R_\text{out,cas} = r_{o2}+r_{o1}+g_{m2}r_{o1}r_{o2}\approx g_m r_o^2
$$
$$
A_v = -g_{m1}\cdot R_\text{out,cas}\approx -(g_m r_o)^2 \quad\text{(ideal load)}
$$
With finite load: $A_v=-g_{m1}(R_L\|R_\text{out,cas})$.

### Worked Example — PYQ 2015-16 Part A Q8
> *`PYQs/2016 mue compre.pdf` p.3*

Identical $M_1$ (CG, top, $V_b$ gate), $M_2$ (CS, bottom). Find $R_\text{out}$:
$$
R_\text{out}=r_o(1+g_m r_o)+r_o\approx \boxed{g_m r_o^2}
$$

### Worked Example — PYQ 2015-16 Part B Q1
> *`PYQs/2016 mue compre.pdf` p.5*

PMOS cascode load + NMOS CS input. $\mu C_{ox}\,W/L=1400\,\mu$A/V², $I_D=100\,\mu$A, $V_T=0.7$ V, $\lambda=0.1$.
$$
g_m=\sqrt{2\cdot 1400\mu\cdot 100\mu}=0.529\text{ mA/V},\quad r_o=\frac{100}{100\mu}=1\text{ M}\Omega
$$
$R_\text{out}=$ NMOS-side $r_o$ $\|$ PMOS-cascode $\approx r_{o,N}=1$ M$\Omega$.
$$
|A_v|=g_m R_\text{out}=0.529m\cdot 1\text{M}=\boxed{529\text{ V/V}}
$$
For full cascode (NMOS cascoded too): $R_\text{out}=(g_m r_o^2)/2$, $A_v\approx -(g_m r_o)^2/2$.

---

## Type 3F — CS with PMOS Active Load

**Theory.** PMOS current source acts as load with $r_{o,P}$. Gain $\approx -g_m r_o/2$ for matched.

$$
A_v=-g_{m,N}(r_{o,N}\|r_{o,P})\xrightarrow{\text{matched}}-\tfrac{1}{2}g_m r_o = -\frac{V_A}{V_{ov}}
$$

### Worked Example — PYQ 2015-16 Part A Q9
> *`PYQs/2016 mue compre.pdf` p.3*

NMOS $M_1$ CS, PMOS $M_2$ as current source. Matched, default $V_{ov}=0.2$ V, $V_A=100$ V:
$$
A_v=-\frac{V_A}{V_{ov}}=\boxed{-500\text{ V/V}}
$$

---

# TIER 4 — Current Mirrors

> Part A Q2 staple; bias every diff amp and OpAmp. ~10–20 marks.

---

## Type 4A — Basic Mirror Sizing

**Theory.** Reference is diode-connected; output(s) share $V_{GS}$. With same $L$:

$$
\frac{I_O}{I_\text{REF}}=\frac{(W/L)_O}{(W/L)_\text{REF}}\quad\text{(ideal)}
$$
$z$ devices in parallel: replace $(W/L)_O$ by $z\cdot(W/L)_\text{each}$.

For BJT mirror with finite $\beta$ and $n$ outputs:
$$
\frac{I_O}{I_\text{REF}}=\frac{1}{1+\frac{n+1}{\beta}}
$$

### Worked Example — PYQ 2022-23 Part A Q2(A)
> *`PYQs/Compre Qp.pdf` p.1*

NMOS mirror with $I_\text{REF}=10\,\mu$A; outputs $I_{D1}=60$, $I_{D2}=20$, $I_{D4}=80\,\mu$A. $\mu_n C_{ox}=200\,\mu$A/V², $\mu_p C_{ox}=60\,\mu$A/V², $L=1\,\mu$m, $V_{ov}=0.2$ V (NMOS at edge: $V_D=-1.3$ V, $V_{SS}=-1.5$ V).

$$
I_D=\tfrac{1}{2}\mu_n C_{ox}\tfrac{W}{L}V_{ov}^2 = 100\mu\cdot W \cdot 0.04 = 4\mu\cdot W
$$
$$
W_1=\tfrac{60}{4}=\boxed{15\,\mu\text{m}},\ W_2=\tfrac{20}{4}=\boxed{5\,\mu\text{m}}
$$
For $M_4$ (PMOS): $I_D=30\mu\cdot W_4\cdot 0.04=1.2\mu\cdot W_4$
$$
W_4 = \tfrac{80}{1.2}=\boxed{66.7\,\mu\text{m}}
$$

### Worked Example — PYQ 2017-18 Part B Q2(A)
> *`PYQs/2018 Compre QP.pdf` p.1*

$M_\text{REF}$ has $343\cdot(W/L)$. Output is $z$ parallel devices each with $p^2(W/L)$. $I_\text{OUT}=I_\text{REF}$, $z, p$ whole numbers.

$$
z\cdot p^2 = 343 = 7^3\Rightarrow \boxed{z=7,\ p=7}
$$

### Worked Example — PYQ 2012-13 Part A Q1(B)
> *`PYQs/Compre with sol.pdf` p.1*

Generate 1, 2, 4 mA from $I_\text{REF}=7$ mA BJT mirror.

**$\beta=\infty$:** Areas $A_1:A_2:A_3:A_\text{REF}=1:2:4:7$.

**$\beta=50$:** With $n=3$ outputs:
$$
\frac{I_O}{I_\text{REF}}=\frac{1}{1+\frac{4}{50}}=\frac{50}{54}=0.926
$$
$$
I_1=0.926,\ I_2=1.852,\ I_3=3.704\text{ mA (each ~7.4% low)}
$$

---

## Type 4B — Cascode and Low-Voltage Cascode Mirrors

**Theory.** Cascoding raises $R_\text{out}$ from $r_o$ to $g_m r_o^2$. Low-voltage cascode (LVC) sacrifices headroom only down to $2 V_{ov}$ instead of $V_T+2V_{ov}$.

**Key relation for LVC:** Bias cascode gate at $V_C=V_T+2V_{ov}$ → bottom transistor's $V_{DS}=V_{ov}$ (at edge), so $V_\text{P,min}=2V_{ov}$.

### Worked Example — PYQ 2017-18 Part B Q2(B)
> *`PYQs/2018 Compre QP.pdf` p.1*

LVC mirror, all NMOS, same $V_{ov}$, $I_\text{REF}=100\,\mu$A, $I_\text{OUT}=2 I_\text{REF}$. Find $V_C$ for $V_\text{P,min}=2V_{ov}$.

For $M_4$ cascode: $V_{S,M4}=V_{DS,M2}$. Force $V_{DS,M2}=V_{ov}$ (M2 at edge of sat):
$$
V_C = V_{S,M4}+V_{GS,M4}=V_{ov}+(V_T+V_{ov})=\boxed{V_T+2V_{ov}}
$$

---

## Type 4C — BJT Cascode Bias Design (CB stage on a CE)

### Worked Example — PYQ 2017-18 Part B Q2(C)
> *`PYQs/2018 Compre QP.pdf` p.2*

NPN cascode: $Q_1$ CE, $Q_2$ CB; $V_{CC}=3.3$, $R_L=2$ k, $R_E=0.5$ k, $\beta=100$, $V_{BE}=0.7$, $V_A=\infty$, $I_C=0.5$ mA, $V_{CE,Q1}=V_{CE,Q2}=0.9$ V, $R_1+R_2+R_3=100$ k.

KVL through $R_C$:
$$
3.3 = 0.5m\,R_C + 0.9 + 0.9 + 0.25\Rightarrow R_C=\boxed{2.5\text{ k}\Omega}
$$
Node voltages: $V_{E1}=0.25$, $V_{B1}=0.95$, $V_{C1}=V_{E2}=1.15$, $V_{B2}=1.85$ V.
$$
I_\text{div}=\tfrac{3.3}{100\text{k}}=33\,\mu\text{A}
$$
$$
R_1=\tfrac{1.45}{33\mu}=\boxed{43.9\text{ k}\Omega},\ R_2=\tfrac{0.9}{33\mu}=\boxed{27.3\text{ k}\Omega},\ R_3=\tfrac{0.95}{33\mu}=\boxed{28.8\text{ k}\Omega}
$$
$$
g_m=\tfrac{0.5m}{0.025}=20\text{ mA/V},\ r_\pi=\tfrac{\beta}{g_m}=5\text{ k}\Omega
$$

---

# TIER 5 — Differential Amplifier

> Part B Q1 — appears every year, ~19–20 marks. The biggest single block on the paper.

---

## Type 5A — DC Bias of Diff Pair

**Theory.** With matched pair and $V_{G1}=V_{G2}$, tail current $I_\text{SS}$ splits equally: $I_D=I_\text{SS}/2$ in each branch. Source node $V_P=V_G-V_{GS}$.

**Algorithm.**
1. $I_D = I_\text{SS}/2$ in each.
2. $V_{GS}=V_T+V_{ov}$, with $V_{ov}=\sqrt{2I_D/[\mu C_{ox}(W/L)]}$.
3. $V_P=V_{G,CM}-V_{GS}$ (NMOS pair).
4. $V_{D1}=V_{D2}=V_{DD}-I_D R_D$ (resistive load) or set by mirror diode.

### Worked Example — PYQ 2022-23 Part B Q1(a)
> *`PYQs/Compre Qp.pdf` p.2*

NMOS diff pair, $I_\text{SS}=90\,\mu$A, $R_{D1}=R_{D2}=40$ k, $V_{ov}=0.2$ V, defaults. $v_\text{in1}=5\sin\omega t+1.4$ V, $v_\text{in2}=10\sin\omega t+1.4$ V (mV for the AC). Find DC at $P$ and $X$ (drain-1).

$$
I_D=\tfrac{90\mu}{2}=45\,\mu\text{A};\ V_{GS}=V_T+V_{ov}=0.9\text{ V}
$$
$$
\boxed{V_P=V_{CM}-V_{GS}=1.4-0.9=0.5\text{ V}}
$$
$$
\boxed{V_X=V_{DD}-I_D R_D=3.3-45\mu\cdot 40\text{k}=3.3-1.8=1.5\text{ V}}
$$

---

## Type 5B — Differential-Mode Gain (Resistive Load)

**Theory.** Half-circuit: source node $P$ is virtual AC ground for differential excitation. Each half is just a CS amp with $v_{gs}=v_{id}/2$.

**Formulas.**
$$
A_{dm}^\text{single} = \frac{v_{o1}}{v_{id}}=-\frac{g_m R_D}{2}
$$
$$
A_{dm}^\text{diff} = \frac{v_{o2}-v_{o1}}{v_{id}}=-g_m R_D
$$
With finite $r_o$: replace $R_D$ by $R_D\|r_o$.

### Worked Example — PYQ 2022-23 Part B Q1(b–e)

From 5A: $g_m=2\cdot 45\mu/0.2=0.45$ mA/V, $R_D=40$ k.

**(b) $A_{dm}$ for $v_\text{out}=v_{o2}-v_{o1}$, $v_\text{in}=v_{i1}-v_{i2}$:**
$$
A_{dm}=-g_m R_D = -0.45m\cdot 40\text{k}=\boxed{-18\text{ V/V}}
$$

**(c) CM/DM decomposition.** AC parts: $v_{i1}=5\sin$, $v_{i2}=10\sin$ (mV).
$$
v_{CM} = \tfrac{v_{i1}+v_{i2}}{2}=7.5\sin\omega t\text{ mV},\quad v_{id}=v_{i1}-v_{i2}=-5\sin\omega t\text{ mV}
$$
DM components: $v_{i1,DM}=v_{id}/2=-2.5\sin$ mV, $v_{i2,DM}=+2.5\sin$ mV. CM components: each $=7.5\sin$ mV.

**(d) Total $V_\text{out}$ AC** (assuming $A_{cm}=0$ from matching):
$$
V_\text{out}^{AC}=A_{dm}\cdot v_{id}=(-18)(-5\sin\omega t\text{ mV})=\boxed{+90\sin\omega t\text{ mV}}
$$

**(e) Plot.** DC component $V_\text{out,DC}=V_X-V_X=0$. AC sinusoid amplitude 90 mV centered at 0 V.

---

## Type 5C — Differential Gain (Active / Mirror Load)

**Theory.** Replace $R_D$ by output resistance of the active load. For PMOS current-mirror load on NMOS pair (both branches feed mirror; output single-ended):
$$
A_{dm}^\text{SE}=g_m\cdot(r_{o,N}\|r_{o,P})
$$
This is *non-inverting* and full-strength (no factor of ½) because the mirror folds the other branch into the output. For both-resistive-load with cascoded outputs:
$$
A_{dm}=g_m\cdot[r_{o,N}\cdot(g_m r_o)\,\|\,r_{o,P}\cdot(g_m r_o)]\approx \tfrac{1}{2}g_m^2 r_o^2
$$

### Worked Example — PYQ 2021-22 Part B Q1(b)
> *`PYQs/MuE 2022 Compre.pdf` p.3*

PMOS diff pair (input $M_{1A}, M_{1B}$), NMOS-mirror load ($M_{2A}, M_{2B}$), tail = 0.1 mA. $V_{ov}=0.25$ V, $V_A=100$ V, $V_T=0.65$ V, $V_{DD}=2.5$ V.

$$
I_D=0.05\text{ mA};\ g_m=\frac{2\cdot 0.05m}{0.25}=0.4\text{ mA/V}
$$
$$
r_o=\frac{V_A}{I_D}=\frac{100}{0.05m}=2\text{ M}\Omega
$$
$$
A_{dm}^\text{SE}=g_m(r_{o,P}\|r_{o,N})=0.4m\cdot 1\text{M}=\boxed{400\text{ V/V}}
$$

---

## Type 5D — Common-Mode Gain & CMRR

**Theory.** For pure CM input, $P$ moves with input. Each half = CS with source degeneration $2 R_\text{SS}$ (the tail's output resistance).

**Formulas.**
$$
A_{cm}^\text{SE}=-\frac{R_D}{2 R_\text{SS}+\tfrac{1}{g_m}}\approx -\frac{R_D}{2R_\text{SS}}
$$
$$
A_{cm}^\text{diff}=0\ \text{(matched)};\quad \text{CMRR}^\text{SE}=\left|\frac{A_{dm}}{A_{cm}}\right|=g_m\cdot 2R_\text{SS}
$$

### Worked Example — PYQ 2012-13 Part A Q3
> *`PYQs/Compre with sol.pdf` p.4*

Current-mirror-load diff amp with $g_m=1$ mA/V, $r_{o3}=100$ k, $r_{o5}=50$ k, $V_{out}=V_O$ at single end. $V_{i1}=V_{i2}=V_{id}$ (CM), output $V_\text{out}$.

$$
A_{od}=g_m(r_{o2}\|r_{o4})=1m\cdot(100\text{k}\|100\text{k})=\boxed{50\text{ V/V}}
$$
$$
A_{cm}\approx -\frac{1}{2 g_m r_{o5}}\cdot\frac{\Delta g_m}{g_m}\quad\text{(via mismatch)}
$$
With $\Delta g_m/g_m\sim$ small (use given $\pm 0.001$ mA/V mismatch):
$$
A_{cm}\approx \frac{0.001}{2\cdot 1m\cdot 50\text{k}}\approx 10^{-5}
$$
$$
\text{CMRR}=\frac{50}{10^{-5}}=5\times 10^6\ (\approx 134\text{ dB})
$$

---

## Type 5E — Input Common-Mode Range (ICMR)

**Theory.** Limits set by saturation of (i) input pair and (ii) tail current source.

**NMOS pair, NMOS tail, resistive load:**
$$
V_{ICM,\max} = V_{DD}-I_D R_D + V_{TN}
$$
$$
V_{ICM,\min} = V_{SS} + V_{ov,tail} + V_{TN} + V_{ov,M1}
$$

**PMOS pair, PMOS tail, NMOS-mirror load:**
$$
V_{ICM,\max} = V_{DD}-V_{ov,tail}-|V_{TP}|-V_{ov,M1}
$$
$$
V_{ICM,\min} = V_{D,M1}+|V_{TP}| = (V_{TN}+V_{ov,N})+|V_{TP}|
$$

### Worked Example — PYQ 2021-22 Part B Q1(b)(iii)
$V_{DD}=2.5$, $V_{ov,tail}=V_{ov,P}=V_{ov,N}=0.25$, $V_{TN}=V_{TP}=0.65$. PMOS pair, NMOS mirror.

$$
V_{ICM,\max}=2.5-0.25-0.65-0.25=\boxed{1.35\text{ V}}
$$
$$
V_{ICM,\min}=(V_{TN}+V_{ov,N})+|V_{TP}|=(0.65+0.25)+0.65=\boxed{1.55\text{ V}\,?}
$$
*Wait — for PMOS pair: $V_{ICM,\min}$ comes from output transistor (M1) saturation when $V_G$ pulls source low. Actually the simpler limit is for $M_1$: $V_{SD,M1}\ge V_{ov,P}$. With $V_{D,M1}=V_{TN}+V_{ov,N}=0.9$, and $V_{S,M1}=V_G+|V_{GS}|=V_G+0.9$, $V_{SD,M1}=V_G\ge V_{ov,P}=0.25$ V.*

$$
\boxed{V_{ICM,\min}=0.25\text{ V},\ V_{ICM,\max}=1.35\text{ V},\ \Delta V_{ICM}=1.10\text{ V}}
$$

---

## Type 5F — Output Voltage Swing

**Theory.** Limits: (high) output transistor cuts off / load saturates; (low) input pair leaves saturation or load mirror leaves saturation.

For NMOS pair with PMOS-mirror load (single-ended):
$$
V_{O,\max} = V_{DD}-V_{ov,P},\quad V_{O,\min}=V_{P}+V_{ov,N}=V_{ICM}-V_{TN}
$$

### Worked Example — PYQ 2021-22 Part B Q1(b)(ii, v)
PMOS pair, NMOS-mirror load, $V_{DD}=2.5$, $V_{ov}=0.25$:
$$
V_{O,\min}=V_{ov,N}=0.25\text{ V},\ V_{O,\max}=V_{DD}-V_{ov,tail}-V_{ov,P}=2.5-0.5=2.0\text{ V}
$$
DC output (set by NMOS diode side): $V_{O,DC}=V_{TN}+V_{ov,N}=0.9$ V.
Symmetric swing limited by lower bound: $0.9-0.25=0.65$ V.
$$
v_\text{in,max,unclipped}=\frac{V_\text{swing}}{|A_{dm}|}=\frac{0.65}{400}=\boxed{1.625\text{ mV peak}}
$$

---

## Type 5G — Input Offset Voltage

**Theory.** Offset = differential input that nulls the output. Mismatches accumulate from $\Delta V_T$, $\Delta(W/L)$, $\Delta R_D$.

**Formulas.**
$$
V_{OS} = \Delta V_T + \frac{V_{ov}}{2}\left[\frac{\Delta(W/L)}{(W/L)}-\frac{\Delta R_D}{R_D}\right]
$$
$$
V_{OS} = \Delta V_T + \frac{V_{ov}}{2}\cdot\frac{\Delta(\mu_n C_{ox})}{\mu_n C_{ox}}\quad\text{(if mismatch in } K_n')
$$
Worst case: add magnitudes of independent terms.

### Worked Example — PYQ 2021-22 Part B Q1(b)(vi)
$V_{ov}=0.25$ V, $\pm 5\%$ $(W/L)$ mismatch.
$$
V_{OS}=\frac{V_{ov}}{2}\cdot\frac{\Delta(W/L)}{W/L}=\frac{0.25}{2}\cdot 0.05=\boxed{6.25\text{ mV}}
$$

### Worked Example — PYQ 2017-18 Part B Q3(d)
> *`PYQs/2018 Compre QP.pdf` p.2*

$\pm 5\%$ in $\mu_n C_{ox}$ AND $\pm 0.01$ V in $V_{TN}$. Worst-case offset, $V_{ov}=$ given (computed from $W/L=5/1$, $I_D=25\,\mu$A):
$$
V_{ov}=\sqrt{\tfrac{2\cdot 25\mu}{140\mu\cdot 5}}=\sqrt{0.0714}=0.267\text{ V}
$$
$$
V_{OS,worst}=\Delta V_T+\frac{V_{ov}}{2}\cdot 0.05=0.01+\frac{0.267}{2}\cdot 0.05=0.01+0.0067=\boxed{16.7\text{ mV}}
$$

---

## Type 5H — Min Supply Voltage for Given ICMR

**Algorithm.** Sum up the headroom drops on the active path: $V_{DD,\min}=\sum V_{ov}+\sum V_T$ in series.

### Worked Example — PYQ 2017-18 Part B Q3(e)
Folded current-mirror diff amp, ICMR ≥ 0.5 V. NMOS pair, NMOS tail, NMOS mirror.

For NMOS input pair on top of tail: $V_{ICM,\min}=V_{ov,tail}+V_{TN}+V_{ov,M1}$.
For NMOS pair below load: $V_{ICM,\max}=V_{DD}-V_{ov,load}+V_{TN}$.
$$
V_{DD,\min}-V_{ICM,\max}+V_{ICM,\min}\ge \Delta V_{ICM}=0.5\text{ V}
$$
$$
V_{DD,\min}=\Delta V_{ICM}+V_{ov,load}-V_{TN}+V_{ov,tail}+V_{TN}+V_{ov,M1}
$$
$$
=0.5+3 V_{ov}=0.5+3\cdot 0.267=\boxed{\approx 1.30\text{ V}}
$$

---

## Type 5I — Folded / Cascode Diff Amp Variants

**Theory.** Folded cascode: input pair drains feed cascodes that fold the current into the opposite supply. Useful for low-voltage with rail-to-rail ICMR. Output Rout is $g_m r_o^2$.

For Part B questions, focus on:
- Identify input pair vs cascode vs mirror
- $A_{dm}=g_m\cdot R_\text{out}$ where $R_\text{out}=$ parallel of two cascode-multiplied $r_o$'s
- Dominant pole at output: $\omega_p=1/(R_\text{out}C_L)$

### Worked Example — PYQ 2017-18 Part B Q3(c)
$C_L=10$ pF, $R_\text{out}\approx g_m r_o^2/2$ for fully cascoded. With $g_m=2I_D/V_{ov}=0.187$ mA/V, $r_o=100/25\mu=4$ M$\Omega$:
$$
R_\text{out}\approx\frac{g_m r_o^2}{2}=\frac{0.187m\cdot(4M)^2}{2}=1.5\times 10^9\,\Omega
$$
$$
\omega_{-3dB}=\frac{1}{R_\text{out}C_L}=\frac{1}{1.5\text{G}\cdot 10p}=\boxed{67\text{ rad/s}}
$$

---

# TIER 6 — Frequency Response

> Part B Q3 — appears every year, ~17 marks. Always involves Miller and OCTC.

---

## Type 6A — High-Frequency MOSFET Model

**Capacitances.**
- $C_{gs}$: gate–source, dominant
- $C_{gd}$: gate–drain, smaller but Miller-multiplied at input
- $C_{db}$: drain–body, contributes to output node
- $C_{sb}$: source–body, often shorted by source–bulk tie

**Device cutoff frequency:**
$$
f_T = \frac{g_m}{2\pi(C_{gs}+C_{gd})}
$$

---

## Type 6B — Miller Approximation

**Theory.** A capacitor $C_F$ between input ($V_1$) and output ($V_2=-A_v V_1$, $A_v>0$) is split into:
- $C_\text{in,Miller}=C_F(1+A_v)$ at input
- $C_\text{out,Miller}=C_F(1+1/A_v)\approx C_F$ at output

**Total at input node of CS amp:**
$$
C_\text{in} = C_{gs}+C_{gd}(1+|A_v|)
$$

**Algorithm.** For each capacitor between two nodes, decide if it's a Miller cap (between input and output of inverting stage). Apply formula. Otherwise leave at original location.

---

## Type 6C — Open-Circuit Time Constants (OCTC)

**Theory.** Approximate dominant-pole 3-dB BW: $\omega_{-3dB}\approx 1/\tau_H$, where
$$
\tau_H = \sum_k C_k R_k
$$
$R_k$ = resistance seen by $C_k$ with **all other capacitors open**.

**Algorithm.**
1. List all caps in the circuit.
2. For each $C_k$: open all others, find $R$ between its terminals (Thevenin).
3. Sum $C_k R_k$. Reciprocal gives $\omega_{-3dB}$.

---

## Type 6D — Pole/Zero Identification

**Theory.** Each independent capacitor + its associated resistance gives a pole. Each path that bypasses the gain (e.g. $C_{gd}$ feeding signal forward) gives a zero.

**Common poles in CS amp:**
- Input (gate) pole: $\omega_{p,\text{in}}=1/[R_\text{sig}(C_{gs}+C_{gd}(1+|A_v|))]$
- Output (drain) pole: $\omega_{p,\text{out}}=1/[(R_D\|r_o)(C_L+C_{db}+C_{gd})]$

**Common zeros:**
- $C_{gd}$ feedforward: $\omega_z = +g_m/C_{gd}$ (RHP zero in CS amp — at frequency where $C_{gd}$ short-circuits gate to drain)

---

## Type 6E — Bode Plot Construction

**Rules.**
- Magnitude: starts at $20\log|A_0|$ (DC). Drops 20 dB/dec at each LHP pole; rises 20 dB/dec at each LHP zero. RHP zero behaves like a zero in magnitude but ADDS phase lag.
- Phase: $-45°$ at each pole frequency, $-90°$ asymptotically. $+45°$ at LHP zero, $+90°$ asymp. RHP zero contributes $-90°$ asymptotically.
- UGB (single-pole approximation): $\omega_\text{UGB} = |A_0|\cdot \omega_{p1}$.

---

## Type 6F — Phase Margin

**Definition.**
$$
\text{PM} = 180° + \angle H(j\omega_\text{UGB})
$$
For dominant-pole system with secondary pole at $\omega_{p2}$:
$$
\text{PM} = 90° - \arctan(\omega_\text{UGB}/\omega_{p2})
$$
- PM $\ge 60°$: well-damped, $\omega_{p2}\ge 2\omega_\text{UGB}$
- PM $= 45°$: $\omega_{p2}=\omega_\text{UGB}$
- PM $< 45°$: ringy / unstable

---

## Type 6G — RHP Zero Compensation (Miller Cc)

**Theory.** In two-stage Miller-compensated OpAmp, $C_c$ feedforward creates RHP zero at $\omega_z=+g_{m2}/C_c$. Since RHP zero adds phase lag, it ruins PM. Mitigate by adding $R_z\ge 1/g_{m2}$ in series with $C_c$ — pushes zero to LHP or infinity.

---

## Worked Example — PYQ 2022-23 Part B Q3
> *`PYQs/Compre Qp.pdf` p.3*

NMOS amp, $V_{ov}=0.2$ V, $\lambda=0.01$, $C_{gs}=10$ pF, $C_{gd}=1$ pF, $C_{db}=C_{sb}=2$ pF, $R_S\ll 1/g_m$. $I_D=0.5$ mA, $R_D=10$ k.

**Setup.**
$$
g_m=\frac{2\cdot 0.5m}{0.2}=5\text{ mA/V},\quad r_o=\frac{1}{0.01\cdot 0.5m}=200\text{ k}\Omega
$$
$$
A_v\approx -g_m(R_D\|r_o)=-5m\cdot(10\text{k}\|200\text{k})=-5m\cdot 9.52\text{k}=-47.6\text{ V/V}
$$

**(a–b) HF model + dominant poles.**

Two independent capacitive nodes ⇒ **2 poles, 1 zero (from $C_{gd}$ feedforward)**.

**(c) Pole frequencies.**
- Input node (gate of $M_1$) — $R_\text{in,eff}=R_S\ll 1/g_m\to$ small. Total $C_\text{in}=C_{gs}+C_{gd}(1+|A_v|)=10+1\cdot 48.6=58.6$ pF. With $R_S$ tiny, this pole is at very high frequency — non-dominant.
- Output node (drain) — $R_\text{out}=R_D\|r_o=9.52$ k$\Omega$. Total $C_\text{out}=C_L+C_{db}+C_{gd}=$ … if no $C_L$: $=C_{db}+C_{gd}=3$ pF.

$$
\omega_{p,\text{out}}=\frac{1}{9.52\text{k}\cdot 3\text{p}}=3.5\times 10^7\text{ rad/s}\approx \boxed{35\text{ Mrad/s}}
$$
(This is the dominant pole.)

Zero (RHP): $\omega_z=g_m/C_{gd}=5\text{m}/1\text{p}=\boxed{5\times 10^9\text{ rad/s}}$.

**(d) Phase at $\omega=10$ krad/s** (well below dominant pole):
$$
\phi\approx -180°-\arctan(10\text{k}/35\text{M})\approx -180°
$$
(The $-180°$ is from the inverting nature; pole hasn't kicked in yet.)

**(e) Bode magnitude.**
- DC gain: $|A_v|=47.6$ → $33.5$ dB
- Drops at $\omega_{p,\text{out}}=35$ Mrad/s
- UGB ≈ $|A_v|\cdot \omega_{p,\text{out}}=47.6\cdot 35\text{M}=\boxed{1.67\text{ Grad/s}}$

**(f) UGB:** $\boxed{1.67\text{ Grad/s}}$ (or $266$ MHz).

---

## Worked Example — PYQ 2021-22 Part B Q2(a)
> *`PYQs/MuE 2022 Compre.pdf` p.4*

Find GM, PM, 3-dB BW for:
$$
\frac{Y(s)}{X(s)} = \frac{20(s+20)}{(s+2)(s+10)}
$$

**DC gain.**
$$
H(0)=\frac{20\cdot 20}{2\cdot 10}=20\to 26\text{ dB}
$$

**Poles & zero.** $\omega_{p1}=2$, $\omega_{p2}=10$, $\omega_z=20$ rad/s.

**3-dB BW** (dominant pole): $\boxed{\omega_{-3dB}=2\text{ rad/s}}$.

**UGB.** Magnitude:
- $26$ dB until $\omega=2$
- Drops at $-20$ dB/dec until $\omega=10$ → $26-14=12$ dB
- Drops at $-40$ dB/dec until $\omega=20$ → $12-12=0$ dB

So $\boxed{\omega_\text{UGB}=20\text{ rad/s}}$.

**Phase at $\omega=20$:**
$$
\phi=-\arctan(10)-\arctan(2)+\arctan(1)=-84.3°-63.4°+45°=-102.7°
$$
$$
\text{PM}=180°-102.7°=\boxed{77.3°}
$$

**GM.** Phase asymptote: $-90°-90°+90°=-90°$, never reaches $-180°$. So $\boxed{\text{GM}=\infty}$.

---

## Worked Example — PYQ 2021-22 Part B Q2(b)
> *`PYQs/MuE 2022 Compre.pdf` p.4*

Two-stage amp, $R_\text{sig}=100$ k$\Omega$ at node $P$. Per device: $g_m=0.4$ mA/V (NMOS), $0.2$ mA/V (PMOS); $r_o=600$ k$\Omega$ (NMOS), $400$ k$\Omega$ (PMOS); $C_{gs}=120$ fF (NMOS), $100$ fF (PMOS); $C_{gd}=60$, $50$ fF; $C_{sb}=C_{db}=20$ fF each.

**Stage gains.**
$$
A_{v1}=-g_{m,M1}(r_{o,M1}\|r_{o,M2})=-0.2m\cdot(400\text{k}\|600\text{k})=-0.2m\cdot 240\text{k}=-48
$$
$$
A_{v2}=-g_{m,M3}\cdot R_{out,2}=-48 \text{ (same, by symmetry)}
$$

**(i) Capacitance at $P$ (input of stage 1).**
$$
C_P=C_{gs,M1}+C_{gd,M1}(1+|A_{v1}|)=100+50(1+48)=100+2450=\boxed{2550\text{ fF}=2.55\text{ pF}}
$$

**Capacitance at $X$ (output stage 1, input stage 2).**
$$
C_X = C_{gd,M1}\!\left(1+\tfrac{1}{|A_{v1}|}\right)+C_{db,M1}+C_{db,M2}+C_{gs,M3}+C_{gd,M3}(1+|A_{v2}|)
$$
$$
\approx 50+20+20+100+2450=\boxed{2640\text{ fF}=2.64\text{ pF}}
$$

**(ii) OCTC for stage 1 BW.**
$$
\tau_H = R_\text{sig}\cdot C_P + R_{out,1}\cdot C_X
$$
$$
= 100\text{k}\cdot 2.55\text{p}+240\text{k}\cdot 2.64\text{p}=255\text{ ns}+634\text{ ns}=889\text{ ns}
$$
$$
\omega_{-3dB,1}=\frac{1}{\tau_H}=1.13\text{ Mrad/s}\Rightarrow \boxed{f_{-3dB,1}=179\text{ kHz}}
$$

**(iii) $C_L$ for pole at $Q$ at $500$ krad/s.** $R_{out,2}=240$ k$\Omega$.
$$
C_L=\frac{1}{\omega R_{out,2}}=\frac{1}{500\text{k}\cdot 240\text{k}}=\frac{1}{1.2\times 10^{11}}=\boxed{8.33\text{ pF}}
$$

---

## Worked Example — PYQ 2021-22 Part B Q2(c) — CMRR Bandwidth

Diff pair with tail $R_{SS}=100$ k$\Omega$, $C_{SS}=0.1$ pF. $A_{dm}$ has dominant pole at $40$ Mrad/s. Find $-3$ dB of CMRR.

**Theory.** $A_{cm}\propto (1+sR_{SS}C_{SS})/(\text{const})$ — has a zero at $1/(R_{SS}C_{SS})$. CMRR $= A_{dm}/A_{cm}$ has effective poles where $A_{dm}$ rolls off OR $A_{cm}$ rises.

$$
\omega_{z,A_{cm}}=\frac{1}{R_{SS}C_{SS}}=\frac{1}{100\text{k}\cdot 0.1\text{p}}=10^8\text{ rad/s}=100\text{ Mrad/s}
$$

CMRR rolls off at the lower of (a) $A_{dm}$ pole = 40 Mrad/s, (b) $A_{cm}$ zero = 100 Mrad/s.
$$
\boxed{\omega_{-3dB,\text{CMRR}}=40\text{ Mrad/s}}
$$

---

## Worked Example — PYQ 2017-18 Part B Q4(b) — Two-Stage Diff Amp Bandwidth

> *`PYQs/2018 Compre QP.pdf` p.3 (Fig 4e/f)*

NMOS pair (M1, M2), NMOS-mirror load (M3, M4 fed by M5, M6 cascode). $I_{SS}=100\,\mu$A, $V_{ov}=0.2$ V, $C_L=5$ pF, $C_\pi=C_{gs}=1$ pF, $C_T=C_{gd}=100C_{gd}=100$ fF (need to interpret).

**$g_m$, $r_o$:**
$$
I_D=50\,\mu\text{A},\ g_m=\frac{2\cdot 50\mu}{0.2}=0.5\text{ mA/V},\ r_o=\frac{100}{50\mu}=2\text{ M}\Omega
$$

**Output pole** (dominant, at output node with $C_L$):
$$
R_\text{out}=r_{o,M2}\|r_{o,M4}=1\text{ M}\Omega
$$
$$
\omega_{p,out}=\frac{1}{R_\text{out}\cdot C_L}=\frac{1}{1\text{M}\cdot 5\text{p}}=\boxed{200\text{ krad/s}}
$$

**Mirror pole** (at diode-connected M3 node, low impedance):
$$
\omega_{p,mirror}=\frac{1}{(1/g_m)\cdot C_x}=\frac{g_m}{C_x}=\frac{0.5m}{2C_{gs}}\quad(\text{both M3 and M4 gates load the node})
$$
$$
=\frac{0.5m}{2\text{p}}=\boxed{250\text{ Mrad/s}}
$$

**DC gain.**
$$
A_{dm}=g_m\cdot R_\text{out}=0.5m\cdot 1\text{M}=500\Rightarrow 54\text{ dB}
$$

**UGB.**
$$
\omega_\text{UGB}=A_{dm}\cdot\omega_{p,out}=500\cdot 200\text{k}=\boxed{100\text{ Mrad/s}}
$$

**Phase margin** (mirror pole as secondary):
$$
\text{PM}=90°-\arctan(\omega_\text{UGB}/\omega_{p,mirror})=90°-\arctan(100\text{M}/250\text{M})=90°-21.8°=\boxed{68.2°}
$$

---

# Quick-Reference Cheat Sheet

```
SAT:   ID = (μCox/2)(W/L)Vov²(1+λVDS)         TRIODE: when |VDS|<|Vov|
       gm = 2ID/Vov,   ro = VA/ID,   gm·ro = 2VA/Vov

CS:        Av = -gm(RD∥ro∥RL)            Rin = ∞ or RF/(1+|Av|)
CS-deg:    Av = -gmRD/(1+gmRs)           Rout↑ = RD∥(gm·ro·Rs)
CD:        Av ≈ 1                         Rout ≈ 1/gm
CG:        Av = +gm(RD∥ro)                Rin = 1/gm
Cascode:   Rout = gm·ro²                  Av = -gm·Rout

DIFF AMP (resistive load):
  Adm,SE = -gm·RD/2          Adm,diff = -gm·RD
  Acm,SE = -RD/(2Rss)        CMRR = 2gm·Rss
DIFF AMP (mirror load, single-ended):
  Adm = gm·(roN∥roP)         Vos = ΔVT + (Vov/2)·Δ(W/L)/(W/L)
ICMR (NMOS pair):
  V,max = VDD - ID·RD + VTN
  V,min = Vov,tail + VTN + Vov,M1

FREQ RESPONSE:
  Miller: Cin = Cgs + Cgd(1+|Av|)
  OCTC:   ωH = 1/Σ(Ck·Rk),  Rk = R seen by Ck w/ others open
  fT  = gm/[2π(Cgs+Cgd)]
  RHP zero (Miller Cc): ωz = +gm/Cc
  PM = 90° - arctan(ωUGB/ωp2)

CURRENT MIRROR:
  Basic:  IO/IREF = (W/L)O/(W/L)REF
  LV cascode:  VC = VT+2Vov,  VP,min = 2Vov
  BJT n outputs: IO/IREF = 1/(1+(n+1)/β)
```

---

# PYQ Cross-Reference

| PYQ | File | Part | Q# | Type | Tier |
|-----|------|------|----|------|------|
| 2012-13 | Compre with sol | A | Q1B | Mirror design | 4A |
| 2012-13 | " | A | Q3 | CMRR mismatch | 5D |
| 2012-13 | " | B | Q1 | Diff amp + freq | 5,6 |
| 2015-16 | 2016 mue compre | A | Q1 | CG cascode | 3D |
| 2015-16 | " | A | Q2 | Intrinsic gain | 1D |
| 2015-16 | " | A | Q3 | CS active load | 3F |
| 2015-16 | " | A | Q4 | Body effect | 1C |
| 2015-16 | " | A | Q5 | CLM, ro | 1B |
| 2015-16 | " | A | Q6 | W/L sizing | 1E |
| 2015-16 | " | A | Q7 | CS gain | 3A |
| 2015-16 | " | A | Q8 | Cascode Rout | 3E |
| 2015-16 | " | A | Q9 | CS active load | 3F |
| 2015-16 | " | A | Q10 | Source follower | 3C |
| 2015-16 | " | B | Q1 | PMOS cascode load | 3E |
| 2015-16 | " | B | Q2 | Mirror sizing | 4A |
| 2015-16 | " | B | Q4 | 2-stage OpAmp | 5,6,7 |
| 2017-18 | 2018 Compre QP | B | Q1 | Feedback | 7 |
| 2017-18 | " | B | Q2a | Mirror ratio | 4A |
| 2017-18 | " | B | Q2b | LV cascode | 4B |
| 2017-18 | " | B | Q2c | BJT cascode bias | 4C |
| 2017-18 | " | B | Q3 | Folded diff amp | 5 |
| 2017-18 | " | B | Q4 | Diff pair freq | 5,6 |
| 2021-22 | MuE 2022 Compre | A | Q3 | CSA + degen | 3B |
| 2021-22 | " | B | Q1a | Feedback (10) | 7 |
| 2021-22 | " | B | Q1b | PMOS diff amp | 5 |
| 2021-22 | " | B | Q2a | Bode from H(s) | 6 |
| 2021-22 | " | B | Q2b | Miller + OCTC | 6 |
| 2021-22 | " | B | Q2c | CMRR -3dB | 6 |
| 2022-23 | Compre Qp | A | Q1 | Self-bias CS | 2A,3A |
| 2022-23 | " | A | Q2A | Mirror sizing | 4A |
| 2022-23 | " | A | Q2B | PMOS region | 1A |
| 2022-23 | " | B | Q1 | Diff amp + DM/CM | 5A,5B |
| 2022-23 | " | B | Q2A | Bias divider, load line | 2B,2C |
| 2022-23 | " | B | Q2B | BJT cascode | 3G |
| 2022-23 | " | B | Q3 | Cascode freq response | 6 |
| 2022-23 | " | B | Q4 | Feedback amplifier | 7 |

---

# TIER 7 — Feedback Amplifiers

> Part B Q4 — every year, ~15 marks. Topology ID + closed-loop calcs + bandwidth.

---

## Master Concepts

**Four topologies** (input mixing × output sampling):

| Topology | Input | Output | Amp Type | Forward $A$ | $\beta$ units |
|---|---|---|---|---|---|
| Series–Shunt | $V$ subtracted | $V$ sensed | Voltage ($V/V$) | $A_v$ | dim'less |
| Shunt–Shunt | $I$ subtracted | $V$ sensed | Transresistance ($V/A$) | $R_m$ | $1/\Omega$ |
| Series–Series | $V$ subtracted | $I$ sensed | Transconductance ($A/V$) | $G_m$ | $\Omega$ |
| Shunt–Series | $I$ subtracted | $I$ sensed | Current ($A/A$) | $A_i$ | dim'less |

**ID rules.** Trace the $\beta$ network terminals:
- Sampling: parallel to load → **shunt** (V); in series with load → **series** (I)
- Mixing: at source/emitter → **series**; at gate/base node → **shunt**

**Closed-loop equations.**
$$
T = A\beta,\quad A_f = \frac{A}{1+T}
$$
$$
R_{\text{in},f} = R_\text{in}(1+T)\ [\text{series}],\quad \frac{R_\text{in}}{1+T}\ [\text{shunt}]
$$
$$
R_{\text{out},f} = \frac{R_\text{out}}{1+T}\ [\text{shunt sample}],\quad R_\text{out}(1+T)\ [\text{series sample}]
$$
$$
\omega_{Hf} = \omega_H(1+T),\quad A_f\cdot\omega_{Hf} = A\cdot\omega_H \text{ (GBW preserved)}
$$

---

## Type 7A — Topology Identification

**Algorithm.**
1. Locate the $\beta$ network (the resistive feedback path).
2. Output side: parallel to $R_L$ → shunt; in series with $R_L$ → series.
3. Input side: at gate/base node → shunt; at source/emitter (in input loop) → series.

**Quick patterns.**
- Single $R_F$ from drain → gate: shunt-shunt
- Divider $R_1, R_2$ from output → source: series-shunt
- $R_E$ in source/emitter, output current sensed: series-series
- Current sample fed back to gate node: shunt-series

---

## Type 7B — $\beta$ Calculation

| Topology | Network | $\beta$ |
|---|---|---|
| Series–Shunt | $V$-divider $R_1$ (out→S), $R_2$ (S→GND) | $R_2/(R_1+R_2)$ |
| Shunt–Shunt | Single $R_F$ (out→in) | $-1/R_F$ |
| Series–Series | $R_E$ in source/emitter | $R_E$ |
| Shunt–Series | Current divider $R_1, R_2$ | $R_2/(R_1+R_2)$ |

---

## Type 7C — Open-Loop Gain $A$ (with $\beta$ Loading)

**Theory.** Compute $A$ with $\beta$ network as resistive load (feedback path nulled). $\beta$-network resistors load both input and output of the open-loop amp.
- Series-shunt: $R_2$ acts as source-degeneration at input, $(R_1+R_2)$ in parallel at output.
- Shunt-shunt: $R_F$ in parallel at both ports.

---

## Type 7D — Resistance with Feedback

Modification rules (above). **Series mixing raises $R_\text{in}$**, shunt lowers it. **Shunt sampling lowers $R_\text{out}$**, series raises it.

---

## Type 7E — Bandwidth, UGB, GBW Preservation

**Theory.** Single dominant-pole approximation: feedback trades gain for bandwidth, GBW unchanged.

$$
\omega_{Hf} = \omega_H(1+T), \quad A_f\,\omega_{Hf} = A\,\omega_H = \text{const}
$$

**Gain crossover** $\omega_x$ (loop): where $|A(j\omega)\beta|=1$.

---

### Worked Example — PYQ 2021-22 Part B Q1(a)
> *`PYQs/MuE 2022 Compre.pdf` p.3*

Open-loop: $A=100$ dB, $R_\text{in}=100$ kΩ, $R_\text{out}=0.1$ kΩ, $\omega_H=1$ k rad/s. Shunt-series feedback, $A_f=40$ dB.

$$
A=10^5,\ A_f=10^2,\quad 1+T = A/A_f = 10^3 \Rightarrow T\approx 1000
$$
$$
\beta = T/A = 10^{-2}\,(\text{A/A})
$$

Shunt input → $R_\text{in}$ shrinks: $R_{\text{in},f}=\boxed{100\,\Omega}$
Series output → $R_\text{out}$ grows: $R_{\text{out},f}=\boxed{100\text{ k}\Omega}$
BW: $\omega_{Hf}=\omega_H(1+T)=\boxed{1\text{ Mrad/s}}$
GBW check: $10^5\cdot 10^3=10^8=10^2\cdot 10^6$ ✓

---

### Worked Example — PYQ 2017-18 Part B Q1
> *`PYQs/2018 Compre QP.pdf` p.1*

3-stage amp with series-shunt feedback. $g_{mn}=2$ mA/V, $g_{mp}=1$ mA/V, $r_o=100$ kΩ. $R_1=150$ kΩ, $R_S=10$ kΩ.

**(a) Topology.** Series-shunt voltage feedback ($\beta$ samples $V_\text{out}$, mixes as $V$ at source via divider).

**(b) $\beta$:**
$$
\beta = \frac{R_S}{R_S+R_1} = \frac{10}{160} = 0.0625
$$

**(c) Open-loop $A$.** Three CS stages with active loads, total $A_o\approx 1000$ V/V (illustrative; exact value depends on circuit details).
$$
T = A\beta = 1000\cdot 0.0625 = 62.5
$$

**(d) Closed-loop:**
$$
A_f = \frac{1000}{63.5}=\boxed{15.75\text{ V/V}}
$$

**(e) Overall $V_o/V_\text{in}$.** With $R_S$ source and $R_{\text{in},f}\to\infty$ (gate-input, series mixing):
$$
\frac{V_o}{V_\text{in}}\approx A_f \approx \boxed{15.75\text{ V/V}\ (24\text{ dB})}
$$

---

### Worked Example — PYQ 2022-23 Part B Q4
> *`PYQs/Compre Qp.pdf` p.3*

NMOS amp, $R_S=1$ kΩ at source of input transistor, $R_F=1$ MΩ output→source feedback, $R_p=1$ kΩ output load, $C_L=1$ nF, $I_5=25\,\mu$A, $I_\text{SS}=200\,\mu$A, $V_{ov}=0.2$ V, $\lambda=0.01$ V$^{-1}$.

**(a) Topology.** Series-shunt voltage feedback.
$$
\beta = \frac{R_S}{R_S+R_F}\approx \frac{1\text{k}}{1\text{M}}=10^{-3}
$$

**(b) Open-loop $A_o$ and $R_\text{out}$.** Two-stage CS:
- Stage 1 ($I_D=I_\text{SS}/2=100\,\mu$A): $g_m=1$ mA/V, $r_o=1$ MΩ. $A_1=-g_m(r_{on}\|r_{op})=-500$.
- Stage 2 ($I_D=I_5=25\,\mu$A): $g_m=0.25$ mA/V, $r_o=4$ MΩ. With $R_p=1$ kΩ at output: $A_2=-g_m(r_o\|R_p)\approx -0.25$.
$$
A_o = A_1\cdot A_2 = (-500)(-0.25) = \boxed{+125\text{ V/V}}
$$
$$
R_\text{out}\approx R_p \| r_{o,2} \approx R_p = 1\text{ k}\Omega
$$

**(c) Closed-loop:**
$$
T = A_o\beta = 125\cdot 10^{-3}=0.125,\quad A_{of}=\frac{125}{1.125}=\boxed{111\text{ V/V}}
$$
$R_{\text{in},f}=R_\text{in}(1+T)\to\infty$ (gate input).
$R_{\text{out},f}=R_\text{out}/(1+T)=1\text{k}/1.125=\boxed{889\,\Omega}$.

**(d) BW.** Output pole $R_p\cdot C_L=1\,\mu$s → $\omega_H=1$ Mrad/s.
$$
\omega_{Hf}=\omega_H(1+T)=1.125\text{ Mrad/s}
$$
$$
\text{UGB}_\text{open}=A_o\omega_H=125\text{ Mrad/s}=\text{UGB}_\text{closed}=A_{of}\omega_{Hf}\,\checkmark
$$

**Gain crossover** $G_x$: where $|A(j\omega)|=1/\beta=1000$. Since $|A_o|=125 < 1000$, the loop gain magnitude never crosses unity → minimal feedback action; this $\beta$ is too small for the available gain.

---

# Mixed Problems — Multi-Concept Integration

> Three exam-style problems combining device, biasing, single-stage, mirror, diff amp, frequency response, and feedback. Use them as final integration practice.

---

## Mixed Problem 1 — Cascoded CS Amplifier
*(Tiers 1, 2, 3, 6)*

**Setup.** NMOS cascode CS amp. $M_1$ CS input (source grounded), $M_2$ CG cascode ($V_{b,M2}=1.5$ V at gate), PMOS active-load $M_3$ ($V_{b,M3}$ TBD). $W/L=20$ for all. Defaults ($\mu_n C_{ox}=140\mu$, $\mu_p C_{ox}=40\mu$, $V_{TN}=0.7$, $V_{TP}=-0.7$, $\lambda=0.1$). $V_{DD}=3.3$ V. Capacitances per device: $C_{gs}=1$ pF, $C_{gd}=0.1$ pF, $C_{db}=50$ fF. Output load $C_L=1$ pF. Source resistance $R_\text{sig}=20$ kΩ. Bias yields $V_{ov,N}=0.25$ V on $M_1, M_2$.

**(a) DC bias.**
$$
I_D = \tfrac{1}{2}\mu_n C_{ox}\tfrac{W}{L}V_{ov,N}^2 = \tfrac{1}{2}(140\mu)(20)(0.25)^2 = \boxed{87.5\,\mu\text{A}}
$$
$$
V_{ov,P}=\sqrt{\frac{2 I_D}{\mu_p C_{ox}(W/L)}}=\sqrt{\frac{175\mu}{800\mu}}=\boxed{0.468\text{ V}}
$$
$$
V_{GS,M1}=V_{TN}+V_{ov,N}=0.95\text{ V}=V_\text{in,DC}
$$
$$
V_{D,M1}=V_{b,M2}-V_{GS,M2}=1.5-0.95=\boxed{0.55\text{ V}}\quad(V_{DS,M1}>V_{ov}\,\checkmark)
$$
For $V_\text{out,DC}=V_{DD}/2=1.65$ V: $V_{b,M3}=V_{DD}-V_{SG,M3}=3.3-1.168=\boxed{2.13\text{ V}}$.

**(b) Small-signal.**
$$
g_{m,N}=\frac{2 I_D}{V_{ov,N}}=\frac{175\mu}{0.25}=\boxed{0.7\text{ mA/V}};\quad g_{m,P}=\frac{175\mu}{0.468}=\boxed{0.374\text{ mA/V}}
$$
$$
r_o=\frac{1}{\lambda I_D}=\frac{1}{0.1\cdot 87.5\mu}=\boxed{114\text{ k}\Omega}\ \text{(each)}
$$
Intrinsic gain: $g_m r_o \approx 80$.

**(c) Cascode output resistance** (looking into drain of $M_2$):
$$
R_\text{out,cas}=r_{o,M1}(1+g_{m,M2}r_{o,M2})+r_{o,M2}=114\text{k}\cdot 81+114\text{k}=\boxed{9.35\text{ M}\Omega}
$$

**(d) Total $R_\text{out}$.** Parallel with PMOS load $r_{o,M3}=114$ kΩ:
$$
R_\text{out}=R_\text{out,cas}\,\|\,r_{o,M3}\approx \boxed{113\text{ k}\Omega}
$$
(Dominated by $r_{o,M3}$ — the un-cascoded PMOS is the bottleneck.)

**(e) Voltage gain.**
$$
A_v=-g_{m,M1}\cdot R_\text{out}=-0.7\text{m}\cdot 113\text{k}=\boxed{-79\text{ V/V}\,(38\text{ dB})}
$$

**(f) Frequency response.**

*Output pole.* $C_\text{out}\approx C_L+C_{db,M2}+C_{db,M3}+C_{gd,M2}+C_{gd,M3}=1\text{p}+50\text{f}+50\text{f}+0.1\text{p}+0.1\text{p}=1.3$ pF (no Miller — gates of $M_2, M_3$ are AC ground).
$$
\omega_{p,\text{out}}=\frac{1}{R_\text{out}\cdot C_\text{out}}=\frac{1}{113\text{k}\cdot 1.3\text{p}}=\boxed{6.8\text{ Mrad/s}}
$$

*Input pole.* Local gain from gate of $M_1$ to drain of $M_1$ is $g_{m,M1}\cdot(1/g_{m,M2})\approx 1.87$ (Miller multiplier on $C_{gd,M1}\approx 2.87$).
$$
C_\text{in}=C_{gs,M1}+C_{gd,M1}\cdot 2.87=1\text{p}+0.287\text{p}=1.29\text{ pF}
$$
$$
\omega_{p,\text{in}}=\frac{1}{R_\text{sig}\cdot C_\text{in}}=\frac{1}{20\text{k}\cdot 1.29\text{p}}=\boxed{38.8\text{ Mrad/s}}
$$

**Dominant pole = output pole at 6.8 Mrad/s.** Cascoding has effectively suppressed Miller multiplication at the input.

**(g) UGB and PM** ($\omega_{p2}=38.8$ Mrad/s as secondary):
Naive: $\omega_\text{UGB}=|A_v|\omega_{p,\text{out}}=79\cdot 6.8\text{M}=537$ Mrad/s. Above $\omega_{p2}$ → use 2-pole formula:
$$
\omega_\text{UGB}^\text{actual}=\sqrt{|A_v|\,\omega_{p1}\,\omega_{p2}}=\sqrt{79\cdot 6.8\text{M}\cdot 38.8\text{M}}\approx \boxed{144\text{ Mrad/s}}
$$
$$
\text{PM}=180°-\arctan\!\frac{144}{6.8}-\arctan\!\frac{144}{38.8}=180°-87.3°-74.9°=\boxed{17.8°}
$$
Insufficient PM — would need Miller compensation if used in feedback.

---

## Mixed Problem 2 — NMOS Diff Amp with PMOS Mirror Load
*(Tiers 1, 4, 5, 6)*

**Setup.** NMOS pair $M_1, M_2$ ($W/L=10$) with NMOS tail $M_5$ ($I_\text{SS}=200\,\mu$A). PMOS mirror load: $M_3$ diode-connected (left), $M_4$ output (right), $W/L=10$ each. Defaults. $C_{gs}=2$ pF, $C_{gd}=0.4$ pF (per device). $C_L=4$ pF at output.

**(a) Bias and small-signal.**
$$
I_D = I_\text{SS}/2=\boxed{100\,\mu\text{A}}
$$
$$
V_{ov,N}=\sqrt{\frac{2\cdot 100\mu}{140\mu\cdot 10}}=\boxed{0.378\text{ V}};\quad V_{ov,P}=\sqrt{\frac{2\cdot 100\mu}{40\mu\cdot 10}}=\boxed{0.707\text{ V}}
$$
$$
g_{m,N}=\frac{200\mu}{0.378}=0.529\text{ mA/V};\quad g_{m,P}=\frac{200\mu}{0.707}=0.283\text{ mA/V}
$$
$$
r_o=\frac{1}{0.1\cdot 100\mu}=100\text{ k}\Omega \text{ (each)}
$$

**(b) Differential gain (single-ended out).**
$$
A_{dm}=g_{m,N}\cdot(r_{o,M2}\|r_{o,M4})=0.529\text{m}\cdot 50\text{k}=\boxed{26.5\text{ V/V}\,(28.5\text{ dB})}
$$

**(c) ICMR** (assume $V_{ov,M5}=V_{ov,N}=0.378$ V, NMOS tail).
$V_{D,M1}=V_{DD}-V_{SG,M3}=3.3-(0.7+0.707)=1.893$ V (set by mirror diode).
$$
V_{ICM,\max}=V_{D,M1}-V_{ov,N}+V_{GS,M1}=1.893-0.378+1.078=\boxed{2.59\text{ V}}
$$
$$
V_{ICM,\min}=V_{ov,M5}+V_{GS,M1}=0.378+1.078=\boxed{1.46\text{ V}}
$$
ICMR range $\approx 1.13$ V.

**(d) Output swing.**
$$
V_{out,\max}=V_{DD}-V_{ov,P}=\boxed{2.59\text{ V}}
$$
$$
V_{out,\min}=V_{S,M2}+V_{ov,N}=V_{ov,M5}+V_{ov,N}=\boxed{0.756\text{ V}}
$$
Range: $\approx 1.83$ V.

**(e) Frequency response.**

*Output pole.* $R_\text{out}=r_{on}\|r_{op}=50$ kΩ. $C_\text{out}\approx C_L+C_{gd,M2}+C_{gd,M4}=4\text{p}+0.4\text{p}+0.4\text{p}=4.8$ pF.
$$
\omega_{p,\text{out}}=\frac{1}{50\text{k}\cdot 4.8\text{p}}=\boxed{4.17\text{ Mrad/s}}
$$

*Mirror pole* (at gate of $M_3$, low-impedance node).
$R_X=1/g_{m,M3}=3.53$ kΩ. $C_X\approx C_{gs,M3}+C_{gs,M4}=4$ pF.
$$
\omega_{p,X}=\frac{1}{3.53\text{k}\cdot 4\text{p}}=\boxed{70.8\text{ Mrad/s}}
$$
Non-dominant by ~17×.

*CMRR pole.* Tail $R_\text{SS}=r_{o,M5}=100$ kΩ; tail capacitance $C_\text{SS}\approx C_{db,M5}+C_{sb,M1}+C_{sb,M2}\approx 0.5$ pF.
$$
\omega_{z,A_{cm}}=\frac{1}{R_\text{SS} C_\text{SS}}=\frac{1}{100\text{k}\cdot 0.5\text{p}}=20\text{ Mrad/s}
$$
CMRR $-3$ dB at $\min(\omega_{p,\text{out}},\omega_{z,A_{cm}})=\boxed{4.17\text{ Mrad/s}}$.

**(f) Worst-case offset** ($\pm 5\%$ in $W/L$, $\pm 0.02$ V in $V_T$):
$$
V_{OS,\text{worst}}=|\Delta V_T|+\frac{V_{ov,N}}{2}\cdot\frac{|\Delta(W/L)|}{(W/L)}=0.02+\frac{0.378}{2}\cdot 0.05=\boxed{29.4\text{ mV}}
$$

---

## Mixed Problem 3 — Two-Stage Amp with Series-Shunt Feedback
*(Tiers 3, 6, 7)*

**Setup.** Two-stage CMOS amp:
- Stage 1: NMOS CS ($M_1$) with PMOS mirror load. $g_{m,M1}=0.5$ mA/V, $r_{o,M1}=200$ kΩ, $r_{o,\text{load1}}=200$ kΩ. $C_{gd,M2}=0.5$ pF (between stage outputs and stage 2 input).
- Stage 2: NMOS CS ($M_2$) with PMOS mirror load. $g_{m,M2}=0.5$ mA/V, $r_{o,M2}=200$ kΩ, $r_{o,\text{load2}}=200$ kΩ.

Output node has $C_L=10$ pF. Internal node $X$ (between stages) has parasitic $C_X=1$ pF.

Series-shunt feedback: $R_1=99$ kΩ (output → source of $M_1$), $R_2=1$ kΩ (source of $M_1$ → GND).

**(a) Open-loop gain.**
$$
A_1=-g_{m,M1}(r_{o,M1}\|r_{o,\text{load1}})=-0.5\text{m}\cdot 100\text{k}=-50
$$
$$
A_2=-g_{m,M2}(r_{o,M2}\|r_{o,\text{load2}})=-0.5\text{m}\cdot 100\text{k}=-50
$$
$$
A_o = A_1\cdot A_2 = (-50)(-50)=\boxed{+2500\text{ V/V}\,(68\text{ dB})}
$$

**(b) $R_\text{out}$ open-loop.**
$$
R_\text{out}=r_{o,M2}\|r_{o,\text{load2}}=\boxed{100\text{ k}\Omega}
$$

**(c) $\beta$ (V-divider, series-shunt).**
$$
\beta = \frac{R_2}{R_1+R_2}=\frac{1}{100}=\boxed{0.01}
$$

**(d) Loop gain & closed-loop.**
$$
T=A_o\beta=2500\cdot 0.01=\boxed{25}
$$
$$
A_f=\frac{A_o}{1+T}=\frac{2500}{26}=\boxed{96.2\text{ V/V}}\quad(\to 1/\beta=100\text{ as }T\to\infty)
$$

**(e) Closed-loop resistances.**
- Series mixing: $R_{\text{in},f}=R_\text{in}(1+T)=\infty$ (gate-input).
- Shunt sampling: $R_{\text{out},f}=R_\text{out}/(1+T)=100\text{k}/26=\boxed{3.85\text{ k}\Omega}$.

**(f) Open-loop dominant pole.** At node $X$: $R_X=100$ kΩ, $C_X^\text{tot}=C_X+C_{gd,M2}(1+|A_2|)=1\text{p}+0.5\text{p}\cdot 51=26.5$ pF.
$$
\omega_{p,X}=\frac{1}{100\text{k}\cdot 26.5\text{p}}=\boxed{377\text{ krad/s}}
$$
Output pole: $R_\text{out}\cdot C_L=1\,\mu$s → $\omega_{p,\text{out}}=1$ Mrad/s. Secondary.

**(g) Closed-loop BW & GBW check.**
$$
\omega_{Hf}=\omega_H(1+T)=377\text{k}\cdot 26=\boxed{9.80\text{ Mrad/s}}
$$
GBW: $A_o\omega_H=2500\cdot 377\text{k}=9.43\times 10^8$; $A_f\omega_{Hf}=96.2\cdot 9.80\text{M}=9.43\times 10^8$ ✓

**(h) Phase margin.** Loop unity crossover with $\omega_{p1}=377$ k, $\omega_{p2}=1$ M:
$$
\omega_x \approx \sqrt{T_o\,\omega_{p1}\,\omega_{p2}}=\sqrt{25\cdot 377\text{k}\cdot 1\text{M}}=3.07\text{ Mrad/s}
$$
$$
\text{PM}=180°-\arctan\!\frac{3.07\text{M}}{377\text{k}}-\arctan\!\frac{3.07\text{M}}{1\text{M}}=180°-83°-72°=\boxed{25°}
$$
PM < 45° — needs Miller compensation cap $C_c$ across $M_2$ (pole-splitting) to push the dominant pole lower and raise PM toward $\geq 60°$.

---

> **Syllabus coverage check.** Mixed Problem 1 hits MOSFET DC bias, intrinsic gain, cascode topology, Miller approximation, OCTC, and PM. Mixed Problem 2 covers diff pair DC/AC, ICMR, swing, mirror pole, CMRR pole, and offset. Mixed Problem 3 spans 2-stage gain, $\beta$ network, loop gain, $R_{\text{in},f}/R_{\text{out},f}$, BW extension, GBW conservation, and PM. Together they exercise every Tier 1–7 concept on the syllabus.
