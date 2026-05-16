# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# ACADS — Personal academic workspace

Long-running personal workspace for the user's undergraduate journey at **BITS Pilani, Pilani campus**. Houses both per-course exam-prep subspaces and degree-wide planning (DE picks, minor completion, PS/thesis routing). Not a codebase — nothing to build, lint, or test.

## Student context

- **Branch:** B.E. Electronics and Instrumentation (E&I) — course prefix `INSTR Fxxx`.
- **Year:** Completed 4 semesters as of 2026-05-15; entering Sem 5 in AY 2026-27.
- **Minor:** Pursuing **Minor in Robotics and Automation**.
- **HuELs:** 3 done.
- **Authoritative reference:** `Bulletin_2025-26.pdf` (951 pp, BITS Pilani bulletin). Pre-extracted plaintext at `Bulletin_2025-26.txt` (5.3 MB) — use `grep` on the `.txt` first, then `Read` the relevant `.pdf` page range. Page indices in the bulletin use `IV-NNN` format for program structures; line numbers in the `.txt` are stable.

## Key bulletin anchors (line numbers in `Bulletin_2025-26.txt`)

- E&I semester-wise pattern: ~12981
- E&I CORE + DE list: ~19121–19189 (pages IV-113 to IV-114). Two-column layout — DE list spans left col of IV-113 → right col of IV-113 → left col of IV-114.
- Minor in Robotics and Automation: 20934–20970 (p.IV-139)
- General minor rules + 2-course overlap cap: 20243–20305 (p.IV-128–129)

## Layout

```
ACADS/
├── CLAUDE.md
├── Bulletin_2025-26.pdf       ← BITS Pilani bulletin (authoritative)
├── Bulletin_2025-26.txt       ← pdftotext -layout extract for grep
├── ConSys/    INSTR F242  Control Systems
├── MuE/       INSTR F244  Microelectronic Circuits
├── MuP/       INSTR F241  Microprocessors & Interfacing
├── PoM/       MGTS F211   Principles of Management
├── PopLit/    HSS F316    Pop Lit & Culture of South Asia (HuEL)
└── SaS/       INSTR F243  Signals & Systems
```

Each course subdir is a sealed exam-prep workspace; most have their own `CLAUDE.md`. See child `CLAUDE.md` files for course-specific evaluation profile and tutoring conventions.

## Cross-workspace isolation

**Treat each course subdir as a sealed unit.** When working inside one course, do not read from siblings unless the user explicitly asks for a comparison. Course materials, PYQ patterns, and tutoring conventions are intentionally course-specific.

The parent (this `ACADS/` root) is the right place for **degree-wide concerns**: DE selection, minor planning, PS-1/PS-2 decisions, thesis routing, and bulletin queries.

## Inherited conventions (apply to all children unless overridden)

- **Persistent state outside the workspace.** The `teach` skill keeps per-course graph at `~/.claude/teach/<course>-2026/`. Load `course.json` before re-parsing source PDFs.
- **PYQ-driven study.** Past papers are the primary signal for what's worth learning.
- **Handouts are authoritative for evaluation scheme.** Course handout PDFs override memory for weights/format/dates.
- **PPT → PDF before reading.** `libreoffice --headless --convert-to pdf --outdir _pdf *.ppt *.pptx`.
- **PYQ screenshots are images.** `Read` them as images.
- **No refactoring of study materials.** These are short-lived crunch spaces.

## Working conventions for the parent (degree-wide work)

- **Always cite the bulletin when answering rule questions.** Quote line numbers from `Bulletin_2025-26.txt`. Page references use `IV-NNN` format.
- **Memory for the user's enrolment state lives in** `~/.claude/projects/-home-agush-mainWS-workspaces-ACADS/memory/` — check it before re-deriving facts about year, completed courses, or minor status.
- **Confirm before registering or applying.** Any plan that affects course registration, minor declaration, or PS allocation needs explicit user confirmation — don't act unilaterally even if the analysis seems clear.

## Parent

Child of `~/mainWS/workspaces/`. See `~/mainWS/CLAUDE.md` for Mission Control conventions. ACADS is **not yet registered** in `~/mainWS/workspaces/INDEX.md` — add when stable.
