# Skills Index

## Stable (deployed)

| Skill | Trigger | Purpose |
|-------|---------|---------|
| **teach** | `/teach` | Exam-focused master tutor: ingests course materials + past papers, diagnoses gaps from performance, generates novel problems at 30-40% error rate, emits date-anchored study plans |

## Drafts

| Skill | File | Status | Purpose |
|-------|------|--------|---------|
| **skill-vagabond** | [drafts/skill-vagabond.md](drafts/skill-vagabond.md) | draft (older flat-file copy already at `~/.claude/skills/vagabond.md` — needs restructure into `vagabond/SKILL.md` to register) | **Unified pipeline**: parallel lattice (8×Sonnet) → convergence scoring → walk (Sonnet) → shadow dialectic (Opus) |
| skill-stochastic | [drafts/skill-stochastic.md](drafts/skill-stochastic.md) | draft (flat-file copy at `~/.claude/skills/stochastic-consensus.md` — needs restructure to register) | Spawn N agents with same prompt, aggregate by consensus |
| skill-chat | [drafts/skill-chat.md](drafts/skill-chat.md) | draft (flat-file copy at `~/.claude/skills/model-chat.md` — needs restructure to register) | Multi-model debate room with synthesizer |
| skill-promote | [drafts/skill-promote.md](drafts/skill-promote.md) | draft | Promote a draft skill from drafts/ to `~/.claude/skills/` |
| skill-journal | [drafts/skill-journal.md](drafts/skill-journal.md) | draft | Append timestamped entry to monthly workspace journal |
| skill-phase-gate | [drafts/skill-phase-gate.md](drafts/skill-phase-gate.md) | draft | Surface current active phase from a ROADMAP.md |

## Skill families

**vagabond** — Cross-domain synthesis and insight generation.
- `skill-vagabond` = **unified pipeline** (use this for full synthesis): parallel lattice → convergence scoring → walk → shadow
- `skill-vagabond-walk` = standalone walk (fast, narrative, when you need a single traversal)
- `skill-vagabond-lattice` = standalone lattice (when you want only the domain map)
- `skill-vagabond-shadow` = standalone shadow (when you have a landing and need dialectic)
