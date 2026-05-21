# Model Chat Brief: Advanced Maker Project from Inventory
*Generated 2026-05-19 | Method: 5-agent debate (Sonnet), 4 rounds + Opus synthesis*

## Final Answer

**Build a ball-on-plate balancer with a phase-space LED scope and a user-triggered polar sand witness — "The Ball Balances. The Sand Remembers."** A steel ball rolls on a 200mm acrylic plate tilted in two axes by SG90/MG996R micro-servos under STM32 PID at 500Hz, with position read by a 4-wire resistive touch panel laminated under the plate. An 8x8 LED matrix beside the plate plots live XY error as a phase-space scope (ephemeral, ~30s persistence via dim fade), and a small polar sand plotter mounted at the rear — triggered manually by a button press, not autonomously — etches the recent trajectory in sand as a physical record. A handheld controller (ESP32 + MPU-6050 + joystick) over nRF24L01 lets a viewer push setpoints in real time, making the system a three-timescale interaction: 1kHz inner loop, 50Hz wireless, 1Hz human intent.

## How We Got Here

The debate started broad: an STM32-driven celestial orrery (S1), inverted pendulum sculpture (S2), self-balancing two-wheeled robot (S3), distributed kinetic balance node (S4), and Sisyphus sand table (S5). Round 2 collapsed the field hard — S2 abandoned the pendulum once MG995's positional-servo limit was named (no torque control = no swing-up), S1 conceded the orrery is open-loop and not display-interactive, and S3's mechanical scrutiny showed the sand table BOM exploded past €60 in rails/belts/magnets. S5's Round 2 hybrid — "balance sculpture that draws" with a CoreXY gantry on a tilt platform — briefly unified everyone, then died in Round 3 when S1 ran the numbers (400-600g moving gantry mass at 5° tilt = 35-50g lateral disturbance exceeding MG995 stiffness, plus a 50Hz servo PWM versus 2-5Hz gantry disturbance bandwidth mismatch). What survived was the **ball-on-plate** core, and the sand idea got demoted from primary mechanism to a separate **witness layer**. Round 4 locked sensing (resistive touch beat IR reflectance 4-to-1), persistence (8x8 phase-space scope beside the plate, unanimous), and the sand witness disposition (3-to-2 yes, but only as user-triggered, neutralizing S2's "kitsch cuckoo clock" objection).

## Locked Spec

- **Project name:** *The Ball Balances, The Sand Remembers* (working title; "Phase-Space Orb" as short form)
- **Primary mechanism:** 200mm acrylic plate, two-axis tilt via 2x MG996R servos in a U-joint gimbal, 20mm steel ball-bearing
- **Position sensing:** **4-wire resistive touch panel**, ~150mm active area, $5-8 panel laminated to underside of acrylic. Reasoning: resistive is dead-simple (ADC + 4 GPIO + alternating drive), 50-100Hz update is plenty for a 1kHz outer-aesthetic system, zero calibration drift, no camera latency, no IR ambient-light failure mode. S1's IR-reflectance proposal (4x TCRT5000 differential) is clever but introduces a calibration coupling between ball position and reflectance (different ball finish = different gain) and only gives ±3mm at 500Hz versus touch's continuous analog readout. The 4-to-1 vote tracks the engineering: touch is the boring correct answer.
- **Ephemeral visualization:** **8x8 MAX7219 LED matrix beside the plate (not under it)**, plotting X-error vs Y-error in phase space at 20Hz with brightness decay over ~30s. S2's "oscilloscope aesthetic" framing is correct — the matrix is too coarse to be a position indicator under the plate (3mm pitch, ~50 pixels across the plate = Minecraft sprite + LED bloom through acrylic) but is *exactly right* as a phase-portrait scope showing the limit cycle of the controller.
- **Persistent visualization (sand witness):** **Yes, but user-triggered, not autonomous.** Small polar sand plotter at rear: 1x NEMA17 (radial arm rotation) + 1x MG995 (needle lift/drop) over a ~100mm circular sand tray. Pressing a button on the handheld dumps the last N seconds of (X,Y) trajectory to the plotter, which replays it in atan2/sqrt polar coordinates. Reasoning: S2 and S3's objection that an always-on sand plotter is a kitsch second project is correct *if* it runs continuously. As a user-invoked "snapshot of the fight," it becomes a deliberate gesture — the viewer chooses what gets remembered. This preserves S1/S4/S5's "the sand remembers" emotional payload while answering S2's "answers a question nobody asked" critique. It also gives S3 their polar plotter without a second calibration loop fighting the main one.
- **Compute split:**
  - **STM32 Blue Pill** — inner PID loop @ 1kHz, servo PWM, touch panel ADC. Deterministic, hardware timers, no OS jitter.
  - **ESP32 #1 (base station)** — 8x8 phase-space rendering, OLED telemetry, 16x2 LCD state ("BAL / FALL / REPLAY"), 7-seg error magnitude, nRF24L01 RX, triggers NEMA17/MG995 sand replay on button event.
  - **ESP32 #2 (handheld)** — MPU-6050 gesture tilt, HW-504 joystick, nRF24L01 TX, replay button, OLED feedback ("CONNECTED / SETPOINT X,Y").
  - **Arduino Uno** — sand-tray stepper driver controller (offloaded from ESP32 to avoid stepping-jitter contaminating the wireless/display loop). Receives target (r, θ) over UART from ESP32, drives A4988 stepper + servo.
- **Wireless:** **nRF24L01** (not WiFi/BLE). S3 originally argued WiFi was more reliable; S4 correctly countered that the *value* of nRF here is deterministic latency (~5-10ms) versus WiFi's stack jitter (10-200ms+). For a control loop that's *partly* in the wireless link, determinism beats raw throughput.
- **Power:** 3S LiPo → LM2596 buck → 5V rail for servos and logic. 18650 pack reserved for handheld (2S + boost or single + boost to 3.3V). Keep stepper power separate (LM2596 #2 at 8-9V from 3S directly to A4988 VMOT) to isolate motor noise from ADC.
- **Aesthetic form factor:** Hardwood (walnut or oak) base, exposed acrylic plate with visible servo linkage (no enclosure), 8x8 matrix on a small angled riser to the right, sand tray inset at the rear-left, OLED + 16x2 + 7-seg arranged as an instrument cluster on the front face. Cable management through a brass-tube spine. The aesthetic statement is *instrument*, not *toy* — exposed mechanism, clean typography on the LCDs, no RGB.

## Inventory Mapping

| Component | Role | Notes |
|---|---|---|
| STM32 Blue Pill | Inner PID loop, servo PWM, touch ADC | Deterministic timers — non-negotiable for 1kHz loop |
| ESP32 #1 | Base station: display rendering, sand trigger, nRF RX | Drives 8x8, OLED, 16x2, 7-seg |
| ESP32 #2 | Handheld: gesture + joystick, nRF TX | Battery-powered |
| Arduino Uno | Dedicated stepper controller for sand plotter | Isolated from main loop; UART slave |
| MPU-6050 #1 | Handheld gesture tilt → setpoint deltas | On ESP32 #2 |
| MPU-6050 #2 | Plate orientation feedback (optional inner-loop trim) | Mounted to plate underside; redundant with touch but useful for fault detection ("plate not where it should be") |
| HC-SR04 #1 | UNUSED in primary build | Reserve for v2 "approach detection" — viewer-presence wake |
| HC-SR04 #2 | UNUSED | Spare |
| DHT22/11 #1 | UNUSED | Not relevant to balance task |
| DHT22/11 #2 | UNUSED | Spare |
| BME280 | UNUSED | Save for weather-station spinoff |
| KY-037 (sound) | UNUSED in primary; optional "clap to trigger sand replay" | S4's R1 idea, deferred |
| Photoresistors x5 | UNUSED | S2's R3 radial-photo sensing rejected in favor of resistive touch |
| OLED #1 (handheld) | Connection state, setpoint readout | I2C |
| OLED #2 (base) | PID gains, loop frequency, ball lost timer | I2C |
| 8x8 LED matrix | **Phase-space scope (XY error)** with brightness decay | MAX7219 SPI; the centerpiece visualization |
| 16x2 LCD | System state: BALANCING / FALLEN / REPLAYING / TUNING | I2C backpack |
| 7-seg x4 | Error magnitude in mm, or settle-time counter | Direct or via shift register |
| L298N | UNUSED in primary | Overkill for servos; reserve for v2 motorized base rotation |
| TB6612FNG | UNUSED | Was for self-balancing bot path |
| Stepper drivers (A4988) x4 | 1x drives sand-plotter NEMA17; 3x spare | Only 1 needed |
| MG995 servo | Sand-plotter needle lift/drop | Positional servo, perfect for binary up/down |
| Coreless motors x8 | UNUSED | S1 correctly flagged coreless ≠ torque without gearbox |
| nRF24L01 #1 | Handheld TX | SPI |
| nRF24L01 #2 | Base RX | SPI |
| BLE module | UNUSED in primary | Optional phone-app PID tuning in v2 |
| 18650 x6 | 2-3 for handheld pack; rest spare | Spot-weld a 2S pack with BMS |
| 3S LiPo | Main base power | Through LM2596 |
| LM2596 buck | 5V logic + servo rails (x2: one logic, one VMOT) | $1 modules |
| VGA camera | UNUSED | S1 proposed it for vision sensing; killed for latency. Save for a separate vision project |
| 555 timer x5 | UNUSED | Reserve for PWM/debug; not needed |
| HW-504 joystick #1 | Handheld setpoint nudge | Analog X/Y to ESP32 ADC |
| HW-504 joystick #2 | UNUSED | Spare |
| Shields, passives | Level shifting, pull-ups, decoupling | As needed |

**Unused tally:** HC-SR04 x2, DHT x2, BME280, KY-037, photoresistors x5, L298N, TB6612FNG, 3 stepper drivers, 8 coreless motors, BLE, VGA cam, 555s x5, 1 joystick. That's a lot of unused inventory — but the alternative was a Frankenstein build trying to justify every component, and the agents correctly killed that path repeatedly. A focused project that doesn't use everything beats a kitchen-sink project that uses it all.

## What Needs Acquisition

- **4-wire resistive touch panel, ~150-180mm** — $5-12 on AliExpress/eBay
- **2x MG996R servos** (inventory has 1x MG995; need 2 matched for the gimbal, MG996R is the metal-gear upgrade) — $8-12
- **200mm acrylic disc, 3mm thickness** — $5 (or laser-cut from scrap)
- **20mm steel ball-bearing** — $2
- **1x NEMA17 stepper** for sand plotter (inventory has A4988 drivers but no actual steppers listed in the prompt — confirm; if you have any 28BYJ-48s, those work too with reduced precision) — $12 if needed, $0 if NEMA17 is on-hand
- **Small sand tray, ~100mm, with fine kinetic/play sand** — $5
- **Walnut or oak board for base, ~300x200x20mm** — $15-25
- **Brass tubing for cable spine** — $5
- **2S 18650 holder + BMS for handheld** — $5

**Total acquisition: ~$60-90.** Well under "a new project" threshold.

## Build Phases

**Phase 1 — Bench balance (Week 1, ~10 hours):** STM32 + 2x servos + resistive touch panel on a breadboard. Plate flat on table, gimbal not yet assembled. Goal: read touch coordinates cleanly, drive servos open-loop from joystick, close a basic P loop. Pass criterion: ball stays on plate for 30s with hand-tuned P only.

**Phase 2 — PID + gimbal mechanical (Week 2, ~12 hours):** Build the wooden base + acrylic gimbal + servo linkage. Tune full PID (suggest: P=0.4, I=0.05, D=0.15 as starting point, expect 4-8 iterations). Add MPU-6050 #2 under plate for fault detection. Pass criterion: ball settles to center within 3 seconds from a 30mm offset, holds within ±5mm.

**Phase 3 — Phase-space scope + display cluster (Week 3, ~8 hours):** Wire ESP32 base, 8x8 MAX7219, OLED, 16x2, 7-seg. UART link STM32→ESP32 streaming (x_err, y_err) at 100Hz. Implement 8x8 phase trace with exponential decay. Pass criterion: phase portrait visibly shows the limit cycle of the controller; you can see the system *think*.

**Phase 4 — Wireless handheld (Week 4, ~8 hours):** ESP32 #2 + MPU-6050 + joystick + nRF24L01 + OLED + battery. Pair with base, stream setpoint deltas. Pass criterion: tilting the handheld visibly moves the ball; replay button registers on base.

**Phase 5 — Sand witness (Week 5, ~10 hours):** Arduino Uno + A4988 + NEMA17 + MG995 needle + sand tray. UART command from ESP32 with replay buffer (last 30s of trajectory in polar form). Tune needle drop force, sand depth, angular resolution. Pass criterion: pressing the handheld button etches the last balance attempt visibly in sand within 20-30s.

**Phase 6 — Finish & integration (Week 6, ~8 hours):** Hardwood base finishing, cable routing through brass spine, instrument-cluster panel mounting, photo/video documentation. Burn-in test: 30 minutes continuous operation.

**Total: ~56 hours over 6 weekends.** S3's "2 days realistic" was for the bench prototype only; the finished display-worthy build is a 6-weekend project.

## Key Risks & Mitigations

1. **Servo PWM jitter at 50Hz limits inner loop bandwidth.** *Mitigation:* Accept it — 50Hz is enough for ball-on-plate (settling time ~1s; you're not landing a Falcon 9). If unhappy, swap MG996R for a digital servo (333Hz PWM, ~$15) post-Phase 2. Don't pre-optimize.

2. **Resistive touch readings noisy near edges / at low pressure.** *Mitigation:* Heavy steel ball (≥20mm bearing) ensures consistent pressure. Median filter (k=5) on ADC samples. Calibrate corners at startup.

3. **Sand plotter mechanical backlash on direction reversal.** *Mitigation:* This is a known polar-plotter issue. Pre-load the radial arm with a light spring, or accept ~0.5mm backlash as acceptable for a 100mm tray (0.5%). Don't go fancy with belt-driven precision — it's an art object, not a CMM.

4. **nRF24L01 dropouts under USB-3 / 2.4GHz Wi-Fi noise.** *Mitigation:* Use the PA+LNA version with external antenna for the base. Implement watchdog on base side: if no packet for 200ms, hold last setpoint, fade 8x8 to amber.

5. **Two-MCU debugging (STM32 + ESP32 + Uno) becomes a nightmare.** *Mitigation:* Strict UART protocol with checksums, logged to a debug OLED line. Build Phase 1 entirely on STM32 alone — don't add MCUs until each subsystem works standalone.

6. **Power: servos brown out the STM32 on transients.** *Mitigation:* Separate LM2596 buck rails for servo and logic. 1000µF cap across each servo's V+. Common ground star topology.

7. **The "demo lasts 60 seconds then it's shelved" problem (S2's R3 critique of the bot).** *Mitigation:* The handheld + sand replay is the answer. It's an *interactive instrument*, not a static demo — the viewer participates, gets a souvenir-in-sand, has a reason to come back. This is the project's strongest design choice.

## Consensus Points

- **Ball-on-plate is the right primary mechanism** (4-of-5 by R3, 5-of-5 by R4). Reasoning: it uses the MPU-6050 + STM32 + small-servo combo natively, fails gracefully (ball rolls off, you put it back), and is genuinely advanced control work. S3's evidence: it's the textbook nonlinear control demo for a reason.
- **8x8 LED matrix as phase-space scope, beside the plate, not under it** (unanimous by R4). Reasoning: S2's "Minecraft sprite" argument killed the under-plate position-display use case; S4 reframed it as the *scope* showing the controller's emotional life. This was the debate's best concrete insight.
- **STM32 owns the inner loop, ESP32 owns display and wireless** (unanimous). Reasoning: hardware timers + no OS = deterministic PID; ESP32's WiFi/BLE stack overhead doesn't belong anywhere near the control loop.
- **Resistive touch beats every alternative for position sensing** (4-of-5; S1 dissents for IR). Reasoning above.
- **Kill the VGA camera path** (unanimous by R3, after S2's "Linux + vision = two projects" landed). The camera is for a different project.
- **Kill the coreless motors** (S1 R2 + nobody defended them). They're for tiny drones, not torque applications.

## Open Disagreements & Resolutions

**Disagreement 1: Resistive touch vs. IR reflectance (4 vs. 1).**
S1's IR-reflectance proposal is genuinely well-engineered (differential pairs, signed XY, 500Hz, ±3mm). But it has three problems S1 underweighted: (a) ambient-light failure modes — the project is meant to be displayed in varied lighting; (b) ball-finish dependency — a fingerprint-smudged ball reads differently than a clean one; (c) calibration drift over time as the IR LEDs age. Resistive touch has none of these. **Resolution: resistive touch.** S1's IR design becomes a v2 backup if touch panels have unexpected failure modes in practice.

**Disagreement 2: Sand witness — yes or no, autonomous or triggered (3-of-5 yes, with the right framing).**
S2 and S3 correctly identified that a continuously-running sand plotter is a second project that calibration-couples with the main loop and adds a kitsch element. S1, S4, S5 correctly identified that the sand layer carries the project's emotional payload — it makes the system *remember*, which is what elevates it from "balance demo" to "instrument." **Resolution: include it, but only as a user-triggered replay of the last 30s of trajectory.** This removes S2's "answers a question nobody asked" objection (the user *is* asking) and S3's "second calibration loop" objection (it doesn't run during balance — replay is sequential, not concurrent). Both dissenters' technical points are honored; both supporters' aesthetic vision is preserved.

**Disagreement 3: WiFi vs. nRF24L01 for the handheld link.**
S3 originally favored WiFi for reliability; S4 corrected this — the relevant property is *latency determinism*, not throughput or range. **Resolution: nRF24L01.** S3's reliability concern is addressed via watchdog + last-setpoint-hold, not by switching to a higher-latency stack.

**Disagreement 4: One MCU or three.**
S3 advocated for a single Pi Zero 2W running Python PID. Everyone else (correctly) said no: Python's GC pause + Linux scheduler jitter is incompatible with a 1kHz loop, and the Pi adds boot time + SD card corruption risk to a display piece. **Resolution: STM32 + ESP32 + Uno split.** S3's "keep it simple" instinct is honored elsewhere (single sensing modality, no camera, no vision stack).

## Emergent Insights

These ideas didn't exist in any single agent's Round 1 position — only the debate produced them:

1. **Triple-timescale interaction as the project's intellectual core.** 1kHz inner PID (STM32, machine time) × 50Hz wireless (nRF, network time) × ~1Hz human gesture (handheld, embodied time). Each timescale is *visible*: the 8x8 phase-space shows the 1kHz loop's character; the handheld OLED shows the 50Hz link's freshness; the sand replay compresses the 1Hz human session into a single artifact. **No single agent had this in R1.** S4 surfaced "emergent behavior lives in the mismatch between timescales" in R2, and the spec converged around making each timescale legible in a different visualization.

2. **Phase-space scope > position display.** All five agents initially thought the 8x8 should display position (under the plate, as a heatmap). S2's "Minecraft sprite" critique + S4's reframe to phase portrait turned a weakness (8x8 is too coarse for position) into the project's strongest aesthetic hook (8x8 is *perfect* for a 2D scope showing the controller's limit cycle). This is the kind of insight a single architect would have missed.

3. **User-triggered persistence as the fix for the "kitsch cuckoo clock" problem.** S2's objection that an autonomous sand layer is decorative noise was correct, but the answer wasn't to remove the sand — it was to make the *user* the trigger. This converts the sand from background ambience into a deliberate gesture (the viewer chooses what gets remembered). The 3-vs-2 split dissolved when this framing emerged in R4.

4. **The inventory is a constraint, not a goal.** R1 saw multiple proposals trying to use every component (S4's R1 distributed node used MPU + sound + DHT + photo + nRF + stepper + servo + matrix — *eight* subsystems). R4's locked spec uses ~40% of the inventory. The debate's biggest meta-lesson: **a focused project that doesn't use everything beats a kitchen-sink project that uses it all.**

## Debate Quality Assessment

- **Strongest contributor: S2 (devil's advocate).** Killed the most bad ideas (rovers, RGB clocks, AI-on-VGA, autonomous sand, under-plate 8x8, MG995 swing-up, hybrid CoreXY-on-tilt). Was wrong about the sand witness in the end, but only because S4 found a framing S2 hadn't considered. The debate would have produced a much worse spec without S2's pressure.
- **Biggest shift: S4.** Round 1 proposed a 2-node distributed sculpture using almost every sensor. By R4 was locked onto the cleanest possible design with one explicit dual-persistence aesthetic. R2's "emergent behavior lives in the mismatch between timescales" was the single most generative line in the transcript.
- **Most productive exchange: S1 R3 ↔ S5 R2.** S5 proposed the CoreXY-on-tilt hybrid; S1 killed it with a back-of-envelope force calculation (35-50g lateral disturbance vs MG995 stiffness, 50Hz PWM vs 2-5Hz disturbance bandwidth). That single numeric critique saved the project from a beautiful idea that wouldn't have worked, and freed the energy to converge on ball-on-plate.
- **Weakest moment:** S3 R4's proposal to run PID in Python on a Pi Zero 2W. Right "keep it simple" instinct, wrong target — the simplification belongs at the sensor and visualization level, not the control loop. Everyone moved past this quickly.

---

**Build this. It's a six-weekend project, ~$80 in acquisition, uses ~40% of your inventory thoughtfully, and ends with an exposed-mechanism hardwood-based instrument that balances a steel ball, plots its own controller's emotional life on an LED scope, and etches a souvenir in sand on demand. It's not a demo. It's an instrument.**
