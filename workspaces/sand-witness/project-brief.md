# The Ball Balances. The Sand Remembers.

**A kinetic instrument that turns thirty seconds of attention into an object you can put in your pocket.**

---

## Project

**Name:** *The Sand Witness* (working title — the machine refers to itself as "the witness" in any text it ever shows)

**Tagline:** *The ball balances. The sand remembers.*

**Pitch.** A 200mm acrylic disc, tilted by two small servos a thousand times a second, holds a steel ball at a point in space. Beside it, an 8×8 LED matrix paints the ball's recent past as a comet of fading light. Behind it, a brass disc the size of a coaster slowly rotates while a needle traces the polar memory of what just happened in the black volcanic sand laid over the brass. A handheld brass controller, the weight of a lighter, lets a stranger tilt the world and — without ever being told they are drawing — leave a drawing. They take it home in their shirt pocket. There are no menus, no labels, no boot screens, no error codes. There is a ball, a trail, and a disc you can lift out.

---

## The Three Public Modes

Three words. Anyone can speak them. Mode is changed by a single press of the unlabeled left button on the handheld; the 16×2 LCD on the base shows the current word.

### DRIFT — the resting state

> *Internally: "Reverie." Firmware state: `IDLE_DRIFT`.*

The machine has nothing to prove. The ball traces a slow Lissajous 3:2 at 15% of full excursion. Servo movement sits at the threshold of audibility. The LED matrix glows a warm amber comet-tail at 30% brightness, holding the ghost of the last trajectory and fading over twenty minutes toward dark. After two minutes with no ball on the plate, the LEDs breathe — one slow pulse every five seconds, like the machine sleeping. The sand disc does not move. The 7-segment shows the boot-lock time (the machine's "mood" — see Boot Ritual).

- **Operator does:** nothing. Watches. Their eye finds the trail before they find the ball.
- **LED does:** warm-amber comet-tail of the ball's path, 1/30 fade per frame.
- **Sand does:** nothing. Holds the last drawing.

### GUIDE — the operator takes the world

> *Internally: "Tempest" if pushed hard, "Pulse" if held gently. Firmware state: `TRAJECTORY` or `GESTURE_LIVE`.*

The operator tilts the handheld. The plate follows. This is the primary interaction — water-in-a-bowl intuition. No instruction is needed; the OLED on the handheld shows a single line of feedback the first time the handheld moves: `tilt me`. After that, silence.

Internally the handheld MPU's gravity vector becomes a setpoint offset, clamped to ±25mm from plate center, low-pass filtered at 5Hz. The LED matrix shifts from warm amber to cool white as RMS error grows — temperature reads as stability. Push the handheld harder than the controller can keep up with and the matrix goes full-bright cool white: this is *Tempest*, the adversarial sub-mode, and the sand-witness arms itself for a scar (see Failure).

- **Operator does:** tilts the handheld. Joystick is available as a silent expert option (precise setpoint nudge, ±5mm, used by people who already know what they're doing).
- **LED does:** live phase-space comet of position-error X vs velocity-estimate Y. Color temperature tracks stability.
- **Sand does:** nothing yet — waiting.

### ENGRAVE — the world remembers

> *Internally: "Witness." Firmware state: `DRAWING`.*

Hold the right button on the handheld for two seconds to prime; release to commit. A single piezo click marks the moment the needle touches the disc — like a shutter. The disc advances 15° (one of 24 sectors), and over the next 30 seconds the needle traces the polar position trajectory of the ball, while the machine keeps balancing as if nothing important is happening. When the trace finishes, the sand wiper smooths the trail behind it — the disc holds *one* drawing at a time. Lift the disc out by its friction-fit brass collar. It fits in a shirt pocket. The underside is stamped with the date and a small logo.

- **Operator does:** holds, releases, waits 30 seconds, picks up the disc. The "oh" moment is when they realize the groove *is* the 30 seconds they just spent.
- **LED does:** continues whatever it was doing in Guide/Drift — engraving is a parallel event, not a foreground one.
- **Sand does:** draws.

---

## Hidden & Expert Modes

These are not in the LCD vocabulary. They are summoned by chords, taps, or hardware events. None of them ever show "INIT" or "GAIN" on any display.

| Mode | Trigger | What it does |
|---|---|---|
| **Storm** *(Tempest, full power)* | Push handheld past clamp limits for >2s | LEDs go full cool-white, plate accepts maximum aggressive setpoints. Next engrave is a radial scar burst, not a position trace. |
| **Echo** | Both shoulder buttons + tilt | Records 30s of handheld gesture (MPU quaternion at 100Hz, downsampled to 50Hz). Release: plate replays your hand. Engrave during replay = sand draws the memory of your hand, not the ball. |
| **Sketchpad** *(Etch-a-sketch)* | Both shoulder buttons held, joystick active | Direct joystick → setpoint mapping, no trajectory layer. Hidden because it looks like a toy and breaks the spell. For builders. |
| **Step Response** | Konami-like chord on handheld (↑↑↓↓ joystick + both buttons) | Cycles Kp/Ki/Kd presets, fires step inputs, settling time on 7-seg. Engineering demo only. |
| **Sysid / Auto-tune** | First cold boot per ball-mass profile, or chord on handheld | PRBS injection 60s per axis, second-order TF estimate on STM32, gains derived. To the user this looks like the boot ritual taking slightly longer. Result shown only as the boot lock time. |
| **Chaos** | Konami code (↑↑↓↓←→←→ + both buttons) | Lorenz attractor x-z projection as setpoint trajectory, ρ re-seeded every 10s. Easter egg; lives in firmware, undocumented on the LCD. |

**Mode-naming rule.** The public LCD vocabulary is **Drift / Guide / Engrave**. The artist names (Reverie, Tempest, Pulse, Fossil, Echo) live in firmware logs and in the build documentation — they are what *we* call the moods. The engineering names (Free-balance, Trajectory, PRBS-ID, etc.) live only in source code and the engineering demo script.

---

## Trajectory Library

Two tiers. Both live on the base ESP32 SPIFFS, but only the public set is reachable from the LCD.

### Public Set (5 trajectories, joystick cycles through them in Guide-trajectory sub-mode)

1. **Lissajous 3:2** (default) — the signature curve, the one in the marketing photo
2. **Rose r = cos(5θ/3)** — five-petal, asymmetric, recognizable
3. **Figure-8** — the calm one
4. **Inward spiral** — collapse to a point
5. **Outward spiral** — birth from a point

These render as float32 50Hz arrays, normalized to plate coordinates, compiled into the base ESP32 image as C arrays. Switching trajectories morphs over 20 seconds (linear interpolation in normalized sample space) — the ball never jumps.

### Expert Set (hidden, accessed by both-shoulder-buttons + joystick-up to enter library mode)

- Lissajous a/b ∈ {1..5} × {1..5} grid
- Rose r = cos(nθ) for n ∈ {2..7}
- Harmonograph (4-pendulum, damping 0.995)
- Lorenz x-z RK4 projection
- Fourier-epicycle SVG approximator (load SVG via WiFi UI in IDLE)
- User-recorded joystick traces (20Hz capture, 250-sample ring buffer, named via 6-character OLED entry, max 30 stored)
- User-recorded handheld gestures (Echo mode captures, same storage)

---

## Sand Witness Behavior

The sand draws **position** as the primary semantic — this is what naive users intuit and what makes the souvenir legible ("that loop in the sand is the loop I made"). The error signal is preserved as a secondary semantic, accessible by a different trigger so technically literate visitors can summon it.

### Trigger semantics (resolved from P4's tap/hold/double-tap, mapped to handheld right button)

| Gesture | Action | What gets drawn |
|---|---|---|
| **Tap** (<300ms) | Replay last 30 seconds | Polar **position** trajectory of the ball — the souvenir drawing |
| **Hold** (2s prime + release) | Live engrave next 30s | Polar **position** trajectory, drawn in real time as it happens |
| **Double-tap** (<500ms gap) | Diagnostic engrave | Polar **error** trace: angle = time, radius = \|error\|. Three concentric error rings layered on one disc = under/critical/over-damped — the canonical engineering artifact |
| **Hold after a deliberate double-drop** | Scar mode | Radial burst pattern. The disc remembers that you tried to break it. |

**Mapping rules (position mode):** sample at 50Hz from the touch panel, 30 seconds = 1500 samples. Angle along the sector = time (linear). Radius from sector centerline = |position - plate_center|, scaled so 30mm = needle's full radial sweep. The needle's lateral micro-oscillation frequency is modulated by instantaneous |error| — clean control gives clean arcs, chatter gives ragged lines. This is the technical depth visible as texture, not as numbers.

**Disc geometry:** 120mm brushed brass, 24 sectors × 15°, one sector per engraving = 24 engravings per disc before overwrite. A trailing silicone wiper rides 5° behind the needle and smooths the previous pass — the disc holds *exactly one* drawing at a time. The 16×2 LCD shows the engraving counter (e.g. `ENGRAVE 07/24`).

**Sand:** Icelandic black volcanic sand, 0.1–0.3mm grain. Black against brushed brass reads as shadow. Brass ages over months; needle-track patterns oxidize into ghost rings — the machine accumulates a patina of its own history beneath the sand. The disc lifts on a friction-fit brass collar. It ships with a small brass tin of replacement sand (40 resets per tin).

---

## Handheld Controller

> *A watchmaker's tool, not a game controller.*

**Form factor.** Camera-grip silhouette, palm-sized, ~180–200g — the weight of a Zippo, not a phone. No lanyard hole.

**Shell.** Solid cast brass, hand-finished, oil-darkened in the recesses. Will develop a patina from skin oil over months.

**Controls.**
- Delrin joystick crown, machined, no markings
- Two unlabeled brass buttons: left engraved with a dot, right engraved with a ring (dot = mode, ring = engrave/witness)
- Two shoulder buttons under the index finger curve, brass, recessed
- Smoked acrylic window over the 0.96" OLED — the OLED is invisible when off

**Inside.** ESP32, MPU-6050 #2 (for tilt-primary control and Echo gesture capture), HW-504 joystick, nRF24L01, 18650 cell with TP4056. No power LED. The brass shell warms slightly when running; that's the only "on" indication.

**Interaction model — resolved.**

> **Tilt is primary.** Joystick is the silent expert option.

Picking up the handheld and tilting it changes the plate. This is the water-in-a-bowl primary interaction, the one a child or stranger discovers in three seconds. The joystick exists for people who already understand what they're holding: ±5mm precise setpoint nudges in Guide mode, trajectory cycling in trajectory sub-mode, library navigation in expert mode. The joystick is never *necessary*. The OLED writes `tilt me` exactly once, on first motion after boot, and then never again.

**Sleep.** Handheld sleeps after 5 minutes of stillness. Wakes on motion (MPU interrupt). nRF link re-syncs in <100ms.

---

## Display Hierarchy

Four surfaces. Each has exactly one job. Discipline here is what separates the instrument from a tech demo.

| Surface | Role | What it shows | What it never shows |
|---|---|---|---|
| **8×8 LED matrix** | The poetry. The reveal. | Comet-tail of ball position (Drift), phase portrait position-err vs velocity-est (Guide), color temperature = stability (warm amber stable → cool white chaotic) | RGB rainbows. Status. Battery. |
| **7-segment** | The machine's mood. | Boot lock time in ms (set at boot, never changes during a session — this *is* the session's mood). In Step Response mode, settling time. | Anything else, ever. |
| **16×2 LCD** | The vocabulary. | Line 1: current mode word (`DRIFT` / `GUIDE` / `ENGRAVE`). Line 2: current trajectory name + engraving counter (`Lissajous 3:2  07/24`). | Errors. Init text. Gain values. |
| **0.96" OLED (handheld)** | Operator-input feedback only. | The single `tilt me` hint on first motion. During expert chord entry, shows the chord being recognized. Otherwise blank. | Telemetry. Status. Battery. |

The fundamental display rule: **if a naive user would see the word "INIT", "OFFSET", "GAIN", or "ERROR" on any display, the design has failed.**

---

## Calibration & Boot Ritual

Boot is a three-second performance, not a process. Total time from power-on to user-ready: ≤5 seconds (warm boot) or ≤90 seconds (cold first-boot per ball-mass profile, which includes silent PRBS sysid).

### The ritual (warm boot)

1. **t=0**: power applied. Servos sweep once, full range, in 1.5s — the machine yawns.
2. **t=0**: LEDs fill outward from center in a 2s expanding ring.
3. **t=1.5**: MPU-6050 #1 finds the gravity vector (100ms, silent).
4. **t=1.6**: PID engages. The 7-segment counts milliseconds upward.
5. **t=X**: when RMS error stays below 2mm for 1 continuous second, the 7-segment **freezes** at value X. That number is the **mood for the session.**

A fast boot (X ≈ 800ms) means the machine is sharp today. A slow boot (X ≈ 2400ms) means something is sticky — maybe room temperature, maybe a fingerprint on the touch panel, maybe the ball is heavier than the calibrated profile expects. The user will, after enough sessions, develop intuitions about what makes the number low. That intuition *is* the relationship with the instrument.

### Cold boot (first time, or after ball-mass profile change)

Same ritual visually, but during the silent PRBS injection phase the LED matrix shows a slow swirling pattern (not the comet). The 7-segment counts up; it just takes longer. The user sees a longer boot; they do not see "PRBS_INJECT_AXIS_X". The boot lock time at the end of cold boot becomes the new mood-baseline for that ball.

### Persistence

Calibration data (servo offsets, gravity vector, identified plant parameters, derived PID gains, ball-mass profile checksum) lives in the last 4KB flash page of the STM32. Warm boot validates checksum and skips PRBS. Cold boot is forced when checksum mismatches or when the user holds the engrave button during power-on.

---

## Failure & Recovery

Failure is graceful. The machine never panics. There are no error codes.

| Event | Response |
|---|---|
| **Ball rolls off plate** | LED matrix pulses one slow ring outward, fading to dark. 7-segment continues showing session mood. State machine → `BALL_LOST`. Servos park to level. The machine waits, patient, indefinite. |
| **Ball returns to plate** | PID re-engages within 200ms of touch-panel detection. LEDs resume from wherever they were before. No fanfare. |
| **Two deliberate ball-drops within 10s** | The machine remembers. Next engrave is a **radial scar burst** instead of a position trace. The disc bears the mark of having been provoked. |
| **nRF link loss >150ms** | State → `SAFE_PARK`. Servos hold last setpoint. LEDs dim 50%. Handheld OLED briefly shows `…`. Link resumes silently when handheld returns. |
| **Servo stall / overcurrent (detected via current sense on buck)** | `FAULT`. Servos disable. LEDs fade to dark over 3s. 7-segment shows `---`. Only path out is power-cycle. |
| **Touch panel reads invalid (no ball OR ghost touch)** | 100ms debounce. If still invalid: → `BALL_LOST`. |

No beeps. No flashing red. The single piezo click is reserved for one event only: needle-touches-disc at the start of an engrave.

---

## Aesthetic Specification

The contract is: **everything visible is a material decision. Nothing is hidden behind plastic.**

### Materials

- **Base:** walnut, 18mm, oil finish (Danish or tung), brass standoffs, exposed mechanism
- **Plate:** 5mm flame-polished clear acrylic, 200mm diameter, no smoked tint
- **Sand disc:** 120mm brushed brass, oxidizable, mounted on NEMA17 via friction-fit brass collar
- **Sand:** Icelandic black volcanic sand, 0.1–0.3mm
- **Needle:** fine chisel tip (steel or brass), draws a line, not a groove
- **Handheld:** cast brass shell, delrin joystick crown, smoked acrylic OLED window
- **3D-printed parts:** PLA or PETG, sanded smooth, painted satin black — no raw matte black plastic visible anywhere
- **Acrylic on enclosure:** none. There is no enclosure.

### Light contract

- The 8×8 LED matrix is the **only** light source on the instrument.
- Warm amber (≈2700K equivalent) at rest. Cool white (≈5500K equivalent) under stress. Temperature reads as stability.
- 30% brightness in Drift, scales to 100% in Storm.
- 1/30 per frame fade, 30 frames to zero — the comet-tail equation.
- **No power LED. No status LED. No RGB rainbows. No "ready" indicator.** The light has one job.
- Ambient light sensor on base ESP32: dims the entire matrix proportionally to room brightness. Night = embers. Day = visible. The user never knows this is happening.

### Sound contract

- Servos are **embraced**, not muffled. Their movement is part of the music.
- One piezo click, ever: needle-touches-disc, at the start of an engrave. Like a shutter.
- No beeps, no startup chimes, no error tones.

### The kill list

- RGB rainbow effects
- Power LEDs of any kind
- Finger-joint laser-cut enclosures
- Button labels
- Raw matte black PLA showing on any visible surface
- Cooling fans
- Smoked acrylic on the main plate
- The word "INIT", "READY", "ERROR", or "v1.0" on any display
- Boot screens longer than 5 seconds
- Lanyard holes
- Stickers

---

## Firmware Architecture

### State machine (STM32 owns canonical state)

```
BOOT → SELF_TEST → CALIBRATE → IDLE_DRIFT
                                  ↓ ↑
                  ┌───────────────┼────────────────┐
                  ↓               ↓                ↓
            GUIDE_TRAJECTORY  GUIDE_GESTURE   DRAWING
                  ↓               ↓                ↓
            GESTURE_RECORD ←──────┘            (parallel)
                  ↓
            GESTURE_REPLAY
                  ↓
            AUTO_TUNE (entered only from CALIBRATE or via chord)

Cross-cutting: BALL_LOST, FAULT, SAFE_PARK (entered from any state)
```

Transitions are atomic. Every transition appends a 32-byte entry to a 64-entry ring log in STM32 RAM. Log is broadcast to both ESP32s on heartbeat. State authority is single-source — neither ESP32 may unilaterally change state; they may only request transitions.

### nRF payload (32 bytes fixed, both directions)

```
[0]    msg_type           uint8
[1]    seq                uint8
[2-3]  crc16              uint16
[4-7]  handheld_quat_w/x  int16 × 2 (Q15)   (handheld → base)
[8-11] handheld_quat_y/z  int16 × 2 (Q15)
[12-13] joystick_x/y      int8 × 2
[14]   buttons            uint8 bitmask
[15]   ms_since_motion    uint8
[16-19] ball_x/y          float16 × 2       (base → handheld)
[20-21] rms_err           float16
[22]   state              uint8
[23]   mode               uint8
[24-25] engrave_counter   uint8 × 2 (used / total)
[26-29] reserved          uint32
[30-31] crc16_payload     uint16
```

Heartbeat every 20ms (50Hz). Link-loss threshold 150ms (≈8 missed). On link loss → `SAFE_PARK` silently.

### Topology

- **STM32 Blue Pill:** PID at 1kHz, state machine, calibration storage, MPU-6050 #1, touch panel ADC, two MG996R PWM. Master of time.
- **Base ESP32:** nRF24 to handheld, polar plotter (NEMA17 + radial servo + piezo), 16×2 LCD, 7-segment, SPIFFS trajectory library, ambient light sensor, optional WiFi/MQTT for IDLE-only OTA + web UI at `ballplate-XXXX`.
- **Handheld ESP32:** nRF24, MPU-6050 #2, joystick ADC, OLED, button matrix, TP4056 charge management, MPU motion-wake interrupt.

### Build order (resolved from P4, sequenced for de-risking)

1. **PID loop on STM32** with bench inputs (no touch panel yet). Validate 1kHz timing.
2. **Touch panel integration.** Verify 50–100Hz position read, calibrate corners, characterize noise.
3. **MPU-6050 #1 gravity vector** in CALIBRATE state. Validate plate-level reference.
4. **Kalman filter (2-state pos+vel)** to feed D-term cleanly. Mandatory — D-term without it is unusable.
5. **PRBS sysid + derived gains.** Replace any hand-tuned gains. Lock the boot ritual.
6. **nRF link** between STM32 and handheld ESP32. Validate 20ms heartbeat, 150ms loss detection.
7. **State machine** with logging. All transitions atomic, all logged.
8. **Base ESP32 peripherals.** LCD, 7-seg, LED matrix driver, ambient light sensor.
9. **Trajectory library** with morphing. Public set first, expert set later.
10. **Handheld gesture record/replay** via MPU quaternion integration.
11. **Polar plotter.** NEMA17 + radial servo + piezo. Start with position mode, then add error mode.
12. **Sand wiper integration.** Critical — without it the disc accumulates and looks like a mess.
13. **Optional WiFi/MQTT** for OTA and web trajectory upload. IDLE_DRIFT only.
14. **Auto-tune per ball-mass profile** (steel / glass / ceramic). Final polish.

---

## The Demos

### The 60-Second Public Demo

For anyone — a friend, a stranger, a child. Three acts. No words from you except the two short prompts.

| t | What happens | What the visitor sees |
|---|---|---|
| 0:00 | Machine is already running in Drift. You don't introduce it. | The trail. Their eye finds the comet before they find the ball. |
| 0:10 | You pick up the handheld and put it in their hand. Say nothing. | They feel the weight. The OLED shows nothing yet. |
| 0:15 | You say only: *"tilt it."* | They tilt. The ball follows. The OLED shows `tilt me`. They figure it out in three seconds. |
| 0:35 | You say only: *"press the right button — hold it."* | They hold. The piezo clicks. The needle touches the disc. The plate keeps balancing. They watch the disc rotate. |
| 1:00 | You lift out the disc, hand it to them. | They hold the brass disc with the black-sand drawing. They realize the groove **is** the thirty seconds they just spent. They put it in their shirt pocket. |

That's the demo. The OH moment is not the balance, not the trail, not even the engraving — it is the realization that they made a thing without knowing they were making it.

### The 90-Second Engineering Demo

For someone who recognizes a PID loop on sight. Six acts.

| t | What happens | What it demonstrates |
|---|---|---|
| 0:00 | Drift mode. Point at the LED matrix. | Phase-space oscilloscope — one axis position-err, other axis velocity-est. |
| 0:10 | Poke the ball with a finger. | Disturbance rejection. LEDs flash a ghost of the last 32 samples. 7-seg shows recovery time in ms. |
| 0:25 | Konami chord on handheld. Step-response sub-mode. Run three steps cycling Kp/Ki/Kd. | Under-damped, critically-damped, over-damped settling times on 7-seg. |
| 0:50 | Select Lissajous 3:2. Point at the phase scope. | Trajectory tracking with feedforward — you can see the feedforward kicking in on the scope. |
| 1:05 | Joystick-record a custom path, replay it. | 250-sample ring buffer, 50Hz replay. |
| 1:15 | Trigger PRBS sysid (chord). 60s collapses to 15s of narration over the boot ritual. Then the new gains run a step response. | Closed-loop system identification, derived gains, measurable improvement. |
| 1:30 | Double-tap engrave. | Polar error trace. Pull last two discs out of a drawer — three concentric error rings from under/critical/over-damped, all on one disc. The canonical engineering artifact. |

---

## The Hero Shot

> *Storm to calm. Ten-second exposure.*

Lights off in the room. Camera on a tripod, ten-second exposure at f/8.

For the first six seconds: Storm mode is active. LED matrix full cool-white. Ball flying. The sand-witness fires a radial scar burst — the needle slashes outward from the disc center in a spray of dark lines. The exposure picks up both the comet trail of the ball *and* the LED's bright spray.

For the last four seconds: the chord releases, the machine collapses into Drift. LED fades to warm amber. The ball settles. The needle finishes its scar and lifts.

The photograph shows: a violently scarred sand disc; a warm amber comet trail dissolving into stillness; the walnut base in deep shadow; brass standoffs catching ambient room light. *Storm and aftermath in a single frame.*

This is the shot for the project page, the print on the wall, the proof.

---

## What Earns It the Permanent Desk Spot

A desk object lives or dies on whether you reach for it on the bad days. This earns the spot because:

1. **It is finished, not productized.** No labels, no logo, no version number. It looks like an instrument from a parallel century.
2. **It rewards both glances and study.** A visitor sees a trail and a ball. An engineer sees a PRBS sysid running silently during boot.
3. **It produces objects.** Every session can leave a souvenir. That's rare for a desk gadget — most just blink.
4. **It develops a relationship.** The brass patinas. The boot-lock-time number teaches you what makes the machine sharp. The sand disc accumulates ghost rings beneath the sand.
5. **It does one thing and does it with conviction.** It is not a platform. It is not extensible. It is not an MVP. It is the thing it is.

---

## Build Phases & Risk List

### Phase 0 — Validation (week 1–2)

- Bench-test MG996R servo response at PID rates. **Risk:** MG996R may exhibit dead-band or backlash that limits achievable bandwidth. *Mitigation:* characterize early; budget for swap to DS3218 or similar if measured bandwidth <8Hz.
- Validate 4-wire resistive touch panel under 200mm acrylic plate. **Risk:** ball pressure may be insufficient for reliable touch. *Mitigation:* test with steel, glass, and ceramic balls of varying mass; calibration profile per ball.

### Phase 1 — Core loop (week 3–4)

- STM32 PID at 1kHz, Kalman filter, basic balance.
- **Risk:** D-term amplifies touch-panel ADC noise into servo chatter. *Mitigation:* Kalman is mandatory; do not skip.

### Phase 2 — Identification & auto-tune (week 5)

- PRBS sysid integrated into boot ritual.
- **Risk:** identified plant model is unstable or non-physical. *Mitigation:* fall back to a hand-tuned default gain set if sysid produces poles outside the unit circle.

### Phase 3 — Wireless & state (week 6–7)

- nRF link, state machine, logging.
- **Risk:** nRF24 link instability in noisy 2.4GHz environments. *Mitigation:* channel-hopping protocol, redundant heartbeat.

### Phase 4 — Aesthetic surfaces (week 8–9)

- LED comet, LCD vocabulary, OLED hint, 7-seg mood timer, ambient light sensor.
- **Risk:** LED matrix update rate too low for smooth fade. *Mitigation:* DMA-driven SPI, target 60Hz refresh.

### Phase 5 — Sand witness (week 10–11)

- NEMA17 + radial servo + piezo + wiper assembly. **This is the hardest mechanical subsystem.**
- **Risks:** sand grain size + needle geometry give ugly lines (test with multiple grains, multiple needle tips before committing); brass disc machining and brass collar friction-fit tolerance (machine multiple discs, pick the best); wiper distance behind needle (5° is a guess — tune empirically).
- **Mitigation:** budget two full weeks. This subsystem will be re-prototyped at least twice. Order brass stock and Icelandic black sand early.

### Phase 6 — Handheld fabrication (week 12–13)

- Cast brass shell. **Risk:** casting tolerance for electronics fit. *Mitigation:* prototype shell in resin first; finalize PCB outline before commissioning brass cast.
- 18650 thermal management inside brass shell — brass conducts heat. *Mitigation:* keep cell loading low; measure shell temperature under continuous use.

### Phase 7 — Integration & polish (week 14–15)

- Modes, chords, easter eggs, Storm, Echo.
- Hero shot.
- Public demo dry-runs with naive users — *iterate on the OLED prompt timing until strangers reliably figure out tilt in 3 seconds without further prompting.*

### Phase 8 — Optional (week 16+)

- WiFi/MQTT/OTA. Web trajectory upload. Logbook of all engravings (date, mood timer, mode, sector). These are nice-to-haves; the instrument is complete without them.

### Top three risks, ranked

1. **Sand witness mechanical quality** — this is the souvenir; if the line is ugly, the project fails. Allocate disproportionate time.
2. **MG996R bandwidth** — if servos can't track, the entire control architecture is wrong. Validate week 1.
3. **Naive-user demo discoverability** — if a stranger doesn't figure out "tilt it" in three seconds, the OLED hint and mode model need rework. Test with real strangers in Phase 7.

---

## BOM Delta & Costs

### Acquired (sunk)

- 2× MG996R servos
- 1× NEMA17 stepper
- STM32 Blue Pill (assumed)
- 2× MPU-6050 (assumed)

### To acquire — electronics

| Item | Qty | Est. cost (USD) |
|---|---:|---:|
| 4-wire resistive touch panel, ≥200mm | 1 | 15 |
| 8×8 LED matrix (MAX7219 or WS2812 backed) | 1 | 8 |
| nRF24L01+ modules | 2 | 6 |
| ESP32 dev boards (base + handheld) | 2 | 16 |
| 0.96" SPI OLED, smoked acrylic window | 1 | 6 |
| HW-504 joystick | 1 | 3 |
| 16×2 LCD (I2C backpack) | 1 | 5 |
| 7-segment display (4-digit, TM1637 or HT16K33) | 1 | 6 |
| TP4056 charge module | 1 | 2 |
| 18650 cell (handheld) + holder | 1 | 8 |
| 3S LiPo (base) or bench supply | 1 | 25 |
| LM2596 buck modules | 2 | 4 |
| Ambient light sensor (BH1750 or TSL2561) | 1 | 4 |
| Piezo element (sand-click) | 1 | 1 |
| Radial servo (sand needle, small — SG90 or MG90S) | 1 | 5 |
| A4988 / DRV8825 stepper driver | 1 | 4 |
| Current-sense module (servo overcurrent fault) | 1 | 4 |
| Misc connectors, cabling, perfboard, JST, headers | — | 25 |
| **Subtotal — electronics** | | **~$147** |

### To acquire — mechanical / materials

| Item | Qty | Est. cost (USD) |
|---|---:|---:|
| Walnut 18mm board, ~400×300mm | 1 | 40 |
| Danish or tung oil finish, fine sandpaper | — | 15 |
| Brass standoffs assortment | — | 12 |
| 5mm clear cast acrylic, 200mm disc (flame-polished edge) | 1 | 15 |
| 120mm brushed brass disc stock (or machined) | 2–3 | 30 |
| Brass collar stock (friction-fit) | — | 10 |
| Brass tin for sand replacement | 1 | 8 |
| Icelandic black volcanic sand (specialty supplier) | 500g | 20 |
| Fine chisel-tip needle (steel or brass) | spare | 6 |
| Silicone strip for wiper | — | 4 |
| Cast brass handheld shell (commissioned or DIY lost-wax) | 1 | 60–150 |
| Delrin rod for joystick crown | — | 6 |
| PLA/PETG filament (satin black final pass) | — | 10 |
| Smoked acrylic for OLED window | — | 5 |
| **Subtotal — mechanical** | | **~$241–$331** |

### Total estimated delta

**≈ $390 – $480** above already-acquired items. The handheld brass shell is the single largest line item and the single largest cost-quality lever — DIY lost-wax casting can bring it under $40, commissioning a small shop will be $100–150.

---

## Five Tensions, Five Decisions

For the record, the five tensions identified in the brief, decided here:

1. **Tilt vs joystick primary** → **Tilt wins as primary.** Joystick is the silent expert option. P5's water-in-a-bowl intuition beats P1's joystick-setpoint precision for the demo path; the joystick remains available for builders who already know the instrument.

2. **Sand: position vs error signal** → **Position is the primary semantic** (default tap and hold). Error trace is preserved as a diagnostic, summoned by **double-tap**. This honors P5's souvenir legibility *and* P2's three-concentric-rings engineering artifact, on the same disc, via the same hardware.

3. **Mode naming** → **Drift / Guide / Engrave** is the public LCD vocabulary. **Reverie / Tempest / Pulse / Fossil / Echo** are firmware mood names and build-doc names. **Free-balance / Trajectory / PRBS-ID** etc. are source-code-only. Three layers, one truth per layer.

4. **Trajectory count** → **5 public + expert library.** Public set is the 5 P1 trajectories (Lissajous 3:2, rose 5/3, figure-8, inward spiral, outward spiral). Expert library lives on SPIFFS, reachable only by chord, and includes the full P4 set plus user-recorded gestures.

5. **Visible vs hidden tech depth** → **Hidden by default, summoned by chord.** PRBS sysid, Kalman filter, feedforward run silently — their only public manifestation is the boot lock time on the 7-segment. Engineering-curious visitors can summon Step Response and Sysid sub-modes via the Konami-like chord. The naive-user surface never shows a gain value, ever.

---

*"The ball balances. The sand remembers."*

*— end of brief —*
