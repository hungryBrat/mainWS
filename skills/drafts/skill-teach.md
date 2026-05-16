---
name: teach
description: >
  Master tutor for any course or certification. Ingests syllabi, handouts, slides,
  and past papers from any institution; classifies external reference papers
  separately from own-course papers; extracts a topic model weighted by what
  examiners actually test; runs diagnostic quizzes to infer real (not self-reported)
  knowledge gaps; generates novel practice problems calibrated to a 30-40% error
  rate (Bjork's desirable difficulty); emits study plans that are format-aware
  (closed-book, open-book, oral, MCQ, project, lab) and scoped to a specific
  upcoming evaluation component by name. Handles any course structure: modular
  lectures, chapter-based, flat topic lists, or certification syllabi with no
  declared structure. Persists state per course on disk so sessions accumulate.
  Triggers on /teach, "tutor me on X", "study program for X", "exam prep for X",
  "quiz me on X", "find my gaps in X", or any subcommand:
  ingest|diagnose|quiz|gaps|plan|review.
disable-model-invocation: false
argument-hint: "<subcommand> [args] — subcommands: ingest|diagnose|quiz|gaps|plan|review"
---

# Skill: Teach — Exam-Focused Master Tutor

**Status:** deployed
**Deployed to:** `~/.claude/skills/teach/SKILL.md`
**Cost:** ingest is heavy (1 Sonnet call per ~10 pages of material). All other commands are 1 Sonnet call each.

---

## Purpose

Build a personal tutor that treats past exam papers as ground truth for "exam-ready," diagnoses knowledge gaps from demonstrated performance, and produces specific, time-anchored study plans. Generic AI tutors fail at exam prep because they optimize for helpfulness and conflate syllabus coverage with student weakness. This skill inverts both: it withholds answers by default and separates "what examiners test" from "what this student doesn't know."

## When to use vs. alternatives

| Use this skill when... | Use X instead when... |
|---|---|
| Preparing for a specific exam with past papers available | Browsing a topic for general curiosity (use a chat) |
| You want persistent state across study sessions | One-off question about a concept (just ask) |
| You need diagnostic gap detection, not summarization | Reading material to learn for the first time |

---

## Arguments

`/teach <subcommand> [args]`

Subcommands:
- `ingest <path-to-course-dir>` — parse and structure all course materials
- `diagnose [--topic X]` — run a calibrated diagnostic quiz (cold-start or topic-targeted)
- `quiz [--topic X] [--n 5]` — generate novel practice problems at the boundary of competence
- `gaps` — report current weak topics with citations to specific past exam questions
- `plan --exam <name>` — emit a day-by-day study schedule for a specific upcoming evaluation (e.g. `mid-sem`, `comp`, `quiz-4`); date and scope auto-derived from `course.json`
- `review <topic>` — Socratic walkthrough of a topic, scoped to past-paper question patterns

If invoked with no subcommand: print this argument list and exit.

---

## Persistent state

Every course gets a directory at `~/.claude/teach/<course-slug>/`:

```
~/.claude/teach/<course-slug>/
├── course.json      ← topic graph, exam-weighted question index, syllabus map
├── student.json     ← mastery vector, attempt log, confidence calibration
├── materials/       ← raw inputs (PDFs, slides, papers) — read-only after ingest
└── chunks/          ← structured extractions per source file (one .json per input)
```

### `course.json` schema

All fields except `schema_version`, `course_slug`, `course_name`, and `ingested_at` are optional and may be absent or null when the source material doesn't declare them.

```json
{
  "schema_version": 3,
  "course_slug": "signals-and-systems-2026",
  "course_name": "Signals and Systems",
  "course_code": "ECE F243",
  "institution": "BITS Pilani",
  "academic_term": "Spring 2026",
  "ingested_at": "2026-04-29",

  "course_structure": "modular",
  // Describes how the syllabus is organised.
  // Options: "modular" (numbered modules/lectures) | "chapter" (textbook chapters)
  //          | "unit" (named units or blocks) | "topic_list" (flat, unordered)
  //          | "week" (week-by-week schedule) | "section" (exam-section driven)
  //          | "none" (no declared structure — self-directed or certification prep)

  "domain_prerequisites": [
    {"subject": "calculus", "level": "undergraduate"},
    {"subject": "complex-variables"},
    {"subject": "linear-algebra", "level": "introductory"}
  ],
  // Subject-level prerequisites the student is assumed to have before starting.
  // Not limited to maths — could be "data-structures", "organic-chemistry-basics", etc.

  "evaluation_components": [
    {
      "id": "mid-sem",
      // User-visible identifier — any kebab-case string. Referred to by /teach plan --exam <id>.
      "name": "Mid-Semester Test",
      "type": "written_exam",
      // Component type. Options:
      //   "written_exam"  — invigilated paper (closed or open book)
      //   "quiz"          — short, frequent, low-stakes (tutorial quizzes, Canvas quizzes)
      //   "assignment"    — take-home, problem sets, essays, coding tasks
      //   "project"       — longer multi-week deliverable
      //   "oral"          — viva, oral exam, presentation
      //   "lab"           — practical/experimental assessment
      //   "portfolio"     — collected work (art, essays, code)
      //   "mock"          — practice paper (no grade, pure prep)
      //   "certification" — external certification exam (GATE, CFA, IELTS, AWS, etc.)
      //   "online_mcq"    — auto-graded online multiple-choice
      //   "coursework"    — continuous assessment, participation grade
      "weight": 0.30,
      // Fraction of final grade. null if unweighted or unknown (e.g. mock, certification).
      "date": "2026-03-09",
      // ISO date string, "weekly", "rolling", "tba", or null.
      "duration_min": 90,
      // null if untimed (take-home, project).
      "format": "open_book",
      // How the exam is sat. Options:
      //   "open_book"     — books/notes permitted (may have restrictions — put them in notes)
      //   "closed_book"   — no materials permitted
      //   "take_home"     — submitted outside of exam conditions
      //   "oral"          — spoken response
      //   "lab"           — practical session
      //   "online"        — proctored or unproctored online
      //   "mixed"         — multiple sections with different formats
      //   null            — unknown or not applicable
      "scope": {
        "type": "units",
        // What sub-set of the course is tested. Options:
        //   "all"         — whole course (ignore ids)
        //   "rolling"     — accumulates progressively, or announced per-occurrence
        //   "units"       — specific unit IDs (module numbers, chapter numbers, week numbers)
        //   "topics"      — specific topic IDs from the topics array
        "ids": [1, 2, 3, 4, 5]
        // Omit or null when type is "all" or "rolling".
      },
      "recurrence": null,
      // For repeating components. Options: "weekly" | "biweekly" | "per-unit" | null.
      // e.g. quizzes might be "weekly"; null for one-off exams.
      "best_of": null,
      // Integer — if only N of M occurrences count (e.g. best 6 of 8 quizzes). null otherwise.
      "notes": "Open book: self-written notes + T1 + T2 allowed. No photocopies."
      // Free text for format restrictions, special rules, or prep advice.
    }
  ],

  "topics": [
    {
      "id": "convolution",
      "name": "Convolution (CT and DT)",
      "aliases": ["convolution sum", "convolution integral"],
      "unit_ref": {
        "type": "module",
        // Matches the course_structure vocabulary: "module" | "chapter" | "unit"
        //   | "week" | "section" | "topic" (when structure is "topic_list" — self-referential)
        "id": 4
        // int or string (e.g. "2A", "Week 4", "Chapter 7").
      },
      "prerequisites": ["system-properties", "lti-systems"],
      "prerequisites_source": "handout_modules",
      // "handout_modules" | "user_provided" | "empty"
      "syllabus_weight": 0.12,
      // Fraction of syllabus coverage declared by the course. null if not stated.
      "textbook_refs": ["T1-ch2", "T1-ch9", "R1-2.1 to 2.4"],
      // null or empty list if no textbook is used.
      "exam_frequency": {
        "by_year_own": {"2015": 2, "2018": 1},
        "by_year_external": {"MIT-2011": 3},
        "weighted_own": 0.14,
        "weighted_external": 0.08
      },
      "bloom_distribution": {
        "recall": 0, "understand": 1, "apply": 5,
        "analyze": 2, "evaluate": 0, "create": 0
      }
    }
  ],

  "exam_questions": [
    {
      "id": "Q-2018-midsem-3",
      // Canonical ID: Q-<year_canonical>-<component_id>-<question_number>
      "provenance": "own_course",
      // "own_course" | "external_reference" | "generated"
      "source_file": "EEE_F243_T1_2017_2.pdf",
      "evaluation_component": "mid-sem",
      // Matches evaluation_components[].id, or null if unknown.
      "year_label": "2017-18-sem2",
      // Verbatim from filename or file content — never normalised.
      "year_canonical": 2018,
      // Best-guess 4-digit calendar year of the exam. null if genuinely ambiguous.
      "stem": "...",
      "topics": ["convolution"],
      "marks": 8,
      // Marks awarded for this question. null for MCQ or unknown.
      "marks_max": 8,
      "bloom_level": "apply",
      "common_errors": ["sign error in limits of integration"],
      "mark_scheme_summary": "...",
      // null if no solution file was paired or found.
      "solution_source": "MSE-solution-2017-18.pdf",
      "stem_reconstructed_from_solution": false,
      "weight_discount": 1.0
      // 1.0 for own_course; 0.0–1.0 for external (default 0.5).
    }
  ],

  "orphans": {
    "questions_without_solutions": ["EEE_F243_C_2015_2.pdf"],
    "solutions_without_questions": ["16-17-midsem-solution.pdf", "midsem-answers-2019-20.pdf"]
  },

  "bundle_files": [
    {
      "file": "SAS Test Papers.pdf",
      "page_ranges": [
        {"pages": "1-12", "id": "SAS-paper-1", "year_label": "2014"},
        {"pages": "13-26", "id": "SAS-paper-2", "year_label": "2016"}
      ]
    }
  ],

  "syllabus_exam_mismatches": [
    {
      "topic": "analog-filters",
      "direction": "over_taught",
      // "over_taught" — in syllabus/handout but absent or rare in exams (safe to deprioritise)
      // "under_taught" — appears in exams but not declared in syllabus (cover independently)
      "syllabus_weight": 0.06,
      "exam_frequency_weighted_own": 0.01,
      "note": "Module 11 in handout; rarely appears in past papers"
    }
  ]
}
```

### `student.json` schema

```json
{
  "course_slug": "linear-algebra-2026",
  "sessions_logged": 4,
  "mastery": {
    "eigenvalues": {"score": 0.55, "n_attempts": 8, "last_practiced": "2026-04-27", "errors_by_type": {"conceptual": 1, "procedural": 3, "transfer": 0}}
  },
  "confidence_calibration_delta": -0.18,
  "attempt_log": [
    {"timestamp": "2026-04-27T14:00:00", "topic": "eigenvalues", "question_id": "generated-a3f1", "result": "incorrect", "error_type": "procedural", "self_reported_confidence": 0.8}
  ]
}
```

State is read at the start of every subcommand and rewritten atomically (write-to-temp + rename) at the end. Schema is versioned (`"schema_version": 1`) so future migrations are clean.

---

## Architecture

```
                              [User: /teach <cmd>]
                                       │
                                       ▼
                        ┌────────────────────────────┐
                        │  Load course.json,         │
                        │  student.json from         │
                        │  ~/.claude/teach/<slug>/   │
                        └────────────┬───────────────┘
                                     │
        ┌──────────┬─────────┬───────┼───────┬─────────┬──────────┐
        ▼          ▼         ▼       ▼       ▼         ▼          ▼
    [ingest]  [diagnose]  [quiz]  [gaps]  [plan]  [review]   (no-args help)
        │          │         │       │       │         │
        └──────────┴────┬────┴───────┴───────┴─────────┘
                        │
                  Update student.json
                  (+ course.json on ingest)
                        │
                        ▼
                  Report to user
```

---

## Execution

### Subcommand 1 — `ingest <path>`

Path may point at a directory or a single file. Goal: produce structured chunks, then aggregate into `course.json`.

**Step 1.1** — Discover materials. Walk the path; classify by filename heuristic and a content-sniff fallback:
- `*exam*`, `*paper*`, `*past*`, `Q[0-9]*`, year patterns → exam paper
- `*sol*`, `*answer*`, `*solution*`, `*key*` → solution file (paired separately)
- `*syllabus*`, `*handout*`, `*outline*` → syllabus
- `*slide*`, `*lecture*`, `*.pptx`, `*.key` → slides
- `*.tex`, `*notes*`, prose-heavy PDFs → notes/textbook

**Provenance classification.** Compare each filename's course code / institution prefix against the syllabus's declared course code. Files whose prefix matches → `own_course`. Mismatches (e.g., `MIT6_003*` ingested under a BITS course) → `external_reference`, recorded with a `weight_discount` (default 0.5) so they contribute to topic frequency but don't dominate. Ask the user to confirm any ambiguous cases interactively rather than guessing silently.

**Step 1.1b — File pairing.** Before parsing, attempt to pair every solution file with its question file using fuzzy matching on (course code, year, exam type). Build a pairing table:

```
EEE_F243_1240_T1_2017_2.pdf  ↔  SS_MSE_Solution_2017-18.pdf   (paired)
EEE_F243_1240_C_2015_2.pdf   ↔  (no solution)                 (orphan-question)
(no question)                ↔  16-17 Midsem Solution.pdf     (orphan-solution)
```

Print the pairing table to the user and ask for confirmation/correction before proceeding. Orphan-questions get `mark_scheme_summary: null` and a note. Orphan-solutions are still parsed — the agent must reconstruct the question stem from the solution layout (most worked solutions repeat the question above the solution).

**Step 1.1c — Bundle detection.** For any single PDF whose filename suggests multiple papers (`*Test Papers*`, `*Bundle*`, `*Compilation*`, or any file >50 pages with multiple "Q1" headers detected at content-sniff time), spawn a splitter agent first that emits a list of (page_range, inferred_paper_id) tuples. Each tuple becomes a separate input to Step 1.2.

**Step 1.2** — Parse with Claude (not external libraries). For each file (or bundle-fragment, or paired question+solution pair), spawn one Sonnet agent with this prompt:

```
You are a course-content extraction agent. Read the attached file(s) and emit a JSON document with this exact schema:

{
  "source_file": "<filename>",
  "paired_with": "<solution-filename or null>",
  "content_type": "exam_paper" | "syllabus" | "slides" | "notes" | "solution",
  "provenance": "own_course" | "external_reference",
  "year_label": "<verbatim from filename or file content — never normalised>",
  "year_canonical": <best-guess 4-digit calendar year int, or null if genuinely ambiguous>,
  "evaluation_component_guess": "<name of component if legible from paper header — e.g. 'Mid-Semester', 'Quiz 3', 'Final Exam' — or null>",
  "chunks": [
    {
      "tag": "[DEFINITION] | [THEOREM] | [WORKED_EXAMPLE] | [EXAM_QUESTION] | [MARKING_SCHEME] | [SYLLABUS_TOPIC] | [SYLLABUS_UNIT] | [TEXTBOOK_REF] | [EVALUATION_SCHEME] | [DOMAIN_PREREQUISITE] | [PROSE]",
      "topics": ["topic-id-1", "topic-id-2"],
      "unit_ref": {"type": "<module|chapter|unit|week|section|topic>", "id": <int or string>},
      "text": "...",
      "metadata": {
        "marks": <int or null>,
        "bloom_level": "recall|understand|apply|analyze|evaluate|create",
        "common_errors": ["..."],
        "textbook_refs": ["..."]
      }
    }
  ]
}

For exam papers: every question MUST become its own [EXAM_QUESTION] chunk with marks, bloom_level, and topic tags. If a solution file is paired, populate common_errors and mark_scheme_summary from it. If this is an orphan-solution (no question file), reconstruct the question stem from the solution layout and tag it [EXAM_QUESTION] with a "stem_reconstructed_from_solution: true" flag.

For syllabus / handout: emit one [SYLLABUS_UNIT] chunk per structural block found (module, chapter, week, unit — whatever vocabulary the document uses), preserving declared ordering. Emit [TEXTBOOK_REF] chunks for any textbook or reference list. Emit [EVALUATION_SCHEME] chunks for each graded component with weight, date, format, and scope. Emit [DOMAIN_PREREQUISITE] chunks for any subject-level prerequisites declared (maths, prior courses, assumed knowledge).

For year_canonical: extract the 4-digit calendar year of when the exam was sat. Academic-year ranges (e.g. "2017-18", "16-17") → use the year the exam occurred (Spring → latter year, Autumn/Fall → former year). Semester suffixes (e.g. "_2", "sem2") → infer accordingly. If the naming convention is opaque, leave null and keep year_label verbatim.

Use kebab-case for topic IDs. Merge synonyms aggressively ("eigendecomposition" and "spectral decomposition" → same topic ID "eigenvalues") and list alternatives in aliases.

Output ONLY the JSON. No prose.
```

Write each result to `chunks/<basename>.json`.

**Step 1.3** — Build `course.json`. After all chunks land, run a single aggregator step in-context:
1. Union all topic IDs across chunks; merge aliases. Infer `course_structure` from the dominant `unit_ref.type` seen across `[SYLLABUS_UNIT]` chunks ("module", "chapter", "week", etc.). If none found, set `course_structure: "none"`.
2. For each topic: count exam appearances by year **per provenance** (own_course vs external_reference); derive `exam_frequency.weighted_own` and `weighted_external` separately using **inverse-age weighting** (multiply each appearance by `1/(1 + log(years_old + 1))`). Apply each external reference's `weight_discount` (default 0.5) to its contribution.
3. Build `evaluation_components` from `[EVALUATION_SCHEME]` chunks. For each component's `scope`, derive `ids` from unit_refs or topic IDs mentioned in the handout's coverage declaration. When the handout only states a date (not a topic list), leave `scope.type: "rolling"` and let the user refine.
4. Build `prerequisites` per topic. If `[SYLLABUS_UNIT]` chunks carry ordered IDs, auto-derive: prerequisites for a topic = topics whose `unit_ref.id < this.unit_ref.id`, filtered to direct conceptual neighbors only (not the full transitive chain). Tag `prerequisites_source: "syllabus_units"`. If no unit ordering exists, leave empty and tag `prerequisites_source: "empty"`. Never LLM-guess freehand dependencies.
5. Populate `domain_prerequisites` from any `[DOMAIN_PREREQUISITE]` chunks.
6. Record `orphans` and `bundle_files` (with page_ranges) for transparency.
7. Compute `syllabus_exam_mismatches` using `weighted_own` only — external papers must not influence what is considered over-taught or under-taught in this course. Assign `direction: "over_taught"` or `"under_taught"` per finding.
8. Write `course.json` atomically.

**Step 1.4** — Report:
- Number of materials ingested by type and provenance (own vs external)
- Pairing summary: paired / orphan-question / orphan-solution counts
- Bundle files split, with extracted paper count
- Number of past exam questions extracted (own vs external)
- Top 5 topics by `exam_frequency.weighted_own`
- Evaluation components found, with dates and formats
- Lecture/exam mismatches (high-value coaching signal)
- Math prerequisites declared
- Confirmation that `course.json` is written and ready
- **Explicit warnings:**
  - if no `slides/` or `notes/` chunks were ingested ("Course Content folder is empty — syllabus_exam_mismatches uses handout module table as proxy for lecture intent; signal is weaker without actual lecture slides")
  - if any evaluation component has fewer than 3 own-course past papers ("sparse data — diagnostic confidence will be lower for `<component>`")

### Subcommand 2 — `diagnose [--topic X]`

Cold-start diagnostic OR targeted gap drill.

**Cold-start (no `--topic`):** Generate 15-20 questions spanning all topics, weighted by `exam_frequency.weighted_own` (fall back to `weighted_external` if own-course data is absent), with a deliberate spread across Bloom levels. Refuse to run if `course.json` doesn't exist.

**Targeted (`--topic X`):** Generate 5-8 questions concentrated on topic X, spanning Bloom levels.

For each question:
1. Present problem.
2. Wait for student answer.
3. Ask for self-reported confidence (0-100%) — recorded but not used to update mastery; used only to compute `confidence_calibration_delta`.
4. Evaluate against ground truth. Classify error (if any) as conceptual / procedural / transfer.
5. Do NOT give the solution yet. Continue to next question.

After the full diagnostic: produce a gap report (same format as `gaps`), update `student.json`, and offer to walk through any wrong answers via `/teach review`.

### Subcommand 3 — `quiz [--topic X] [--n N]`

Default: 5 questions. Boundary-of-competence sampling: target topics where `mastery.score` is in `[0.3, 0.7]` (the 30-40% wrong zone). If no `student.json` exists yet, prompt the user to run `/teach diagnose` first; do not silently quiz cold.

**Generation rule:** Problems must be **novel** — derived from past-paper patterns but not direct adaptations. The agent receives the topic's exam questions as exemplars and must generate fresh questions that test the same Bloom level and concept structure. Explicit anti-pattern: "rewrite 2023 Q4 with different numbers."

**Pedagogical posture (hybrid):**
- Attempt 1 wrong → Socratic hint that names the concept being tested but not the next step
- Attempt 2 wrong → release worked solution + flag the error type
- Attempt correct → ask the student to explain *why*, in one sentence (Feynman check); record but do not gate on this

After each question, update `student.json` mastery using a simple EWMA: `new_score = 0.7 * old_score + 0.3 * outcome` (1.0 for first-attempt correct, 0.5 for second-attempt correct, 0.0 for wrong).

### Subcommand 4 — `gaps`

Read `student.json` and `course.json`. Produce a markdown report:

```
# Knowledge Gaps — <course name>
*As of: <date> | Sessions logged: N*

## Critical (high exam frequency, low mastery)
- **eigenvalues** (mastery 0.42, exam weight 0.18) — appears in 2022 Q3, 2023 Q4, 2024 Q2.
  Last 3 attempts: 2 procedural errors (sign in characteristic polynomial), 1 conceptual.
  Recommended review: §4.2 of Strang, then drill `/teach quiz --topic eigenvalues`.

## Moderate (medium frequency or medium mastery)
...

## Low priority (low exam frequency, regardless of mastery)
...

## Lecture/exam mismatches you should know about
- **jordan-normal-form** appears in syllabus but not past 5 exams. Skim only.
- **block-matrices** appears in 2 recent exams but not in lecture handouts. Cover this independently.

## Confidence calibration
You are <over|under>confident by ~<X>% on average.
```

### Subcommand 5 — `plan --exam <component-id>`

`<component-id>` references an entry in `course.json.evaluation_components` (e.g. `mid-sem`, `comp`, `quiz-4`). The skill auto-derives `exam_date`, `format`, and `modules_in_scope` from there. If `--exam` is omitted, default to the next upcoming component by date.

**Cold-start guard:** If `student.json.sessions_logged < 3`, refuse to emit a real plan. Instead emit a "weak prior" labeled clearly:

```
⚠️  WEAK PRIOR — only N diagnostic sessions logged.
The plan below is generic, not personalized. Run /teach diagnose at least
3 times before relying on this output.
```

**Real plan generation:**
1. Resolve in-scope topics from `scope`. `scope.type: "all"` → every topic. `scope.type: "units"` → topics whose `unit_ref.id` is in `scope.ids`. `scope.type: "topics"` → explicit list. `scope.type: "rolling"` → all topics not yet assessed.
2. Compute days available: `exam_date - today`. If `date` is "rolling" or null, ask the user for a target date before continuing.
3. Compute "forgetting pressure" per in-scope topic: `pressure = component_weight * (1 - mastery) * decay(days_since_last_practice)`.
4. **Format-aware Bloom targeting.** Adjust the problem mix based on `format`:
   - `closed_book` → over-weight recall + understand (rote recall and derivation matter)
   - `open_book` → over-weight apply + analyze + evaluate (lookup is free; speed and judgement matter); first deliverable of the plan is a personal one-page reference sheet
   - `take_home` or `assignment` → over-weight evaluate + create (depth and originality rewarded)
   - `oral` → add verbal articulation drills: student must explain each concept aloud in ≤60 seconds
   - `online_mcq` → over-weight recall + understand; add elimination-strategy practice
   - `lab` → focus on procedural accuracy; include worked setup walkthroughs
   - `project` → decompose into milestones; plan maps to milestone deadlines, not daily problem quotas
5. Allocate problem quotas per day, weighted by pressure. Reserve the final 20% of available days for full-length past-paper simulations using **own-course** papers preferentially (external_reference papers are warm-up only, not closing drills).
6. Schedule short-cycle reviews using SM-2 intervals, compressed as the deadline approaches.
7. Output a day-by-day markdown table with: date, in-scope topics, activity (problems / review / simulation / milestone), and format-specific notes.

**Recurring components (quiz, coursework):** When `recurrence` is set, return a rotation calendar aligned to the recurrence cadence instead of a countdown schedule. If `best_of` is set (e.g. best 6 of 8 quizzes), flag that the student has a skip budget and plan accordingly — early quizzes are lower-stakes.

### Subcommand 6 — `review <topic>`

Socratic walkthrough scoped to past-paper patterns for that topic.

1. Pull the 3 most recent past-paper questions on the topic.
2. Present each as a guided dialogue: "What's the first move here? Why?"
3. Use the marking scheme to validate intermediate steps without giving them away.
4. End with a 1-sentence Feynman-style summary the student must produce.

This subcommand is the recovery path after wrong answers in `quiz` or `diagnose` — it does NOT update mastery scores (review is for understanding, quiz is for measurement).

### Step N — Save & report

Every subcommand ends with:
1. Atomic write of any updated state files.
2. Brief user-facing summary (3-5 lines max): what ran, what changed, suggested next command.

---

## Output format

Per-subcommand outputs are markdown printed to the chat. Persistent artifacts live under `~/.claude/teach/<course-slug>/`. Long reports (gaps, plan) are also written to `~/.claude/teach/<course-slug>/reports/<cmd>_<date>.md` so the student has a history.

---

## Design notes

- **Why exam-papers-first:** All 5 stochastic-consensus agents agreed past papers are higher signal than syllabi. Examiners' question distribution diverges from stated syllabus emphasis; treating exams as ground truth and the syllabus as a scope-gate matches what experienced exam coaches do.

- **Why persistent JSON, not SQLite:** v1 prioritizes transparency and editability. JSON files are diffable, hand-correctable, and grep-friendly. SQLite becomes worth it only when query complexity justifies the opacity (probably not before ~50 courses or ~1000 attempts).

- **Why Claude-as-parser at ingest:** Libraries (PyMuPDF, python-pptx) give text; Claude gives meaning. Ingestion runs once per course, so the cost is amortized. The semantic tagging ([DEFINITION], [EXAM_QUESTION], etc.) is what enables every downstream feature — raw chunks would force re-parsing at query time.

- **Why 30-40% error rate target:** Bjork's desirable-difficulty zone. Below this, the student is bored and over-confident; above it, learning collapses into frustration. Operationalizing it as a sampling rule (target topics where `mastery ∈ [0.3, 0.7]`) makes "calibrate to the student" a concrete algorithm, not a vibe.

- **Why withhold-by-default with Socratic escalation:** Stochastic consensus split here (Agent 3 strong, Agent 4 skeptical). The hybrid resolves the tension: students who are gaming for answers get blocked on attempt 1; students who are genuinely stuck get unblocked on attempt 2. The Feynman check on correct answers catches lucky guesses without slowing strong students.

- **Why inverse-age weighting for topic frequency:** Older papers reflect a stable distribution of testable topics — they're the long-run signal. But examiners' style and difficulty drift, so recent papers are over-weighted in *style* but not in *what's tested*. This asymmetry was Agent 1's outlier insight and survives scrutiny.

- **Why refuse cold-start plans:** Agents 3 and 5 both flagged that day-1 personalized plans are fiction. Honesty about this builds trust; printing a confident-looking but uninformed plan trains the student to over-rely on the tool.

- **Why allow prerequisite auto-derivation when the handout has explicit module ordering:** Agent 5 flagged auto-generation as unreliable in general, and that's correct for free-form syllabi. But many real handouts (e.g. BITS course handouts) include a numbered Module table where each module has a stated prerequisite chain ("Module 4 covers convolution, requiring Modules 2-3"). When that structure exists, derivation is deterministic, not inferential — so we use it and tag the source as `prerequisites_source: "handout_modules"`. When it doesn't, we leave the field empty rather than LLM-guessing. This is the post-pressure-test refinement; the original rule was too conservative.

- **Why novel problems, not adapted past papers:** Adapting past papers builds false confidence — the student memorizes the structure. Novel problems testing the same Bloom level on the same concept are the actual training signal.

- **Why `unit_ref` instead of `module_no`:** The BITS pressure-test revealed a hardcoded assumption: all courses use numbered modules. Certification exams (CFA, GATE, IELTS), textbook courses, and week-based curricula each use different structural vocabulary. `unit_ref: {type, id}` carries the vocabulary of the source material explicitly, so downstream logic can handle any of them without special-casing.

- **Why `scope` replaces `modules_in_scope`:** Same reason — the old field assumed integer module numbers. `scope: {type, ids}` works when ids are topic strings, chapter labels, week numbers, or implicit ("all"/"rolling").

- **Why `evaluation_component_guess` is free-text, not an enum:** Hard-coding `"mid-sem" | "comp"` into the parser locked it to Indian university naming conventions. Free text from the paper header ("Mid-Semester Test", "Quiz 3", "Paper 1") is more robust; the aggregator normalises it to a user-chosen `id` when building `course.json`.

- **Why `domain_prerequisites` instead of `math_prerequisites`:** Not all prerequisite knowledge is mathematics. A literature course might list "critical theory basics"; a CompSci course might list "data structures"; a language course might list "intermediate French". `domain_prerequisites: [{subject, level?}]` accommodates all.

- **Why `format` includes `oral`, `lab`, `online`, `mixed`:** The original `open_book | closed_book | take_home` was written for university written exams. Oral vivas, lab practicals, and online MCQ platforms each need different prep strategies — all now handled explicitly in `plan`'s format-aware Bloom targeting.

- **Anti-pattern avoided — chatbot UI:** Agent 3 was right that CLI subcommands fit Claude Code's skill model better than a conversational tutor. Each subcommand is a discrete operation with clear inputs/outputs and a write-back to state.

- **Anti-pattern avoided — emotional state modeling:** Agent 5 self-flagged this as infeasible. Skipping it. Confidence calibration delta is the closest the skill gets, and it uses observable data (stated confidence vs. actual outcome).

- **Known limitation:** Sparse past papers (<3 years) are an open problem. The skill warns the user when this is detected and falls back to syllabus-driven question generation, but the diagnostic signal is genuinely weaker.

- **Known limitation:** Concept-node granularity is unresolved. Default behavior is whatever the syllabus and exam papers naturally use; the user can override by editing `course.json` manually (which is why JSON over SQLite).

---

## Pressure-test findings (2026-04-29, BITS SaS course)

Ran a partial ingest dry-run against `workspaces/ACADS/SaS/` (1 handout + 17 PYQs, no slides). Surfaced 10 design gaps; v1 spec patched to address all of them:

| # | Finding | Patch |
|---|---|---|
| 1 | Real courses have 2+ exams (Mid-Sem + Comp) with different scope, weight, format | `evaluation_components` array in `course.json`; `plan --exam <id>` instead of `--exam-date` |
| 2 | Quizzes (15%) and assignments (10%) matter for grade — skill ignored them | `evaluation_components` covers all graded components; plan supports `--exam quizzes` and `--exam assignments` |
| 3 | External-reference papers (MIT 6.003 in a BITS course) skew topic frequency | `provenance` field per question; separate `weighted_own` vs `weighted_external`; default `weight_discount: 0.5` |
| 4 | Open-book vs closed-book exams need different prep strategies | `format` field per component; format-aware Bloom targeting in `plan`; recommend handwritten formula sheet for open-book |
| 5 | Solutions arrive as separate files, not appended to question papers | Step 1.1b file-pairing pass; orphan-question and orphan-solution handling |
| 6 | Some solutions exist without matching question papers | Orphan-solution agent prompt reconstructs question stem from solution layout |
| 7 | Bundle PDFs contain multiple papers in one file | Step 1.1c bundle-detector + splitter agent before Step 1.2 |
| 8 | Year codes are inconsistent: `2017_2`, `2017-18`, `16-17` all coexist | Store `year_label` verbatim AND `year_canonical` as best-guess int; agent decodes BITS-style sem codes at parse time |
| 9 | Math prerequisites listed in handout (calculus, complex variables, AGP, probability) are real signal | `[MATH_PREREQUISITE]` chunk tag + `math_prerequisites` field at course level; diagnostic can check these as a pre-flight |
| 10 | Module table in handout provides explicit ordering — auto-prerequisite derivation is safe here | Prerequisite rule relaxed: derive when `[SYLLABUS_MODULE]` chunks have ordering; tag source as `handout_modules` |

**Schema v3 changes (generification pass, 2026-04-30):**

| Field changed | v2 (BITS-specific) | v3 (generic) |
|---|---|---|
| `math_prerequisites` | list of strings | `domain_prerequisites: [{subject, level?}]` |
| `module_no` on topic | integer | `unit_ref: {type, id}` |
| `modules_in_scope` on component | int array or "all" | `scope: {type, ids}` |
| `evaluation_component_guess` in parser | hardcoded enum | free-text from paper header |
| `format` on component | open/closed/take_home | + oral, lab, online, mixed |
| `type` on component | (absent) | written_exam, quiz, assignment, project, oral, lab, portfolio, mock, certification, online_mcq, coursework |
| `recurrence` / `best_of` | (absent) | added for rolling components |
| `[MATH_PREREQUISITE]` chunk tag | math-only | `[DOMAIN_PREREQUISITE]` |
| `[SYLLABUS_MODULE]` chunk tag | module-specific | `[SYLLABUS_UNIT]` |
| year decoder in parser prompt | BITS sem-code heuristic | generic calendar-year extraction |
| `syllabus_exam_mismatches` | no direction field | `direction: "over_taught" \| "under_taught"` |
| `bundle_files` | filenames only | includes `page_ranges` array |
| plan MATLAB note | MATLAB-specific | generic component-type-aware prep |

**What stayed unchanged after pressure-test:** the core loop (generate→attempt→evaluate→update→schedule), the JSON-over-SQLite choice, withhold-by-default Socratic posture, 30-40% error-rate target, novel-problem generation.

**Still unverified (would need a full live ingest to test):** robustness of the year-decoder on edge cases; whether the orphan-solution stem reconstruction works on poorly-scanned PDFs; whether the bundle splitter can detect paper boundaries reliably without a TOC.

---

## Related skills

- `stochastic-consensus` — used to design *this* skill; could be invoked manually to debate any contested pedagogical question that comes up during use ("should I prioritize X or Y this week?")
- `vagabond` — orthogonal; cross-domain synthesis, not exam-bound learning
- Future: `skill-flashcard` could complement by handling pure SRS-style retention drills outside an exam window

---

*Drafted: 2026-04-29*
*Promoted: 2026-05-04*
