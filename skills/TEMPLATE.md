---
name: skill-name
description: >
  One sentence Claude uses to decide whether to invoke this skill.
  Be specific: name the trigger phrases or conditions. This line is
  read at every session — keep it under 300 characters.
disable-model-invocation: false
argument-hint: "[required-arg] (optional: [optional-arg, default: X])"
---

# Skill: [Human-readable name]

**Status:** draft | stable | deployed
**Deployed to:** `~/.claude/skills/[name]/SKILL.md` *(fill in when promoted)*
**Cost:** *e.g., 1 Sonnet call | 5 Sonnet + 1 Opus | negligible*

---

## Purpose

What problem does this solve? Why does it exist as a skill rather than a plain prompt or
a one-off instruction? One paragraph max. If you can't answer this, the skill shouldn't exist.

## When to use vs. alternatives

| Use this skill when... | Use X instead when... |
|---|---|
| ... | ... |

*(Fill in competing skills or approaches. Omit section if no meaningful alternatives.)*

---

## Arguments

- `$0` — Description of the required argument. *(required)*
- `$1` — Description of optional argument. Default: `value`. *(optional)*
- `$2` — Description of optional argument. Default: `value`. *(optional)*

If invoked with no arguments: describe fallback behavior or error.

---

## Architecture

*(Include for multi-step or multi-agent skills. Delete for simple single-step skills.)*

```
[ASCII diagram of data flow, agent topology, or execution sequence]

Example:
         [Input]
            |
     ┌──────┴──────┐
     v             v
  [Agent A]    [Agent B]    ← parallel
     |             |
     └──────┬──────┘
            v
      [Synthesizer]
            |
       [Output.md]
```

---

## Execution

### Step 1 — [Name of step]

*(What Claude does. Be specific: which tools, which agents, what parameters.)*

If this step uses an Agent tool, include the exact prompt template:

```
[Agent prompt template]

TASK: {task}
CONTEXT: {context}

INSTRUCTIONS:
1. ...
2. ...

OUTPUT FORMAT:
...
```

### Step 2 — [Name of step]

*(Repeat pattern for each step. Number them. Multi-agent steps should note whether
agents run in parallel or sequentially.)*

### Step N — Save & report

1. Write output to `[path/to/output_file]`
2. Report to user:
   - What was done
   - Key finding or result
   - Output file path

---

## Output format

Describe what the user sees at the end. If the skill writes a file, specify:
- File path pattern (e.g., `.tmp/[skill-name]_[slug].md`)
- File structure (headers, sections)

---

## Design notes

*(Explain non-obvious choices. Why this architecture? What anti-patterns did you avoid?
What tradeoffs were made? This section is what separates a skill from a recipe.)*

- **Why [choice]:** reason
- **Anti-pattern avoided:** description
- **Known limitation:** description and workaround

---

## Related skills

- `skill-a` — composes with this skill when X
- `skill-b` — use instead when Y

---

*Drafted: YYYY-MM-DD*
*Promoted: YYYY-MM-DD (fill in when deployed)*
