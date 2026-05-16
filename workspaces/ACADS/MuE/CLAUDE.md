# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is an exam-prep workspace for **ECE F244 — Microelectronic Circuits** (BITS Pilani, Pilani Campus, Sem 2 2025-26). It is a teaching and study space, not a codebase. The primary activity is tutoring, problem-solving, and formula-sheet building toward the Comprehensive Examination.

## Workspace Contents

- `ECE_F244_*.pdf` — Course handout with syllabus, module plan, and evaluation scheme.
- `PYQs/` — Previous year comprehensive exam papers (2013–2023) and select solutions.
- `STUDY_GUIDE.md` — **Canonical study reference.** 7-tier breakdown of every exam-relevant problem type (easy → hard), each with theory, formulas, algorithm, and worked PYQ examples in LaTeX. Ends with 3 mixed multi-tier integration problems and a PYQ cross-reference table.

## Study Guide Tier Map

The guide organises all syllabus content by problem-type tiers, not by textbook chapters. When teaching, **reference the tier number** so the student can locate the relevant section:

| Tier | Topic | Maps to PYQ slot |
|---|---|---|
| 1 | MOSFET device problems (region, CLM, body effect, intrinsic gain, sizing) | Part A small parts |
| 2 | DC biasing (self-bias, divider, load line) | Part A Q1, Part B Q2(a) |
| 3 | Single-stage amplifiers (CS, CD, CG, cascode, active-load variants) | Part A Q1, Part B Q2(b) |
| 4 | Current mirrors (basic, cascode, low-voltage, BJT) | Part A Q2 |
| 5 | Differential amplifier (DC, $A_{dm}$, ICMR, swing, offset, CMRR) | Part B Q1 |
| 6 | Frequency response (HF model, Miller, OCTC, Bode, PM, RHP zero) | Part B Q3 |
| 7 | Feedback amplifiers (topology ID, $\beta$, $A_f$, $R_{in,f}$, $R_{out,f}$, BW) | Part B Q4 |

When the student asks a question or attempts a PYQ, locate it on this map first, then point them to the corresponding STUDY_GUIDE section before working through it.

## Exam Context

- **Comprehensive Exam date:** 11 May 2026, 9am
- **Weightage:** 105 marks (35% of total)
- **Structure:** Part A Closed Book (40–60 marks, 60–90 min) + Part B Open Book (60–70 marks, 90–120 min)
- **Textbook:** Sedra & Smith, *Microelectronic Circuits*, 7th Ed. (primary)
- **Reference:** Razavi, *Design of Analog CMOS Integrated Circuits* (maps to Prime Ref chapters cited in course plan)

## Exam Structure (from 2021–2023 PYQs)

### Part A — Closed Book
Two questions, ~20 marks each (some years 4 questions at 15 marks):
- **Q1:** Biased MOSFET circuit — DC Q-point (IDQ, VGSQ, VDSQ), then gm/ro, Rin/Rout, voltage gain and current gain.
- **Q2:** One of: current mirror W/L sizing; PMOS ID in triode/saturation; CS with active load; VTC sketch; cascode small-signal model.

### Part B — Open Book (4 questions, every year without exception)

| Question | Topic | Marks |
|----------|-------|-------|
| Q1 | Differential amplifier | ~20 |
| Q2 | CS bias/load-line + cascode small-signal | ~19 |
| Q3 | Frequency response (high-freq model, Miller/OCTC, Bode, UGB) | ~17 |
| Q4 | Feedback amplifier (topology ID, open/closed loop, Rin/Rout, BW) | ~15 |

Current mirrors appear in Part A for sizing problems and as bias elements inside diff amp / opamp questions — not as a standalone Part B question in recent papers. 2-stage CMOS OpAmp has not appeared as a dedicated Part B question in 2021–2023; its sub-formulas (Adm, ICMR, Rout) are absorbed into Q1/Q2.

## Priority Order for a Zero-Theory Student

1. MOSFET ID equation + small-signal model (gm, ro) — unlocks Part A and all of Part B
2. Differential amplifier — highest Part B marks, most predictable question
3. Feedback amplifier topology identification — easiest marks once the 2x2 rule is internalized
4. Frequency response (Miller + OCTC + Bode) — mechanical once the algorithm is known
5. CS/Cascode gain + DC bias workflow — Part B Q2 and all of Part A

## Formula Anchors by Cluster

**MOSFET Basics**
```
Saturation: ID = (μCox/2)(W/L)(VGS−VT)²(1+λVDS)
Triode:     ID = μCox(W/L)[(VGS−VT)VDS − VDS²/2]
gm = 2ID/Vov = √(2μCox(W/L)·ID)
ro = 1/(λID) = VA/ID
Intrinsic gain: μ = gm·ro = 2VA/Vov
```

**Single-Stage Amplifiers**
```
CS:  Av = −gm(RD||ro||RL),  Rin = ∞,  Rout = RD||ro
CS with Rs:  Av = −gm·RD/(1+gmRs)
CD (source follower): Av ≈ gmro/(1+gmro) ≈ 1,  Rout ≈ 1/gm
CG:  Av = gm(RD||ro),  Rin = 1/gm
Cascode: Rout = ro2(1+gm2·ro1) ≈ gm2·ro1·ro2
```

**Differential Amplifier**
```
Adm (resistive load) = −gm·RD          [half-circuit, single-ended]
Adm (active load)    = −gm(ron||rop)
Acm ≈ −RD/(2Rss)  →  CMRR ≈ 2gm·Rss
ICMR,min = VSS + Vov,tail + VTN
ICMR,max = VDD − |Vov,load| − |VTP|    (resistive load)
           VDD − |Vov,load|             (mirror load)
Output swing per side: Vov below VDD, Vov above VSS
Offset: Vos = ΔVT + (Vov/2)·(ΔID/ID − ΔW/W)
```

**Frequency Response**
```
High-freq model adds: Cgs (gate-source), Cgd (gate-drain, Miller path), Cdb (drain-bulk)
fT = gm / [2π(Cgs+Cgd)]
Miller: Cin = Cgd(1+|Av|),  Cout = Cgd(1+1/|Av|)
OCTC: τH = Σ Ck·Rk  (Rk = resistance seen by Ck, all other caps open)
Dominant pole: ωp1 = 1/τH
UGB = |Ao|·ωp1    (single dominant pole)
RHP zero (Miller Cc): ωz = gm/Cc
Bode corner plot: −20 dB/dec per pole, +20 dB/dec per zero
```

**Feedback Amplifiers**
```
Identify input port: series → V subtracted, shunt → I subtracted
Identify output port: shunt → V sensed, series → I sensed
Topologies: Series-Shunt (Vamp), Shunt-Shunt (Rtrans), Series-Series (Gtrans), Shunt-Series (Iamp)
Af = A/(1+Aβ),   loop gain T = Aβ
Rinf  = Rin·(1+T)   [series input]   or  Rin/(1+T)   [shunt input]
Routf = Rout/(1+T)  [shunt output]   or  Rout·(1+T)  [series output]
ωHf = ωH·(1+T)    (gain-BW product preserved)
```

## Default Device Parameters (unless stated otherwise in a problem)

```
VDD = VCC = 3.3V
NMOS: μnCox = 140 μA/V², VT = 0.7V, λ = 0.1 V⁻¹, Vov = 0.2V
PMOS: μpCox = 40 μA/V², VT = −0.7V, λ = 0.1 V⁻¹, Vov = 0.2V
BJT:  β = 100, VA = 100V, VBE,on = 0.6V, kT/q = 25mV
```

Bulk of NMOS → GND; bulk of PMOS → VDD (unless body effect is explicitly part of the question).

## Tutoring Conventions

- **Always tie explanations to PYQ patterns.** When teaching a concept, show how it maps to a real compre question type and specific sub-part marks.
- **Part A answers must be derivable from memory** — do not assume formula sheet access. Teach the derivation once, then drill the result.
- **Part B allows open book** — focus on the analysis workflow (DC → small-signal model → apply formula cluster), not formula recall.
- **Exam style:** Show full procedure, report answers with units, state assumptions explicitly. Partial credit is given for correct methodology even with a numerical error — always write the setup.
- **BJT is self-reading** per the course plan but appears in ~50% of Part B papers as one sub-part. Structurally identical to MOSFET analysis; gm = IC/VT, ro = VA/IC. Lower priority than MOSFET topics.
- **Student context:** Starting from zero theory. Prefer concrete worked examples over derivation-first explanations. One concept → one PYQ sub-part, immediately.
- When building a formula sheet, organize by the four Part B question slots above, not by topic abstraction.
