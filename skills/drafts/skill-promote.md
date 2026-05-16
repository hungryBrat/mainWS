---
name: skill-promote
description: >
  Promote a draft skill from skills/drafts/ to ~/.claude/skills/ so Claude Code
  can invoke it. Updates status, deployed-to path, and promoted date in both the
  draft and the deployed copy. Triggers on "promote skill", "deploy skill",
  "promote this skill", or /skill-promote.
disable-model-invocation: false
argument-hint: "[skill-name or path/to/draft.md] (optional: --dry-run)"
---

# Skill: Skill Promote

**Status:** draft
**Deployed to:** `~/.claude/skills/skill-promote.md` *(fill in when promoted)*
**Cost:** negligible — file reads + writes, no agents

---

## Purpose

Promoting a skill from draft to deployed requires: finding the file, updating three
metadata fields, creating `~/.claude/skills/` if it doesn't exist, writing the deployed
copy, and updating the draft in place. That's enough steps to get wrong or skip. This
skill makes promotion a single command with a validation pass before anything is written.

---

## When to use vs. alternatives

| Use this skill when... | Use X instead when... |
|---|---|
| A draft skill is ready to deploy | You want to edit a skill — use Read/Edit directly |
| Creating `~/.claude/skills/` for the first time | You want to demote/archive a skill — edit manually |
| Updating an already-deployed skill from its draft | — |

---

## Arguments

- `$0` — Skill name or draft file path. *(required)*
  Accepted forms:
  - `journal` → resolves to `skills/drafts/skill-journal.md`
  - `skill-journal` → same
  - `skills/drafts/skill-journal.md` → used directly
  - Absolute path → used directly
- `$1` — `--dry-run`: show what would be done without writing anything. *(optional)*

If invoked with no arguments: list all files in `skills/drafts/` with their current
status line, and ask the user which to promote.

---

## Execution

### Step 1 — Resolve draft file path

1. If `$0` is a readable file path (relative or absolute), use it directly.
2. Otherwise, treat `$0` as a skill name:
   - Strip a leading `skill-` prefix if present to get `{name}`
   - Search in order:
     a. Walk up from cwd to find a `skills/drafts/` directory
     b. Try `skills/drafts/skill-{name}.md`
     c. Try `skills/drafts/{name}.md`
3. If no file found after all attempts: report the search locations tried and stop.
   Ask the user to provide the explicit path.

### Step 2 — Read and validate the draft

Read the draft file. Validate:

- [ ] Frontmatter block exists (starts with `---`, has `name:` field)
- [ ] `name:` field is non-empty — this becomes the deployed filename
- [ ] `description:` field is non-empty
- [ ] File contains `**Status:**` line
- [ ] File contains `*Drafted:` line

If any check fails: report which field is missing and stop. Do not proceed to write.

If `**Status:**` already contains `deployed`: warn —
> "This draft is already marked deployed. Re-promoting will overwrite `~/.claude/skills/{name}.md` with the current draft content. Continue? (y/n)"
> Wait for user confirmation before proceeding.

Extract:
- `skill_name` — value of the `name:` frontmatter field (e.g., `journal`)
- `deployed_path` — `~/.claude/skills/{skill_name}.md`
- `today` — current date from `currentDate` context (YYYY-MM-DD format)

### Step 3 — Dry-run report (if `$1 = --dry-run`)

Print what would happen and stop:

```
Dry run — no files written.

Draft:    skills/drafts/skill-{name}.md
Deployed: ~/.claude/skills/{name}.md  (will be created)

Changes to apply:
  **Status:** draft  →  **Status:** deployed
  **Deployed to:** ...  →  **Deployed to:** ~/.claude/skills/{name}.md
  *Promoted: (fill in...)  →  *Promoted: {today}*

~/.claude/skills/ exists: yes / no (will be created)
```

### Step 4 — Build promoted content

Take the full draft file content and apply three substitutions:

1. `**Status:** draft` → `**Status:** deployed`
   - If the line reads `**Status:** draft | stable | deployed`, replace the whole
     options string with just `deployed`

2. `**Deployed to:** \`~/.claude/skills/[name].md\` *(fill in when promoted)*`
   → `**Deployed to:** \`~/.claude/skills/{skill_name}.md\``

3. `*Promoted: YYYY-MM-DD (fill in when deployed)*`
   → `*Promoted: {today}*`
   - Also matches `*Promoted: (fill in when deployed)*` without the date placeholder

Apply substitutions to produce `promoted_content`. This is what gets written to
both the deployed path and back to the draft file.

### Step 5 — Write deployed file

1. If `~/.claude/skills/` does not exist: create it with Bash (`mkdir -p ~/.claude/skills/`).
2. Write `promoted_content` to `~/.claude/skills/{skill_name}.md` using the Write tool.

### Step 6 — Update draft in place

Write `promoted_content` back to the original draft file path using the Write tool.

This keeps the draft in sync: its status now reads `deployed` and the promoted date is
filled in. The draft is the source of truth; the deployed copy is a snapshot.

### Step 7 — Report

```
Promoted: {skill_name}

  Draft:    skills/drafts/skill-{name}.md  (updated)
  Deployed: ~/.claude/skills/{skill_name}.md  (written)
  Date:     {today}

Claude Code will pick up the skill in the next session.
To use it immediately in this session, restart Claude Code.
```

If `~/.claude/skills/` was created for the first time, add:
```
  Note: Created ~/.claude/skills/ for the first time.
```

---

## Output format

Inline confirmation only. No separate output file.

---

## Design notes

- **Why update the draft in place, not just write to deployed:** The draft is the
  canonical source. If only the deployed copy reflects the promoted date, the two fall
  out of sync. Keeping both updated means the draft directory tells you the full history
  of each skill without cross-referencing.
- **Why extract `skill_name` from frontmatter `name:`, not from filename:** The
  frontmatter `name:` is what Claude Code reads to identify the skill. A filename like
  `skill-journal.md` could differ from `name: journal`. The deployed filename must match
  the frontmatter `name:` exactly, or the skill won't be discoverable.
- **Why validate before writing:** Partial writes are worse than no write. Validating
  first ensures the deployed skill is well-formed. A skill without a `description:` will
  silently never trigger.
- **Why warn on re-promotion rather than block:** Re-promoting is the correct workflow
  for updating a deployed skill after editing the draft. Blocking it would require
  a separate "update" command. A warning with confirmation is the right friction level.
- **Anti-pattern avoided:** Reading the deployed file to check current state before
  overwriting. The draft is the source of truth — always promote from draft, never
  from the deployed copy.
- **Known limitation:** Substitution in Step 4 assumes the exact string formats from
  the TEMPLATE.md. If a skill was written with different phrasing for the Status or
  Promoted lines, the substitution may not match. In that case, report what was found
  and ask the user to fix the draft's metadata fields before re-running.

---

## Related skills

- `journal` — log the promotion as a journal entry after promoting
- `phase-gate` — check project phase status before deciding what to promote next

---

*Drafted: 2026-04-28*
*Promoted: (fill in when deployed)*
