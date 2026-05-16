# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# mainWS — Mission Control

This is the primary working workspace. All major initiatives, child workspaces, and skill development originate here. It is a **coordination layer, not a codebase** — there is nothing to build, test, or lint.

## Directory structure

```
mainWS/
├── CLAUDE.md              ← you are here
├── skills/                ← custom Claude Code skill development
│   ├── drafts/            ← work-in-progress skills
│   ├── INDEX.md           ← living registry of all skills (update on every add/promote)
│   ├── improvements.md    ← backlog of improvements for existing skills
│   └── TEMPLATE.md        ← canonical skill scaffolding — read before drafting
├── workspaces/            ← child workspace registry
│   └── INDEX.md           ← living index of all child spaces (update when creating/archiving)
├── journal/               ← cross-session log (YYYY-MM.md, one file per month)
├── scratch/               ← ephemeral exploration — nothing here is permanent
└── .tmp/                  ← skill output landing zone (research briefs, vagabond outputs, etc.)
```

## Skill lifecycle

Skills live in `skills/drafts/` during development and are promoted to `~/.claude/skills/` when stable.

**Drafting:** Copy `skills/TEMPLATE.md`. The frontmatter `description` field is what Claude reads to decide when to invoke the skill — keep it under 300 characters and name trigger phrases explicitly.

**Promotion:** Claude Code only loads custom skills from the directory layout `~/.claude/skills/<name>/SKILL.md` — flat `.md` files at `~/.claude/skills/` are silently ignored. To promote: `mkdir -p ~/.claude/skills/<name>` then copy the draft to `~/.claude/skills/<name>/SKILL.md`, fill in "Deployed to" and "Promoted" in the skill's footer, and update `skills/INDEX.md` to move it from Drafts → Stable. `<name>` must match the `name:` field in the skill's frontmatter — that's what `/<name>` resolves to. New skills hot-load mid-session; no Claude Code restart required.

**Deployed skills** (at `~/.claude/skills/<name>/SKILL.md`):
- `stochastic-consensus` — 5-agent parallel research poll + Opus synthesis. Outputs to `.tmp/research_debate_<slug>.md`.
- `vagabond` — cross-domain synthesis engine (lattice → walk → shadow). Outputs to `.tmp/vagabond_<slug>.md`.
- `model-chat` — structured multi-model debate.
- `teach` — exam-focused master tutor (ingest / diagnose / quiz / gaps / plan / review).

**Draft skills** (in `skills/drafts/`):
- `skill-vagabond` — unified pipeline (newer revision of deployed `vagabond`)
- `skill-stochastic` — newer revision of deployed `stochastic-consensus`
- `skill-chat` — newer revision of deployed `model-chat`
- `skill-phase-gate`, `skill-journal`, `skill-promote`

**Invocation:** A deployed skill is triggered either by its slash command (`/stochastic-consensus`, `/vagabond`, `/teach`, `/model-chat`) or by a natural-language phrase listed in its frontmatter `description`. When iterating on a draft, read the description field first — that's the contract.

**Cost awareness:** Multi-agent skills are not cheap. `vagabond` spawns 8 Sonnet + 1 Sonnet walk + 1 Opus dialectic. `stochastic-consensus` spawns 5 Sonnet + 1 Opus. Don't invoke these casually — they're for high-stakes synthesis, not quick lookups.

**Improvements backlog:** When a deployed skill misbehaves or a refinement idea surfaces during use, append it to `skills/improvements.md` with the skill name and a one-line rationale. Don't patch deployed skills mid-session unless the bug is urgent — batch fixes through the backlog.

## Child workspaces

Registered in `workspaces/INDEX.md`. Creating a new one: make the directory, add a `CLAUDE.md` with purpose + stack + conventions, register in INDEX.md.

**Active workspaces:**
- `workspaces/solo/` — Autonomous Mobile Robot (ROS 2 Humble, Linorobot2, ESP32). See `solo/ROADMAP.md` for phases and gate criteria.
- `workspaces/ACADS/` — Academic study spaces. Currently contains `SaS/` (Signals and Systems, ECE F243, BITS Pilani Spring 2026) and `ConSys/`. *Not yet in INDEX.md — add when expanding.*

## Working conventions

- **Ask before creating** new directories or child workspaces.
- **Register everything** — new skills in `skills/INDEX.md`, new workspaces in `workspaces/INDEX.md`.
- **Journal entries** go in `journal/YYYY-MM.md`. One file per month. Freeform.
- **`.tmp/` is write-only ephemeral** — skills write outputs here; do not treat these files as permanent.
- **`scratch/` is empty by default** — clean it periodically; nothing in it should persist.
- **Memory is in use** — consult `~/.claude/projects/-home-agush-mainWS/memory/MEMORY.md` at session start when context matters.

## Collaboration style

- Direct and opinionated. Recommend, don't hedge.
- Skip trailing summaries — the diff is readable.
- When suggesting workspace structure changes, ask before creating.
