---
name: journal
description: >
  Append a timestamped entry to the current workspace's monthly journal file
  (journal/YYYY-MM.md). Creates the file and directory if they don't exist.
  Triggers on "log this", "journal entry", "add to journal", "note this", or /journal.
disable-model-invocation: false
argument-hint: "[note content] (optional: [workspace path override])"
---

# Skill: Journal

**Status:** draft
**Deployed to:** `~/.claude/skills/journal.md` *(fill in when promoted)*
**Cost:** negligible — 1-2 file ops, no agents

---

## Purpose

You have a `journal/YYYY-MM.md` convention in every workspace but writing to it requires
navigating to the right directory, finding the right file, and manually formatting an
entry. That friction means it doesn't get used. This skill collapses the entire workflow
into one command: provide a note, it lands in the right file with a timestamp, done.

It exists as a skill because it has non-trivial logic (workspace detection, date
resolution, file creation, entry formatting) that shouldn't be re-explained every time.

## When to use vs. alternatives

| Use this skill when... | Use X instead when... |
|---|---|
| Logging a decision, observation, or phase result | Writing a long structured document — use Write directly |
| End-of-session notes | Planning next steps in detail — use the ROADMAP.md |
| Recording a phase gate pass or demo result | You need to search past entries — Read the journal file |

---

## Arguments

- `$0` — The note content to log. *(required)* Can be a sentence, a paragraph, or a
  bulleted list — any freeform text. Claude will not modify the content.
- `$1` — Workspace path override. *(optional)* Default: auto-detected (see Step 1).
  Use when invoking from outside the target workspace directory.

If invoked with no arguments: prompt the user — "What would you like to log?"

---

## Execution

### Step 1 — Detect workspace root

Walk up the directory tree from the current working directory looking for a `CLAUDE.md`
file. The directory containing the first `CLAUDE.md` found is the workspace root.

If no `CLAUDE.md` is found after walking to `/home`: use the current working directory
as the workspace root and note this in the output.

If `$1` is provided: use that path as the workspace root directly (skip detection).

### Step 2 — Resolve journal file path

- Journal directory: `<workspace_root>/journal/`
- File name: `<workspace_root>/journal/YYYY-MM.md` where YYYY-MM is the current
  year and month (from the `currentDate` context variable).

If the `journal/` directory does not exist: create it with Bash (`mkdir -p`).

If the journal file does not exist: create it with a minimal header:

```markdown
# Journal — YYYY-MM

*[Workspace name from CLAUDE.md h1, if readable — else workspace directory name]*

---
```

### Step 3 — Format the entry

Entry format:

```markdown
### YYYY-MM-DD

[note content exactly as provided in $0 — no rewording, no summarizing]

---
```

Rules:
- The date header is `### YYYY-MM-DD` using the current date.
- If another entry with the same date already exists in the file, append under a
  continuation marker instead of a new `###` header:

  ```markdown
  *(continued)*

  [note content]

  ---
  ```

- Do not reword, clean up, or summarize the user's note. Append verbatim.
- One blank line before the `###` header (after the previous `---`).

### Step 4 — Append and confirm

Use the Edit tool to append the formatted entry to the end of the journal file.

Report to user:
- File written to (full path)
- The date header used
- First line of the note (as confirmation the right content was written)
- Total entry count in the file (count of `###` headers) — gives a sense of how
  active the journal is

Example confirmation:

```
Logged to journal/2026-04.md

### 2026-04-28
"Phase 0 bench test complete. Motor pair A+C matched at 341 CPR ±0.5%..."

(Entry 3 in this month's journal)
```

---

## Output format

Inline confirmation only. No separate output file — the journal file IS the output.

---

## Design notes

- **Why YYYY-MM.md (monthly files, not daily or single):** Daily files fragment the log
  too much for a project journal — you can't scan a week at a glance. A single file
  grows unwieldy over months. Monthly is the right granularity for project work.
- **Why verbatim content, no rewriting:** The journal is a personal record, not a
  summary. Rewriting would introduce latency and lose the exact phrasing the user
  chose. If the user wants a summary, they can ask for one separately.
- **Why detect workspace via CLAUDE.md, not cwd:** The user may invoke this skill from
  a subdirectory (`bom/`, `nodes/`). Walking up to find CLAUDE.md correctly resolves to
  the workspace root regardless of where the terminal is.
- **Why create journal/ automatically:** Requiring the user to create the directory
  first adds friction that defeats the purpose of the skill. Silent creation is
  appropriate here — the user explicitly asked to journal.
- **Anti-pattern avoided:** Writing to a `.tmp/` file or a fixed path. Journal entries
  are permanent records, not ephemeral output. They belong in the workspace's `journal/`
  under version control.
- **Known limitation:** If invoked from outside any workspace (e.g., bare home
  directory), workspace detection falls back to cwd. The entry still gets written — just
  note the fallback in the confirmation so the user can move it if needed.

---

## Related skills

- `phase-gate` — use before journaling to get the current phase status, then log the
  result with `journal`
- `skill-promote` — after journaling a skill draft as "ready", promote it

---

*Drafted: 2026-04-28*
*Promoted: (fill in when deployed)*
