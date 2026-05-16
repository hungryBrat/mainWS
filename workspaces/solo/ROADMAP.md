# Solo Project Roadmap
*Differential-drive autonomous mobile robot — ROS 2 Humble + Linorobot2*
*Started: 2026-04-28 | Target: 13 weeks | Framework: phase-gated milestones + side quests*

---

## How to use this document

- **Main trunk** — 7 phases, each ending in a gated demo. Work top to bottom.
- **Side quests** — tagged with the earliest phase they unlock. Launch any time you want a change of pace or a quick win. 1–5 days each.
- **Gate criteria** — binary pass/fail. Don't advance until the gate passes.
- **Improvements** — planned upgrades beyond the base spec; pick them up after Phase 3+.

Progress tracking: check boxes as you go. Use `git tag phase-N-complete` when a gate passes.

---

## Stack reference

| Layer | Component |
|---|---|
| SBC | Raspberry Pi 4 (4GB), Ubuntu 22.04 |
| MCU | ESP32-WROOM-32, micro-ROS firmware |
| Motors | JGA25-370 × 3 (characterize, use matched pair) |
| Motor driver | TB6612FNG |
| LIDAR | Slamtec RPLidar A1M8 |
| IMU | BNO055 |
| Chassis | 2WD aluminum, ~200mm wheelbase, 65mm wheels |
| Battery | 3S LiPo 2200mAh, MP1584 buck for Pi rail |
| ROS | ROS 2 Humble, slam_toolbox, Nav2 |
| Reference | Linorobot2 (`linorobot/linorobot2`) |

---

## Phase 0 — Bench Characterization
**Duration:** 1 week | **Branch:** `phase-0/bench`

### Why this phase exists
Hardware decisions made here silently affect every downstream phase. Encoder noise on
shared grounds looks like PID failure. Mismatched motor pairs bias odometry. An
undersized battery browns out the ESP32 mid-navigation. Fix these before assembly.

### Tasks

- [ ] **Power budget spreadsheet**
  - Measure: motor stall current (both motors simultaneously)
  - Measure: RPLidar A1 draw (rated 500mA, verify)
  - Measure: Pi 4 under SLAM load (typically 3-4W)
  - Measure: ESP32 + TB6612 at full PWM
  - Total + 30% headroom → battery minimum mAh
  - Verify MP1584 buck handles Pi peak without dropout

- [ ] **Motor characterization (all 3 motors)**
  - Bench fixture: power motor directly at rated voltage through ammeter
  - Record: stall current, no-load RPM (free-run), back-EMF constant Ke
  - Drive step input, record encoder ticks over time → fit first-order transfer function
  - Output: `bom/motor_model.yaml` with fields: `Kv`, `time_constant_ms`, `deadband_pwm`, `stall_current_A`
  - **Motor matching:** compute gear-ratio-corrected CPR for all 3; select closest pair (±2%)

- [ ] **Encoder signal quality (oscilloscope)**
  - Connect motor + encoder to bench supply and TB6612 simultaneously
  - Probe encoder A/B lines on oscilloscope under motor load
  - Accept if: clean square wave, no ringing >10% Vcc, no glitches at commutation
  - Fail if: add 100nF cap across motor terminals + ferrite bead; re-test
  - If still noisy: add optoisolator (PC817) on each encoder line
  - Document: oscilloscope screenshot to `bom/encoder_signal_quality.png`

- [ ] **IMU smoke test**
  - Wire BNO055 to ESP32 I2C; run a scan sketch; confirm address 0x28
  - Log raw quaternion output for 60 seconds stationary — verify noise floor <0.1°

- [ ] **Star-ground wiring diagram**
  - Draw the ground topology: single star point near battery
  - Motor ground and logic ground meet ONLY at that point
  - Diagram to `bom/ground_topology.png`

### Gate criteria
- [ ] Battery mAh confirmed and ordered/available
- [ ] Matched motor pair selected and documented
- [ ] Encoder signal passes oscilloscope check
- [ ] `motor_model.yaml` committed

### Demo
Oscilloscope screenshot showing clean A/B quadrature under motor load, posted to journal.

---

## Phase 1 — Mechanical Assembly + tf2 Commissioning
**Duration:** 2 weeks | **Branch:** `phase-1/assembly`

### Tasks

- [ ] **Mechanical assembly**
  - Mount motors, encoders, wheels on chassis
  - Mount Pi, ESP32, TB6612, IMU
  - Wire power rails per star-ground diagram
  - Cable management: zip ties, no flopping leads near motor shaft

- [ ] **Flash ESP32 micro-ROS firmware**
  - Clone `linorobot/linorobot2_hardware`
  - Configure `config/custom_robot.h`: wheel radius, encoder CPR (341), wheelbase
  - Flash with `pio run --target upload`
  - Verify micro-ROS agent connects: `ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0`
  - Confirm topics: `/odom`, `/imu/data`, `/joint_states`, `/cmd_vel`

- [ ] **URDF + tf2 tree**
  - Edit Linorobot2 URDF: wheel diameter, wheelbase, lidar mount offset, IMU mount offset
  - Commit URDF as canonical. **Do not edit after this phase.**
  - Run `ros2 run tf2_tools view_frames` — save PDF output to `bom/tf_tree_phase1.pdf`
  - Verify tree: `map → odom → base_link → [base_laser, imu_link, wheel_left, wheel_right]`

- [ ] **Teleop smoke test**
  - `ros2 launch linorobot2_bringup bringup.launch.py`
  - Drive robot with `teleop_twist_keyboard`
  - Verify in RViz2: robot model moves, all frames attached, no dangling transforms

- [ ] **Measure USB-serial round-trip latency**
  - Script: send `/cmd_vel` twist, measure time until `/odom` reflects command
  - Accept if: <5ms average jitter
  - Fail: switch ESP32 connection to hardware UART (GPIO 16/17)

### Gate criteria
- [ ] `ros2 run tf2_tools view_frames` produces clean tree PDF
- [ ] `/odom` publishes at ≥20Hz
- [ ] Teleop drives robot; RViz2 shows all transforms moving

### Demo
RViz2 screen recording: robot model with all frames visible, driven by teleop.

---

## Phase 2 — Closed-Loop Control + Odometry
**Duration:** 2 weeks | **Branch:** `phase-2/control`

### Tasks

- [ ] **PID inner loop tuning**
  - Enable velocity PID in Linorobot2 firmware config
  - Tune Kp: ramp to target velocity, measure overshoot
  - Tune Ki: eliminate steady-state error under load (push on robot gently)
  - Tune Kd: dampen oscillation at velocity step changes
  - Target: <5% steady-state error, no oscillation at cruise speed

- [ ] **Dynamics identification from data**
  - Command step velocity, log `/joint_states` encoder ticks vs time
  - Plot velocity response; identify: dead-band, rise time, time constant
  - Compare to `motor_model.yaml` predictions — update if materially different
  - Document: friction dead-band per wheel, asymmetry between L/R if any

- [ ] **Differential-drive kinematics validation**
  - Verify: commanded wheel velocities → correct chassis twist (forward/turn math)
  - Test: command forward at 0.3 m/s for 5 seconds; measure actual distance

- [ ] **Square trajectory test (the gate)**
  - Command: 1m forward, 90° left, 1m forward, 90° left, 1m forward, 90° left, 1m forward
  - Measure: final position error vs start (tape measure)
  - Record: `/odom` trace in RViz2

- [ ] **(Optional) EKF fusion**
  - Install `robot_localization`; configure `ekf.yaml` for wheel odom + IMU fusion
  - Compare `/odometry/filtered` vs raw `/odom` on square trajectory
  - Improvement in drift: quantify

### Gate criteria
- [ ] 1m commanded = 0.95–1.05m measured
- [ ] 360° commanded rotation error < 5°
- [ ] Square trajectory: robot returns within 10cm of start

### Demo
Video: robot drives 1m square and returns close to start. `/odom` path shown in RViz2.

---

## Phase 3 — SLAM  *(highest-risk phase, owns the schedule buffer)*
**Duration:** 3 weeks | **Branch:** `phase-3/slam`

### Why 3 weeks
SLAM Toolbox parameter convergence on real floor surfaces requires 6–8 iterative
tuning sessions. Carpet vs. tile vs. doorway transitions each invalidate a prior-tuned
parameter set. Do not rush this phase.

### Tasks

- [ ] **RPLidar bring-up**
  - Install `rplidar_ros2`; verify `/scan` at 8Hz in RViz2
  - Check scan range: points visible at 0.2–12m, no blind sectors

- [ ] **slam_toolbox initial setup**
  - Mode: `async` (online_async_launch.py)
  - Initial `slam_params.yaml` from Linorobot2 defaults
  - Drive robot manually through a 20+ sqm space; watch map build in RViz2

- [ ] **Iterative parameter tuning**
  - Key params to tune: `minimum_travel_distance`, `minimum_travel_heading`,
    `loop_search_maximum_distance`, `correlation_search_space_dimension`
  - Accept map if: walls straight, room corners ~90°, no double walls
  - Run 3+ tuning sessions across different surfaces/rooms

- [ ] **Save the map**
  - `ros2 run nav2_map_server map_saver_cli -f maps/room_main`
  - Commit `.pgm` and `.yaml` to `maps/`

- [ ] **Switch to localization mode**
  - `localization_launch.py` with saved map
  - Drive robot from 3 different starting positions; verify AMCL particles converge
  - Convergence criterion: particle cloud diameter < 0.3m within 10 seconds

- [ ] **Heap check**
  - Monitor Pi 4 RAM during 30-min SLAM session: `free -m` every 60s
  - Confirm: no unbounded growth (slam_toolbox known issue in lifelong mode)
  - If growth detected: cap node's memory limit via systemd or restart policy

### Gate criteria
- [ ] Map saved and usable (walls straight, no major artifacts)
- [ ] Localization mode: AMCL converges in <10s from any start
- [ ] No memory leak over 30-minute session

### Demo
Video: robot mapping a full room in real-time (RViz2), then navigating back to origin
using the saved map.

---

## Phase 4 — Computer Vision + Perception  *(optional fork)*
**Duration:** 2 weeks | **Branch:** `phase-4/vision`

> This phase is optional but recommended. You can proceed to Phase 5 (Nav2) without it
> and add vision later. If you skip: document this in journal and continue to Phase 5.

### Tasks

- [ ] **Camera bring-up**
  - Install Pi Camera v2 or USB webcam
  - Launch `v4l2_camera` or `camera_ros` node; verify `/image_raw` in RViz2
  - Confirm: 30fps at 640×480 does not saturate Pi CPU alongside SLAM localization

- [ ] **AprilTag detection (recommended first target)**
  - Install `apriltag_ros`
  - Print Tag36h11 family tags; mount at known locations
  - Verify: `/apriltag/detections` publishes with correct pose

- [ ] **Color blob tracking (alternative quick win)**
  - OpenCV HSV-range blob detector; publish centroid as `/detected_object`
  - Demo: robot turns to face colored cone

- [ ] **Obstacle layer integration (connects to Nav2)**
  - Configure Nav2 costmap `obstacle_layer` to use `/image_raw` or depth proxy
  - Verify: camera-detected obstacle appears in costmap

### Gate criteria
- [ ] Chosen detection pipeline running at ≥20fps
- [ ] Detection result published as ROS 2 topic
- [ ] No CPU regression on SLAM localization

### Demo
Screen recording: live camera feed with bounding box overlaid, robot turns toward detected object.

---

## Phase 5 — Nav2 Integration
**Duration:** 2 weeks | **Branch:** `phase-5/nav2`

### Tasks

- [ ] **Nav2 bringup**
  - Launch `navigation_launch.py` with saved map + AMCL localization
  - Verify costmaps (global + local) visible in RViz2

- [ ] **Nav2 parameter tuning**
  - `DWBLocalPlanner`: tune `max_vel_x`, `min_vel_x`, `acc_lim_x`, `max_rotational_vel`
  - `NavFn`: set inflation radius to match robot footprint + safety margin
  - `RecoveryNode`: enable rotate-in-place and back-up recovery

- [ ] **Waypoint navigation test**
  - Send 10 sequential goals via RViz2 "Nav2 Goal" tool
  - Acceptance: reaches all 10 without human intervention

- [ ] **Dynamic obstacle test**
  - Walk in front of robot during navigation
  - Verify: robot stops, replans, continues

- [ ] **CPU profile under full stack**
  - `htop` during navigation: SLAM localization + Nav2 + camera (if Phase 4 done)
  - Accept if: all Nav2 nodes running at ≥10Hz

### Gate criteria
- [ ] 10 sequential goals reached without intervention
- [ ] Robot recovers from 2 dynamic obstacles
- [ ] No watchdog kills or topic dropouts during 10-minute run

### Demo
Video: 10-goal autonomous navigation run in real room, including one dynamic obstacle.

---

## Phase 6 — Integration + Mission Demo
**Duration:** 1 week + buffer | **Branch:** `phase-6/integration`

### Tasks

- [ ] **End-to-end integration test**
  - Single launch file starts everything: micro-ROS bridge, SLAM localization, Nav2, vision
  - Robot boots → localizes → ready for goals, all within 60 seconds

- [ ] **Patrol mission**
  - Hardcode 4-point patrol loop in a behavior tree or simple Python node
  - Robot completes 3 laps without intervention

- [ ] **ROS 2 bag recording**
  - Record a full patrol run: `ros2 bag record -a`
  - Playback in RViz2; verify all topics intact

- [ ] **Git tags and docs**
  - Tag: `git tag phase-6-complete`
  - Write a 1-page `README.md` at repo root: what the robot does, how to run it

### Gate criteria
- [ ] Full patrol demo runs 3 consecutive times unaided
- [ ] ROS 2 bag playback shows complete run
- [ ] `README.md` exists and is accurate

### Demo
Video: robot executing patrol loop autonomously, narrated.

---

## Side Quests

Mini-projects that branch off the main trunk. Each is self-contained, fun, and
deliverable in 1–5 days. They don't block the main phases — pick them up when you want
a change of pace or a quick win.

---

### SQ-01 — Motor Characterization Dashboard  *(unlocks: Phase 0)*
**Duration:** 2 days | **Type:** tooling + data viz

Build a real-time matplotlib dashboard that plots encoder tick rate, commanded PWM,
and measured velocity simultaneously during PID tuning sessions. Export as a PNG per
tuning run. Makes the dynamics identification in Phase 2 much more intuitive.

**What you'll learn:** serial communication, live matplotlib, data normalization.
**Output:** `tools/motor_dashboard.py`

---

### SQ-02 — Drift Geometry Analysis  *(unlocks: Phase 2)*
**Duration:** 2 days | **Type:** math + visualization

After passing the square-trajectory gate, run 10 square loops and record `/odom`.
Analyze: does drift accumulate linearly or exponentially? Is it primarily rotational
or translational? Plot drift vector as a function of distance traveled. This gives
you an intuitive understanding of differential-drive odometry error models.

**What you'll learn:** error propagation, odometry uncertainty, matplotlib heatmaps.
**Output:** Jupyter notebook + plots in `analysis/odometry_drift/`

---

### SQ-03 — IMU vs. Wheel Odom Comparison  *(unlocks: Phase 2)*
**Duration:** 1 day | **Type:** sensor fusion insight

Record `/odom` (wheel only) and `/imu/data` simultaneously during a sharp turn.
Plot heading estimate from each source over time. Observe divergence during wheel
slip. This is the motivating example for why EKF fusion exists.

**What you'll learn:** sensor fusion intuition, ROS 2 bag analysis.
**Output:** notebook in `analysis/odom_vs_imu/`

---

### SQ-04 — SLAM Parameter Sensitivity Map  *(unlocks: Phase 3)*
**Duration:** 3 days | **Type:** analysis + automation

Write a script that systematically varies 3 slam_toolbox parameters across a grid,
runs a mapping session, and scores map quality (wall straightness via Hough transform,
area coverage). Output a 3D sensitivity surface. Tells you where the parameter space
is flat (robust) vs. steep (brittle).

**What you'll learn:** automated experimentation, image analysis, ROS 2 launch API.
**Output:** `analysis/slam_sensitivity/`

---

### SQ-05 — AprilTag Docking Station  *(unlocks: Phase 4)*
**Duration:** 4 days | **Type:** applied robotics

Place an AprilTag on a "charging dock" (a cardboard box). Write a ROS 2 node that:
1. Detects the tag
2. Computes approach vector
3. Sends Nav2 goals to align robot nose within 5cm of the dock

No physical charging needed — it's a precision docking demo. Very visual payoff.

**What you'll learn:** coordinate frame transforms, tag-relative goal computation, Nav2 programmatic goals.
**Output:** `nodes/docking_node.py`

---

### SQ-06 — Frontier Exploration  *(unlocks: Phase 5)*
**Duration:** 5 days | **Type:** autonomy upgrade

Implement a simple frontier-based exploration node: find the nearest unexplored boundary
in the slam_toolbox map, send it as a Nav2 goal. Robot autonomously maps a room without
human teleoperation. This is one of the most satisfying demos in mobile robotics.

**What you'll learn:** occupancy grid processing, ROS 2 nav2_simple_commander API, costmap queries.
**Output:** `nodes/frontier_explorer.py`

---

### SQ-07 — Battery State Monitor  *(unlocks: Phase 1)*
**Duration:** 1 day | **Type:** embedded + ROS integration

Add a voltage divider (R1/R2 resistor network) to an ESP32 ADC pin reading the LiPo
pack voltage. Publish `/battery_state` (sensor_msgs/BatteryState). Add an RViz2 panel
or terminal overlay showing voltage and estimated state of charge. Prevents the
"robot dies mid-demo" failure mode.

**What you'll learn:** ADC scaling, SoC estimation curve, custom ROS 2 messages.
**Output:** addition to ESP32 firmware + `nodes/battery_monitor.py`

---

### SQ-08 — Custom Behavior Tree Node  *(unlocks: Phase 5)*
**Duration:** 3 days | **Type:** software architecture

Write a custom Nav2 Behavior Tree node that implements "patrol-and-alert": robot
navigates patrol waypoints and, if it detects a colored object (from Phase 4), stops
and publishes an alert event. Teaches you how Nav2's BT plugin system works.

**What you'll learn:** Nav2 BT architecture, C++ plugin system (or Python BT wrapper).
**Output:** `bt_nodes/patrol_alert/`

---

### SQ-09 — Velocity Profiler  *(unlocks: Phase 2)*
**Duration:** 2 days | **Type:** control systems

Implement trapezoidal velocity profiles for cmd_vel: smooth acceleration ramp, cruise,
deceleration ramp. Compare stop-distance and position accuracy vs. step-input commands.
This is the practical implementation of "dynamics" that Phase 2 only brushes.

**What you'll learn:** motion profiles, jerk limiting, ROS 2 action servers.
**Output:** `nodes/velocity_profiler.py`

---

### SQ-10 — ROS 2 Bag Replay Dashboard  *(unlocks: Phase 6)*
**Duration:** 2 days | **Type:** tooling

Build a Foxglove Studio layout (or custom RViz2 config) that replays a bag file and
shows: robot trajectory, LIDAR scan, velocity commands, battery state, and any
detections — all on one screen. Useful for post-mission analysis and debugging.

**What you'll learn:** Foxglove, mcap format, ROS 2 bag indexing.
**Output:** `tools/replay_layout.json`

---

## Recommended Improvements to the Base Plan

These are upgrades beyond the debate spec. Pick them up in order of phase relevance.

### I-01 — robot_localization EKF  *(Phase 2, optional)*
Fuse wheel odometry with IMU using `robot_localization`'s EKF node. Reduces heading
drift by 40–60% during long runs. Low effort with Linorobot2 — it ships a default
`ekf.yaml`. Upgrade the Phase 2 square-trajectory gate to compare fused vs. raw.

### I-02 — Thermal characterization in motor_model.yaml  *(Phase 0)*
JGA25-370 encoder signal quality degrades under sustained load as the motor heats.
Add a 10-minute sustained-run test to Phase 0: record encoder quality at cold and
at thermal steady state. If degradation detected, add a periodic motor cooling pause
to the firmware.

### I-03 — Per-phase git branch + tag discipline  *(all phases)*
`git checkout -b phase-N/description` for every phase. Merge to `main` only when gate
passes. Tag with `git tag phase-N-complete -m "gate: ..."`. Creates a clean history
you can roll back to without losing work.

### I-04 — Gazebo as a parallel integration test environment  *(Phase 3+)*
After Phase 3, build a Gazebo world matching your physical test room. Use it to:
- Test Nav2 parameter changes before deploying to hardware
- Regression-test the full stack after any code change
- Run the frontier explorer (SQ-06) before hardware deployment

### I-05 — Structured phase journal  *(all phases)*
Before each phase: write a 5-line "plan" entry in `journal/YYYY-MM.md`.
After the gate passes: write a 5-line "retro" entry. What took longer than expected?
What was easier? These become the most valuable long-term references.

### I-06 — costmap tuning for tight spaces  *(Phase 5)*
Nav2's default inflation radius is sized for TurtleBot3 (wider robot). Tune
`inflation_radius` and `robot_radius` to your actual chassis dimensions. Robots that
can navigate 80cm-wide hallways become dramatically more useful than ones that can only
navigate open rooms.

### I-07 — Watchdog node  *(Phase 5+)*
Write a simple ROS 2 node that monitors: `/cmd_vel` age (no commands for >5s → stop),
battery voltage threshold (SQ-07 required), and `slam_toolbox` liveliness. If any check
fails, publish `Twist(0,0,0)` and sound buzzer. Prevents runaway robots.

### I-08 — Add 3 encoder CPR verification to Phase 0  *(Phase 0)*
Before trusting `motor_model.yaml`, command exactly 1 full wheel revolution, count raw
encoder ticks. Verify against 341 (expected). If ≠, recalibrate all downstream odom
constants before proceeding. This takes 5 minutes and prevents weeks of phantom drift.

---

## Timeline

```
Week  1      Phase 0  — Bench characterization
Weeks 2-3    Phase 1  — Assembly + tf2
Weeks 4-5    Phase 2  — Control + odometry
Weeks 6-8    Phase 3  — SLAM  (buffer lives here)
Weeks 9-10   Phase 4  — Vision  (optional; can parallel with Phase 5 if CPU permits)
Weeks 11-12  Phase 5  — Nav2
Week  13     Phase 6  — Integration + demo
```

**Principle:** Lock the gates, not the dates. If Phase 3 finishes in 2 weeks, bank
that week as float. If it slips to 4 weeks, you're still on time.

**Confidence:**
- 12 weeks: achievable with prior ROS/Linux experience
- 13 weeks: recommended (this plan)
- 16 weeks: appropriate if learning Linux concurrently or <8 hrs/week

---

## Risk Register

| Risk | Phase | Severity | Mitigation |
|---|---|---|---|
| Encoder signal corruption (motor noise on shared ground) | 0 | High | Oscilloscope check in Phase 0; star-ground; 100nF caps; optoisolator if needed |
| JGA25-370 gear ratio variance ±5% across lots | 0 | High | Order 3, characterize all, use matched pair |
| Wrong motor SKU / incompatible pinout | 0 | High | Buy from verified seller with photo evidence; no substitutions |
| Power budget undersized — ESP32 browns out | 0 | High | Phase 0 spreadsheet; 30% headroom; verify under stall |
| tf2 tree misconfigured | 1 | High | Freeze in Phase 1; `view_frames` PDF as receipt |
| SLAM Toolbox parameter variance per floor type | 3 | Medium | 6-8 tuning sessions budgeted; try carpet + tile + doorway |
| USB-serial latency jitter | 1 | Medium | Measure before Phase 2 gate; UART fallback |
| Pi 4 CPU ceiling under full stack | 4-5 | Medium | Phase sequencing; never run camera + SLAM + Nav2 simultaneously until Phase 5 |
| Linorobot2 BOM substitution breaks integration | all | Medium | BOM locked through Phase 3 |
| SLAM Toolbox memory growth (lifelong mode) | 3 | Low | Heap monitor; restart policy |
| Encoder signal degrades under motor heat | 2 | Low | Thermal probe during sustained test |

---

## Reference Resources

| Domain | Resource |
|---|---|
| Kinematics + dynamics | Siegwart, Nourbakhsh, Scaramuzza — *Introduction to Autonomous Mobile Robots* (2nd ed.) Ch. 2-3 |
| PID control | Brett Beauregard — "Improving the Beginner's PID" blog series |
| SLAM | Hess et al., "Real-Time Loop Closure in 2D LIDAR SLAM," ICRA 2016 |
| Nav2 | Macenski et al., "The Marathon 2," IROS 2020 |
| micro-ROS | Belsare et al., ROSCON 2021 |
| Vision | Redmon & Farhadi, YOLOv3, 2018 |
| Build reference | `linorobot/linorobot2` GitHub repo |
| ROS 2 tutorials | Articulated Robotics YouTube (Josh Newans) |
| Official docs | `docs.ros.org/en/humble` |

---

*This roadmap was generated via 5-agent model-chat debate + Opus synthesis on 2026-04-28.*
*Project name: Solo | Full brief: `mainWS/.tmp/model_chat_amr_project_spec.md`*
