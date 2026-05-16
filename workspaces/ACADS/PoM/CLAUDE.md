# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# PoM — Principles of Management (study workspace)

Study workspace for BITS Pilani **MGTS F211 — Principles of Management** (II Semester 2025–26, Robbins & Coulter 15th ed.). Not a codebase — there is nothing to build, lint, or test.

## Purpose

Prepare for evaluations against the course handout (`Course Handout.pdf`). Comprehensive exam is 3 hours, 35% weight, close-book by department convention (handout says TBA). The exam is case-application: examiners reward NAMING the framework, LISTING its components verbatim, then APPLYING to a scenario.

## Directory layout

```
PoM/
├── CLAUDE.md                ← you are here
├── Course Handout.pdf       ← syllabus, 20 topics, 4 parts, evaluation scheme
├── Course Content/          ← lecture material (PDFs + PPT/PPTX)
│   └── _pdf/                ← PPTs auto-converted to PDF (libreoffice --headless)
├── PYQs/                    ← previous year question screenshots (PNG)
└── STUDY_GUIDE.md           ← consolidated revision document (83 KB, all 19 topics, cheat-sheet, model Q&A, PYQ pattern)
```

## Working conventions

- **PPTs must be converted before reading.** `libreoffice --headless --convert-to pdf --outdir _pdf *.ppt *.pptx` from inside `Course Content/`. The `Read` tool handles PDFs natively but not `.ppt`/`.pptx`.
- **Long extractions → parallel agents.** When summarizing all course content, dispatch one general-purpose subagent per syllabus Part (Intro / Managerial Competencies / Mgmt Functions / Business Functions). `Explore` agents are read-only and cannot Write — use `general-purpose` if the agent must persist output.
- **Ephemeral scratch files** use the `.tmp_*.md` prefix and are not durable. Final output is consolidated in `STUDY_GUIDE.md` and the scratch files are deleted.
- **PYQs are screenshots** — `Read` them as images. Two papers are present in this set; question patterns repeat across them.

## Syllabus shape (from handout)

Four parts, 36 LH, 20 topics:
1. **Introduction** — Mgmt & orgs, Mgmt history, External environment, Org culture, CSR & ethics, Global environment
2. **Managerial Competencies** — Teams, Communication, Personality
3. **Management Functions** — Decisions/Planning, Org Structure, Motivation, Leadership, Controlling
4. **Business Functions** — Marketing, HRM, Finance, Operations, Strategy

High-yield topics (appear in both PYQ papers): decision-making under uncertainty (Maximax/Maximin/Laplace/Minimax-regret on a payoff matrix), Porter's value chain, Balanced Scorecard, financial ratios + fund flow, Fiedler / Hersey-Blanchard / Path-Goal leadership, Job Characteristics Model, 4Ps marketing mix, BCG matrix, Porter's 5 forces, conflict types, communication channel choice.

## Collaboration style

Direct and exam-focused. Frameworks over prose. When summarizing course material, preserve verbatim lists/elements — examiners mark on those, not paraphrase. Confirm before creating new directories or moving files outside this workspace.
