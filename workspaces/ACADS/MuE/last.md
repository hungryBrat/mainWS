# last.md — Frequency Response & Feedback Amplifiers Crash Reference

Procedural cheat-sheet for the two clusters where YouTube/notes are weak. Read top-to-bottom in one sitting (~60 min). Every formula here has shown up in 2021–2023 Part B.

---

# PART 1 — FREQUENCY RESPONSE (Part B Q3, ~17 marks)

## 1.1 The High-Frequency MOSFET Model

Take your low-frequency small-signal model (gate floating, drain-source dependent current `gm·vgs`, output resistance `ro` from D to S) and **add three capacitors**:

```
        Cgd
   G ----||---- D
   |           |
  Cgs        gm·vgs (current source D→S)
   |           |
   S          ro (D to S)
              |
              Cdb (D to bulk = GND for NMOS)
```

| Cap | Between | Role |
|-----|---------|------|
| Cgs | Gate–Source | Input cap, big |
| Cgd | Gate–Drain | The Miller cap — small but amplified |
| Cdb | Drain–Bulk | Output node cap |
| Csb | Source–Bulk | Usually irrelevant if source grounded |

**That's it.** Forget every other cap unless the problem explicitly gives you one.

## 1.2 Transit Frequency fT

The frequency at which the MOSFET's current gain falls to 1.

```
fT = gm / [2π · (Cgs + Cgd)]      [Hz]
ωT = gm / (Cgs + Cgd)             [rad/s]
```

Direct one-line formula. Plug numbers in. Done.

## 1.3 The Miller Approximation — When and How

**When to use it:** You have a CS-style amplifier (gate is input, drain is output), Cgd bridges the input and output, and the gain from input to output is some negative number `−Av` (where `Av > 0`).

**The trick:** Cgd "looks bigger from the input side" because the drain swings opposite to the gate.

```
At input:   C_in,Miller = Cgd · (1 + Av)        ← this dominates input pole
At output:  C_out,Miller = Cgd · (1 + 1/Av)     ← ≈ Cgd if Av is large
```

**Procedure for a CS amp:**

1. Find low-freq gain magnitude: `Av = gm · Rout` where `Rout = RD || ro` (or whatever load).
2. Total cap at input node: `C_in = Cgs + Cgd(1+Av)`
3. Total cap at output node: `C_out = Cdb + Cgd(1 + 1/Av) ≈ Cdb + Cgd`
4. Pole at input: `ωp,in = 1 / (R_signal · C_in)` where R_signal is the resistance driving the gate
5. Pole at output: `ωp,out = 1 / (Rout · C_out)`
6. **Dominant pole** = whichever is smaller (lower frequency).

## 1.4 OCTC Method (Open-Circuit Time Constants)

When Miller is messy or the circuit has more nodes (cascode, common-gate stage, source follower), OCTC gives you the dominant pole directly.

**Procedure:**

1. **Kill all independent sources** (Vsig → 0, but keep DC bias intact in the small-signal model).
2. For each capacitor `Ck` in the circuit:
   - Open all OTHER caps.
   - Inject a test current at the terminals of Ck.
   - Find the resistance `Rk` seen across that cap's terminals.
   - Compute `τk = Ck · Rk`.
3. Sum: `τH = Σ τk = Σ (Ck · Rk)`
4. Dominant high-frequency pole: `ωp1 ≈ 1/τH`
5. Hence `f-3dB ≈ 1 / (2π·τH)`.

**Why this works:** OCTC over-estimates τH slightly but gives a clean formula for the −3dB BW when one pole is dominant.

**For Cgd specifically (the tricky one):** When you find the resistance seen by Cgd, you cannot just say "RD || something" because Cgd connects two nodes that are coupled by gm. The shortcut: 
```
R_seen_by_Cgd = R_gate + R_drain + gm · R_gate · R_drain
```
where `R_gate` is the Thevenin resistance from gate to ground (with Cgs and Cgd opened), `R_drain` is the Thevenin from drain to ground.

## 1.5 Pole/Zero Identification — Intuitive Method

Each independent capacitor in the circuit produces **one pole**. The pole frequency is `1/(R·C)` where R is the resistance seen at that cap's nodes.

**Zeros come from:**
- A signal path that bypasses the transistor (e.g., direct path through Cgd from gate to drain). This produces a **right-half-plane (RHP) zero** at `ωz = gm/Cgd` for a CS stage.
- A current-mirror load with finite output cap → **mirror pole** (a pole, not a zero), typically at `gm3/Cmirror` where M3 is the diode-connected mirror device.

**Number of poles = number of independent capacitors that store energy at independent nodes.**

## 1.6 Bode Plot Construction (Corner-Plot Method)

You'll be asked: "Sketch the Bode magnitude plot, mark Ao, all poles, all zeros, and UGB."

**Step-by-step:**

1. **Plot starts at low-frequency gain `|Ao|` in dB**, flat.
2. At each pole frequency `ωpk`: slope changes by **−20 dB/dec**.
3. At each zero frequency `ωzk`: slope changes by **+20 dB/dec**.
4. **UGB** is the frequency where the magnitude curve crosses 0 dB.
5. For a single dominant pole: `UGB = |Ao| · ωp1` (gain-bandwidth product).
6. For two poles before unity-crossing: `UGB ≈ √(|Ao|·ωp1·ωp2)` (unusual on this exam).

**Sketch convention:** Use log-log paper with rad/s on x-axis, dB on y-axis. Label each corner. Show slopes between corners.

## 1.7 Phase Calculation at a Specific Frequency

```
∠H(jω) = − Σ arctan(ω/ωpk)  + Σ arctan(ω/ωzk)
```

For an RHP zero, the phase contribution is **−arctan(ω/ωz)** (subtracts, doesn't add — this is what makes RHP zeros bad for stability).

**Worked-style example (mirroring 2023 Q3 part d):**
"Calculate phase at ω = 10K rad/s with poles at 1K and 100K rad/s, no zeros."
```
Phase = −arctan(10K/1K) − arctan(10K/100K)
      = −arctan(10) − arctan(0.1)
      = −84.3° − 5.7°
      = −90°
```

## 1.8 UGB Calculation Methods

**Method 1 (single dominant pole, most common):**
```
UGB = |Ao| × ωp1
```

**Method 2 (from open-loop and closed-loop relationship for feedback):**
```
UGB(closed loop) = UGB(open loop)    [GBW preserved]
ωH,f = ωH · (1 + Aβ)                  [closed-loop -3dB extends]
```

## 1.9 Worked Procedure: 2023 Part B Q3 Style

Circuit: Cascode CS stage (M1 = CS input transistor, M2 = CG cascode on top, M3 = current source bias).

**Given:** Vov = 0.2V, λ = 0.01, Cgs = 10pF, Cgd = 1pF, Cdb = Csb = 2pF, RD = 10kΩ, ID = 0.5mA.

**Step 1 — Compute small-signal parameters:**
```
gm = 2·ID/Vov = 2·0.5m/0.2 = 5 mA/V
ro = 1/(λ·ID) = 1/(0.01·0.5m) = 200 kΩ
```

**Step 2 — Identify nodes with caps:**
- Node X (drain of M1, source of M2): C at this node = Cdb1 + Csb2 + Cgs2
- Node Vout (drain of M2): Cdb2 + Cgd2 (Miller is small here because cascode pushes |Av_M2| → 1)
- Input node (gate of M1): Cgs1 + Cgd1·(1+Av_M1)

**Step 3 — Cascode trick:** The cascode has a low impedance node at X (looking into source of M2 ≈ 1/gm2 = 200Ω), so the gain across M1 is small (Av_M1 ≈ −gm1/gm2 ≈ −1). This means **Miller multiplication on Cgd1 is tiny** — that's the whole point of cascode. Result: input pole moves to high frequency.

**Step 4 — Dominant pole is at output:**
```
Rout (cascode) ≈ gm2·ro1·ro2 = 5m · 200k · 200k = 200 MΩ
But it's loaded by RD = 10kΩ, so effective Rout ≈ 10 kΩ
Cout = Cdb2 + Cgd2 ≈ 3pF
ωp,dom = 1/(10k · 3p) = 33.3 Mrad/s
```

**Step 5 — Low freq gain:**
```
|Ao| = gm1 · Rout_eff = 5m · 10k = 50 V/V
```

**Step 6 — UGB:**
```
UGB = |Ao| · ωp,dom = 50 · 33.3M = 1.67 Grad/s
```

This is the procedure. Memorise the steps, not the numbers.

---

# PART 2 — FEEDBACK AMPLIFIERS (Part B Q4, ~15 marks)

## 2.1 The Whole Topic in One Sentence

You have an amplifier `A`. You take a fraction `β` of the output and subtract it from the input. The new gain is `Af = A/(1+Aβ)`. Everything else is consequences.

## 2.2 Topology Identification — The 2x2 Rule

This is **the** skill. Get this right and you've earned 5 marks before doing any math.

There are two questions to ask:

**Q1: How is the feedback signal combined at the INPUT?**

| Visual cue | Type | What gets subtracted |
|---|---|---|
| Feedback signal connects to input via a **series element** (resistor in series with source, or feedback enters at the source/emitter terminal) | **Series** | Voltage |
| Feedback signal **lands at the same node** as the input signal (Kirchhoff's current sum) | **Shunt** | Current |

**Q2: How is the feedback signal SAMPLED at the OUTPUT?**

| Visual cue | Type | What's sensed |
|---|---|---|
| Feedback network connects **across** the output (in parallel with load) | **Shunt** | Output voltage |
| Feedback network is **in series** with the load (in the output current path) | **Series** | Output current |

Combine them — you get one of four:

| Input | Output | Topology Name | Amp Type | Ideal A | Ideal Rin | Ideal Rout |
|---|---|---|---|---|---|---|
| Series | Shunt | Series-Shunt | Voltage amp | V/V | ∞ | 0 |
| Shunt | Shunt | Shunt-Shunt | Transresistance | V/I (Ω) | 0 | 0 |
| Series | Series | Series-Series | Transconductance | I/V (S) | ∞ | ∞ |
| Shunt | Series | Shunt-Series | Current amp | I/I | 0 | ∞ |

**Mnemonic:** "First word = input port. Second word = output port. Shunt at a port pushes that port's R toward 0; series pushes it toward ∞."

## 2.3 Identification Walkthrough — The Visual Procedure

Given a circuit, do this in order:

1. **Find the feedback network** — usually a passive box (R's and C's) connecting output back to input.
2. **Trace the OUTPUT side first.** Look at where the feedback network attaches to the output node. Is it tapped across the output (sees Vout)? → **Shunt-output**. Is it in the path of the output current (sees Iout)? → **Series-output**.
3. **Trace the INPUT side.** At the input node of the basic amplifier (gate, base), is the feedback signal arriving as a current at that node (KCL with input)? → **Shunt-input**. Or is it modifying the source-degeneration / arriving in series with the input source? → **Series-input**.
4. Combine. Done.

**Quick check:** If you got "Series-Shunt", the amplifier should be a voltage amplifier with high Rin and low Rout — does that match the open-loop circuit? If yes, you're right.

## 2.4 Open-Loop Analysis (with loading)

To compute `A`, `Rin`, `Rout` you must "open the loop" but **keep the loading effect** of the feedback network.

**The two-port loading rule:**

| Topology | Input loading | Output loading |
|---|---|---|
| Series-Shunt | β-network with output **shorted** (find R into β-network from input side) — put in series with input | β-network with input **opened** — put in parallel with output |
| Shunt-Shunt | β-network with output **shorted** — put in parallel with input | β-network with input **shorted** — put in parallel with output |
| Series-Series | output **opened** — put in series with input | input **opened** — put in series with output |
| Shunt-Series | output **opened** — put in parallel with input | input **shorted** — put in series with output |

**Memory hook:** "Same port at the same end" — at any port, if that port is **shunt**, the other port of the β-network is **shorted** (and the loading is in parallel). If that port is **series**, the other port is **opened** (and the loading is in series).

## 2.5 Computing β

β is the **feedback factor** — the ratio of the feedback signal returned to the input over the signal sampled at the output.

```
β = (signal returned to input) / (signal sampled at output)
```

Units depend on topology:

| Topology | β units | Why |
|---|---|---|
| Series-Shunt | V/V | voltage in, voltage out |
| Shunt-Shunt | I/V (S) | voltage sampled, current returned |
| Series-Series | V/I (Ω) | current sampled, voltage returned |
| Shunt-Series | I/I | current in, current out |

**Procedure:** Once you've identified the β-network, compute it as a two-port. For a simple resistive divider (R1, R2) sampling Vout and returning Vf:
```
β = R1 / (R1 + R2)     [series-shunt example]
```

## 2.6 Closed-Loop Transformations — The Five Equations

Once you have `A`, `β`, `Rin` (open-loop, with loading), `Rout` (open-loop, with loading), `ωH` (open-loop −3dB):

```
Loop gain:        T = A·β
Closed-loop gain: Af = A / (1 + T)
```

**Rin transformation:**
```
Rinf = Rin · (1 + T)     if input port is SERIES
Rinf = Rin / (1 + T)     if input port is SHUNT
```

**Rout transformation:**
```
Routf = Rout / (1 + T)   if output port is SHUNT (voltage sensed)
Routf = Rout · (1 + T)   if output port is SERIES (current sensed)
```

**Bandwidth (always extends):**
```
ωH,f = ωH · (1 + T)
GBW = Af · ωH,f = A · ωH    (preserved)
```

**Memory hook:** "Negative feedback makes the amplifier behave more **ideally** for its type." Voltage amp ideally has Rin → ∞ and Rout → 0 — feedback raises Rin (series in) and lowers Rout (shunt out). Always pushes toward the ideal column in the table at 2.2.

## 2.7 Worked Procedure: 2023 Part B Q4 Style

Circuit: M1 (CS) → M2 (CS) → output. Feedback resistor RF from output back to gate of M1. Source resistance Rs at the source of M1.

**Step 1 — Identify topology:**
- **Output side:** RF taps Vout (in parallel with load) → **Shunt** at output.
- **Input side:** RF lands directly on the gate of M1 (same node as Vin via Rsig) → **Shunt** at input.
- Topology: **Shunt-Shunt** → Transresistance amplifier (input is current, output is voltage, A has units of Ω).

**Step 2 — Find loading on open-loop circuit:**
- At input: β-network is just RF. Short the output side of RF (it samples voltage, so set Vout=0). Then RF appears from gate to ground at the input. This loads Rin.
- At output: short the input side of RF (set Iin=0). Then RF appears from drain of M2 to ground at the output. This loads Rout.

**Step 3 — Compute open-loop A (transresistance):**
- Treat the amplifier as: current Iin enters gate node (now loaded with RF to ground); converts to Vgs1 = Iin·(RF || rgate) ≈ Iin·RF if RF small.
- Wait — for shunt-shunt, the open-loop A is `Vout/Iin`. So drive with a current source Iin at the input, find Vout.
- For two-stage CS-CS: `A = (Vout/Iin) = (RF || ...) · gm1·(RD1||...)·gm2·(RD2||RF||RL)` — work out specific to circuit.

**Step 4 — Compute β:**
- β = If/Vout where If = current through RF returned to input node when output is at Vout.
- For just RF: β = −1/RF (sign indicates inversion; magnitude `|β| = 1/RF`).
- Units: 1/Ω = S ✓ (matches shunt-shunt expectation).

**Step 5 — Compute closed-loop:**
```
T = A·β = A/RF              (numerical, dimensionless)
Af = A/(1+T)                (Ω, transresistance)
Rinf = Rin/(1+T)            (shunt input → divides)
Routf = Rout/(1+T)          (shunt output → divides)
ωHf = ωH·(1+T)              (BW extends)
```

**Step 6 — Sanity check:**
- A·β should be **dimensionless** for any topology. Verify: A in Ω, β in S → product is dimensionless. ✓
- Closed-loop must approach ideal: shunt-shunt → low Rin, low Rout. After transformation, both shrink. ✓

## 2.8 The Four Most Common Tricks/Gotchas

1. **Sign of β.** Always take magnitude when computing T = |Aβ|. The negative sign of A handles the "negative" in negative feedback.

2. **Loading is non-negotiable.** A common mistake is using the un-loaded gain `A` of the bare amplifier and then asking why the closed-loop gain prediction is wrong by 30%. Always compute A with the β-network's loading.

3. **What's "input" and "output" of the β-network?** The β-network has two ports. The port that's connected to the OUTPUT of the amplifier is the β-network's input. The port connected to the AMPLIFIER's input is the β-network's output. Don't get this backwards when you compute loading by shorting/opening ports.

4. **Shunt input = inject current.** For shunt-input topologies, the "input signal" you should think about is a current `Iin`, not a voltage. Use a Norton equivalent at the input. The closed-loop gain `Af = Vout/Iin` (or `Iout/Iin`) — the right ratio depends on output type.

## 2.9 Quick Reference: 8-Step Solution Template for Any Feedback Question

1. Identify topology (input port type, output port type) → name it.
2. Identify the β-network (the passive box returning output to input).
3. Loading at input: short or open the β-network's far port per the rule. Add to amplifier's input.
4. Loading at output: short or open the β-network's far port per the rule. Add to amplifier's output.
5. Compute open-loop `A`, `Rin`, `Rout`, `ωH` of the **loaded** amplifier.
6. Compute β from the β-network alone (units must match expected).
7. Compute T = Aβ (must be dimensionless).
8. Apply the five closed-loop equations: `Af`, `Rinf`, `Routf`, `ωHf`.

---

# PART 3 — Common Mixed Patterns (Both Topics)

## 3.1 "Find UGB in open and closed loop"

```
Open-loop UGB = A · ωH
Closed-loop UGB = A · ωH    [identical — GBW preserved]
But closed-loop −3dB = ωH · (1+T) [larger]
And closed-loop gain = A/(1+T)    [smaller]
```

So you give the same number for UGB twice — that's correct, not a mistake.

## 3.2 "Gain crossover frequency Gx in closed loop"

The frequency where `|loop gain T(jω)| = 1`. For a single-pole T(s):
```
T(s) = T0 / (1 + s/ωp)
|T(jωgx)| = 1  →  ωgx = ωp · √(T0² − 1)  ≈  T0·ωp     (if T0 >> 1)
```

So `Gx ≈ T0 · ωp = (Aβ) · ωH = β · UGB`. Useful shortcut.

## 3.3 "−3dB frequency of CMRR" (2021/2022 type sub-part)

CMRR(s) = Adm(s)/Acm(s). If Adm has dominant pole at ωd and Acm has dominant pole at ωc, the −3dB of CMRR is at the **pole of Acm** (because Acm rolls off and CMRR rises — wait, CMRR is the ratio, so its −3dB is determined by where Acm has its pole, since Adm pole appears in numerator and Acm pole in denominator).

**Shortcut:** −3dB of CMRR = pole of Acm = `1/(Rss · Css)` typically, where Rss is tail current source resistance.

---

## End-of-Reference Checklist

Before the exam, verify you can do these from memory in under 90 seconds each:

- [ ] Draw the high-freq MOSFET model with three caps placed correctly.
- [ ] State Miller approximation in 5 lines.
- [ ] Write the OCTC procedure in 4 numbered steps.
- [ ] Sketch a Bode plot from `Ao = 100`, ωp1 = 1k, ωp2 = 1M, ωz = 10k.
- [ ] Recite the 2x2 topology table.
- [ ] State the loading rule (shunt → short, series → open).
- [ ] Recite the five closed-loop equations.
- [ ] Identify topology of an arbitrary feedback circuit (use 2023 Q4, 2021 Q1).

If all eight are solid, you have ~32 marks across Q3 and Q4 locked in.
