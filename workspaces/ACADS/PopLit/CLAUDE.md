# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace purpose

This is an **exam-preparation workspace**, not a code repository. There is nothing to build, test, or lint.

It contains source materials for **HSS F316 — Popular Literature and Culture of South Asia** (BITS Pilani, Sem 2 2025-26, instructor Dr. Shriya Raina), and exists to support the user's preparation for the comprehensive exam (40% weight, 180 min, open book, **2026-05-04**).

## Source materials

- `Popular Literature and Culture of South Asia (1).pdf` — official course handout (Part II): course description, evaluation scheme, lecture-by-lecture course plan with prescribed texts and films
- `Course review 04.03.26_PLCSA.docx` — instructor's course review document with main analytical points covered per unit

These are read-only inputs; do not modify them.

## Persistent state outside this directory

The course has been ingested via the `teach` draft skill (`~/mainWS/skills/drafts/skill-teach.md`). Persistent state lives at:

```
~/.claude/teach/pop-lit-south-asia-2026/
├── course.json         ← topic graph, evaluation components, key concepts per unit
├── student.json        ← mastery vector (currently empty — no diagnostics run)
├── reports/
│   └── plan_2026-05-03.md     ← 12-hour pre-exam study plan
└── teaching/
    └── full_session.md         ← full crash-course content, priority-ordered
```

When working on this course, **load `course.json` for unit/topic structure** rather than re-parsing the source documents.

## Course structure (8 units in priority order for the comp exam)

1. Theoretical frameworks (introduction lectures) — culture as process, margin–centre, transnationalism, gendering, politics of representation, diaspora
2. India — *The White Tiger* (Adiga), *Slumdog Millionaire*
3. Pakistan — Manto stories, *Khamosh Pani*, *The Reluctant Fundamentalist*
4. Comparative synthesis — gender/violence, partition's afterlife, labour/bodies, narration
5. Afghanistan — *Osama* (2003), *The Kite Runner* (novel + 2007 film)
6. Sri Lanka — *Funny Boy* (Selvadurai)
7. Bangladesh + Nepal — *Lajja* (Nasrin), *The Machinists*, *True Cost*; "Hands"
8. Bhutan — *The Circle of Karma* (Choden)

## Exam profile

| Component | Weight | Format | Duration |
|---|---|---|---|
| Continuous evaluations | 40% | Take-home, rolling | — |
| Mid-Semester | 20% | Open book | 90 min |
| **Comprehensive (2026-05-04)** | **40%** | **Open book** | **180 min** |

No past papers are available. Exam-frequency weighting in `course.json` is therefore unset; planning is syllabus-driven.

## Conventions for this workspace

- **Don't add code, tests, or build infrastructure** — this workspace doesn't need any.
- **Don't create new top-level files** without asking. Materials belong in source-doc folders or the teach skill state directory.
- When user requests teaching, study plans, or exam prep, **work from `course.json` and `teaching/full_session.md`** rather than re-deriving from the PDFs.
- The user prefers direct, opinionated teaching over hedged summaries. Use the analytical phrases established in `teaching/full_session.md` for consistency across sessions.

## Parent workspace

This is a child workspace under `~/mainWS/workspaces/ACADS/`. See `~/mainWS/CLAUDE.md` for the broader Mission Control conventions (skill lifecycle, journal, scratch, etc.).
