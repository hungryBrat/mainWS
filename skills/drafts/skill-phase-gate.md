---
name: phase-gate
description: >
  Read a ROADMAP.md and surface the current active phase: incomplete tasks,
  gate criteria status, and demo target. Triggers on "phase gate", "where am I
  in the roadmap", "what's left in this phase", "check my phase", or /phase-gate.
disable-model-invocation: false
argument-hint: "(optional: [path/to/ROADMAP.md]) (optional: all)"
---

# Skill: Phase Gate

**Status:** draft
**Deployed to:** `~/.claude/skills/phase-gate.md` *(fill in when promoted)*
**Cost:** negligible — 1 Read + inline parse, no agents

---

## Purpose

During a multi-phase project, you need to know exactly where you are and what's
blocking you from advancing — without opening and manually scanning the ROADMAP.md.
This skill reads the roadmap, finds the first phase with incomplete work, and surfaces
only the actionable information: what's left, what the gate requires, what the demo is.

It exists as a skill because "check my progress" is a frequent, low-friction query that
should return a structured answer in seconds, not require navigating a long document.

## When to use vs. alternatives

| Use this skill when... | Use X instead when... |
|---|---|
| Starting a work session — "where was I?" | You want to edit the roadmap — use Read directly |
| Deciding whether you can advance to the next phase | You want a full project overview — read ROADMAP.md |
| Quick check of gate criteria before a demo | You want cross-project status — use `project-brief` |

---

## Arguments

- `$0` — Path to ROADMAP.md. *(optional)* Default: auto-detect (see Step 1).
- `$1` — `all` to show a summary table of every phase's status instead of focusing on
  the current phase. *(optional)*

If invoked with no arguments: auto-detect ROADMAP.md and show current phase.

---

## Execution

### Step 1 — Locate ROADMAP.md

Search in this order, stop at first match:

1. If `$0` is provided: use that path directly.
2. `./ROADMAP.md` (current working directory)
3. Walk up parent directories looking for `ROADMAP.md`
4. Search `./workspaces/*/ROADMAP.md` (mainWS hub pattern)

If no ROADMAP.md found: report the search locations tried and ask the user to provide
the path explicitly.

### Step 2 — Parse phases

Read the file. Identify all phase sections: headers matching `## Phase N` or
`## Phase N —` (with any title suffix).

For each phase, extract:
- **Tasks:** lines matching `- [ ]` (incomplete) and `- [x]` (complete)
- **Gate criteria:** the `### Gate criteria` subsection — its `- [ ]` and `- [x]` lines
- **Demo:** the `### Demo` subsection text (first paragraph only)
- **Duration/week target:** from the phase header line if present

Compute per-phase:
- `tasks_done` = count of `- [x]` lines (excluding gate criteria section)
- `tasks_total` = count of all `- [ ]` + `- [x]` lines (excluding gate criteria)
- `gates_done` = count of `- [x]` in gate criteria section
- `gates_total` = count of all gate criteria lines
- `status`: `complete` if all gate criteria checked, `active` if any task incomplete,
  `not started` if zero tasks checked

### Step 3 — Determine current phase

**Default view (no `all` flag):**

Current phase = first phase whose `status` is `active`.

If no active phase found (all complete): report project complete with a summary.
If all phases are `not started`: treat Phase 0 (or first phase) as current.

**All-phases view (`$1 = all`):**

Skip this step; go directly to Step 4b.

### Step 4a — Report current phase

Output the following, in order:

```
## Phase [N] — [Name]  ([tasks_done]/[tasks_total] tasks · [gates_done]/[gates_total] gates)

**Progress:** [████████░░] 80%   ← ASCII bar, 10 blocks, proportional fill

### Remaining tasks
- [ ] task one
- [ ] task two
...

### Gate criteria
- [x] gate one  ✓
- [ ] gate two  ← BLOCKING
...

### Demo target
[demo description from ROADMAP.md]

---
Next phase: Phase [N+1] — [Name]
```

Rules:
- Show only incomplete tasks in "Remaining tasks" (already-checked ones are done, skip them)
- In "Gate criteria", show ALL criteria (checked and unchecked) — knowing what's already
  passed matters
- Mark any unchecked gate criterion with `← BLOCKING`
- If zero remaining tasks but unchecked gates exist, flag this explicitly:
  "All tasks complete — gate criteria not yet verified"

### Step 4b — Report all phases (when `$1 = all`)

Output a summary table followed by the current phase detail:

```
## Roadmap Status

| Phase | Name | Tasks | Gates | Status |
|---|---|---|---|---|
| 0 | Bench Characterization | 5/5 | 4/4 | complete ✓ |
| 1 | Assembly + tf2 | 3/8 | 0/3 | active ← |
| 2 | Control + Odometry | 0/6 | 0/3 | not started |
...

Current: Phase 1 (detail below)
[then full Step 4a output for current phase]
```

---

## Output format

Printed inline to the conversation. No file written.

The output is designed to be scanned in under 10 seconds:
- Progress bar gives instant % feel
- Remaining tasks are the action list
- BLOCKING labels on gate criteria are the stop signs

---

## Design notes

- **Why parse gate criteria separately from tasks:** Gates are binary go/no-go signals;
  tasks are work items. A phase can have all tasks checked but gates unverified (e.g.,
  a bench test not yet run). Conflating them hides this state.
- **Why ASCII progress bar:** A number alone (3/8) is less immediately readable than a
  visual fill. The bar is 10 blocks so each block = 10% — easy mental math.
- **Why not write output to a file:** Phase gate is a query, not a record. Writing it
  would pollute the journal with noise. If the user wants to record the state, they
  should use `journal`.
- **Known limitation:** Checkbox parsing assumes standard `- [ ]` / `- [x]` GitHub
  Flavored Markdown. Indented sub-tasks (nested checkboxes) are counted but their
  parent task's state is not inferred. If you have nested tasks, count may be off.
- **Known limitation:** Section detection relies on `### Gate criteria` and `### Demo`
  headers existing verbatim. If your ROADMAP.md uses different header names, edit the
  skill's Step 2 to match.

---

## Related skills

- `journal` — use after a phase-gate check to log what you found or decided
- `project-brief` — broader workspace status (not yet built); phase-gate is the
  phase-level zoom

---

*Drafted: 2026-04-28*
*Promoted: (fill in when deployed)*
