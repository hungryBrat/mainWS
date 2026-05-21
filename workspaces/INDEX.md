# Workspace Registry

Living index of all child workspaces. Update when creating, archiving, or changing status.

## Active

| Name | Path | Purpose | Started |
|------|------|---------|---------|
| Solo | `workspaces/solo/` | Differential-drive autonomous mobile robot — ROS 2 Humble + Linorobot2 | 2026-04-28 |
| Outreach Pipeline | `workspaces/outreach-pipeline/` | India early-stage robotics/embedded/hardware/edge-ML contact discovery + outreach | 2026-05-18 |

## Paused

| Name | Path | Purpose | Last active |
|------|------|---------|------------|
| — | — | — | — |

## Archived

| Name | Path | Purpose | Archived |
|------|------|---------|---------|
| — | — | — | — |

---

## Creating a new child workspace

1. Create the directory (sibling to mainWS or under `workspaces/`)
2. Add a `CLAUDE.md` with: purpose, how it relates to mainWS, key conventions
3. Add an entry in this file under **Active**
4. If it's a git repo: `git init` and make an initial commit
