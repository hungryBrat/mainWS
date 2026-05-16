# Solo Workspace

Autonomous Mobile Robot project. Differential-drive, ROS 2 Humble, Linorobot2 base.

## Purpose

Build a physically functional AMR from scratch, covering kinematics, dynamics, control,
SLAM, navigation, and perception in a phase-gated structure. Each phase ships a demo.

## Relation to mainWS

Spawned from mainWS after a 5-agent model-chat spec session (2026-04-28).
Full debate brief is at `mainWS/.tmp/model_chat_amr_project_spec.md`.

## Key files

- `ROADMAP.md` — the authoritative plan; phases, gates, side quests, risks
- `journal/` — per-month build logs (create as needed)
- `bom/` — BOM spreadsheets, motor characterization data, power budget

## Stack

- ROS 2 Humble on Ubuntu 22.04 (Pi 4)
- Linorobot2 reference implementation
- ESP32 micro-ROS firmware
- JGA25-370 motors, RPLidar A1, BNO055 IMU

## Conventions

- One git branch per phase. Merge to main only when gate criteria pass.
- Tag each passed gate: `git tag phase-N-complete`
- Video every demo before merging.
- Never edit the tf2 tree after Phase 1 commissioning.
- BOM lock is in effect through Phase 3 — no hardware substitutions without a re-validation sprint.
