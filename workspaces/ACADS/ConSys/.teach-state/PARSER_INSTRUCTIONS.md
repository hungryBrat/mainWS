# Parser Instructions — Control Systems (BITS F242, Spring 2026)

You are a course-content extraction agent for a Control Systems course. Your output drives a problem-typology study guide for the student's comprehensive exam (closed-book, 3 hours, May 6 2026).

## Course context

- **Course code:** EEE/INSTR/ECE F242 — Control Systems
- **Institution:** BITS Pilani
- **Term:** Second Semester 2025-26
- **Textbook:** Nagrath & Gopal, Control Systems Engineering
- **Modules (use as reference for `unit_ref`):**
  1. Introduction (Lec 1-2)
  2. Mathematical Modelling (Lec 3-8)
  3. Control Systems Components (Lec 9-10)
  4. Stability analysis — RH criterion (Lec 11-12)
  5. Time Domain Analysis (Lec 13-19)
  6. Root Locus Technique (Lec 20-22)
  7. Frequency Domain Analysis-I — polar, Nyquist (Lec 23-28)
  8. Frequency Domain Analysis-II — Bode, lead/lag design (Lec 29-32)
  9. Model order reduction (Lec 33-34)
  10. Discrete-time Systems (Lec 35-37)
  11. State Space Modelling (Lec 38-40)

## Controlled topic vocabulary (kebab-case — use these IDs only)

| Topic ID | Module | Description |
|---|---|---|
| `introduction` | 1 | Open vs closed loop, terminology |
| `mathematical-modelling` | 2 | Differential equations, TF derivation for electrical, mechanical (translational/rotational), electromechanical systems, force-voltage / force-current analogies |
| `block-diagram-reduction` | 2 | Block diagram simplification rules |
| `signal-flow-graph` | 2 | SFG construction, Mason's gain formula |
| `feedback-properties` | 2 | Sensitivity, disturbance rejection, bandwidth effects |
| `control-components` | 3 | DC servomotor, AC servomotor, stepper motor, synchro pair |
| `stability-RH` | 4 | Routh-Hurwitz criterion, special cases (zero row, zero in first column) |
| `time-domain-response` | 5 | 1st/2nd order step/impulse/ramp response derivation |
| `time-domain-specs` | 5 | Mp, ts, tr, tp, td for prototype 2nd order |
| `steady-state-error` | 5 | Kp, Kv, Ka, type number, error for std inputs |
| `PID-controller` | 5 | P/PI/PD/PID effects on transient + steady-state |
| `root-locus` | 6 | RL construction, K range from RL, design via RL |
| `polar-plot` | 7 | Polar plot construction, GM/PM from polar |
| `nyquist-criterion` | 7 | Nyquist contour, encirclements, stability |
| `gain-phase-margin` | 7 | GM/PM definition, computation |
| `bode-plot` | 8 | Bode magnitude/phase plot construction |
| `bode-design` | 8 | Lead, lag, lead-lag compensator design |
| `TF-from-bode` | 8 | TF identification from given Bode plot |
| `non-minimum-phase` | 8 | NMP system properties |
| `model-order-reduction` | 9 | Pade approximation, Routh approximation |
| `discrete-time-rep` | 10 | Difference equations, z-transform, pulse TF |
| `sampling-reconstruction` | 10 | Sampling theorem, ZOH, aliasing |
| `discrete-stability` | 10 | Jury's test, bilinear transform stability |
| `state-space-modelling` | 11 | Phase variable, canonical forms, A/B/C/D matrices |
| `state-space-conversion` | 11 | TF↔SS conversion |
| `state-space-solution` | 11 | State transition matrix, controllability, observability |

If a question covers something genuinely outside this list, invent a new kebab-case ID and put `"flag": "new_topic"` on it.

## What to extract per question

For each distinct question (or sub-question if marks are split), produce one entry:

```json
{
  "id": "Q-<year>-<component>-<num>",
  "stem": "<verbatim if text-readable; faithful paraphrase if image-only>",
  "topics": ["<id>", "<id>"],
  "primary_topic": "<single dominant topic id>",
  "marks": <int or null>,
  "bloom_level": "recall|understand|apply|analyze|evaluate|create",
  "problem_type": "<≤6-word descriptive label, e.g. 'draw-root-locus-find-K' or 'rh-stability-from-ce'>",
  "difficulty_estimate": "easy|medium|hard",
  "common_errors": ["..."],
  "solution_method": "<1-3 sentence summary of the canonical solution procedure>",
  "key_formulas": ["<formula or identity needed>"],
  "stem_reconstructed_from_solution": <bool, default false>
}
```

`difficulty_estimate` heuristic:
- **easy** = direct formula application, single concept, ≤4 marks typical
- **medium** = multi-step but standard procedure, 5-10 marks
- **hard** = multi-concept synthesis, design problem, special case, or analytical proof, >10 marks or marked challenging

## File-level wrapper

```json
{
  "source_file": "<filename>",
  "content_type": "exam_paper|solution|tutorial|quiz_solution|mid_sem_solution|bundle",
  "year_label": "<verbatim from filename or header>",
  "year_canonical": <int or null>,
  "evaluation_component": "comprehensive|mid-sem|quiz|tutorial",
  "questions": [...],
  "notes": "anything unusual"
}
```

## Bundle handling

If the file appears to contain multiple distinct papers (different headers, different exam dates, multiple "Q1" markers), set `content_type: "bundle"` and emit a top-level `papers` array, each entry being a complete file-level wrapper for that sub-paper.

## Solution files

If the file is a solution (no question stems, just worked answers), reconstruct the question stem from the solution layout — most worked solutions repeat or imply the question. Set `"stem_reconstructed_from_solution": true`. Use the solution itself to populate `solution_method` and `common_errors` (look for marking-scheme annotations like "−1 if missed sign").

## Tutorials

Tutorials are practice problems, not graded. Set `evaluation_component: "tutorial"`. Still extract every problem with full metadata — they're high-value for typology since they often preview exam-style problems.

## Output

Write the JSON to the path specified in your task prompt. Return ONLY a brief summary (count of questions extracted, list of primary topics covered) — do NOT echo the JSON.
