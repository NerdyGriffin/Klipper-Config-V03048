# AI Agent Instructions for Klipper-Config-V0-3048

This is a Klipper configuration for a Voron 0.2 (V0.3048) built around a BTT SKR Pico V1.0 mainboard and an LDO Nitehawk-36 toolboard. It uses hardware-agnostic macros from `nerdygriffin-macros`, plus local printer-specific overrides.

## Workspace Context

**File locations:**
- **Local config**: `/home/pi/printer_data/config` (this printer)
- **VT-1548 config**: `/mnt/vt-1548/printer_data/config` (NFS mount, writable for cross-printer dev)
- **Macros repo**: `/home/pi/klipper-nerdygriffin-macros` (local, sync to VT-1548 via `dev/sync_macros_repo.sh`)

## Architecture Overview
- Main entry: `printer.cfg` includes all subsystems and plugin configs.
- Symlinked plugin: `nerdygriffin-macros/` (shared macros), `KAMP/` (Adaptive Purge/Smart Park). Do not modify symlinked content; override locally.
- Hardware-specific files: `nitehawk-36.cfg` (extruder, toolhead filament sensor, LEDs), stepper/heater in `printer.cfg`.
- Local overrides: `nerdygriffin-macros.cfg`, included *last* (after every `nerdygriffin-macros/*.cfg`) and organized by upstream filename. Holds the beeper pin, `[servo wipeServo]`, and every shared-macro variable override.
- Optional probe: `Klicky-Probe/` present but commented out; Z homes to endstop centrally without Beacon.

## Key Subsystems
- Status LEDs (`status_macros.cfg`): `STATUS_*` patterns drive `bed_light` and Nitehawk `toolhead` pixels. LED variables are overridden in `nerdygriffin-macros.cfg` (`_LED_VARS`).
- Homing (`homing.cfg`):
  - Sensorless XY via TMC2209 with pre/post current changes (`_HOME_PRE_AXIS`/`_HOME_POST_AXIS`).
  - Z homes at center (`X=60,Y=60`) using switch endstop; `_CG28` provided for conditional homing.
- Print flow (`print_macros.cfg`):
  - `PRINT_START`: heat soak based on bed temp, hot scrub via `CLEAN_NOZZLE`, re-home Z, `SMART_PARK` (KAMP), then purge (`LINE_PURGE`).
  - `PRINT_END`: retract, park at rear-10mm, disable sensors, delayed save/shutdown.
- Filament sensors (two, on different MCUs):
  - `encoder_sensor` (`filament_motion_sensor`, SKR Pico `^gpio16`): flow/clog detection. Enabled after
    start, disabled on end and idle timeout. A `switch_sensor` block sits commented out on the *same*
    pin in `printer.cfg` — the two are mutually exclusive.
  - `extruder_sensor` (`filament_switch_sensor`, `^!nhk:gpio3` in `nitehawk-36.cfg`): toolhead switch.
    `PAUSE` is deliberately commented out of its `runout_gcode` — the switch mount / ball bearing /
    filament path tolerances mean filament must be pushed to one side to trigger it. That is a
    **mechanical** fault, not an electrical one; the invert is correct and the switch reports
    consistently when pressed.
  - The `^` pullup is optional here: the Nitehawk-36 V1.4 filament header already has a 10K pull-up to
    3V3 (R16), a 100R series resistor to the MCU pin (R19), and a 100nF cap to GND (C27) forming an
    ~1ms RC debounce. It is set for parity with VT-1548, not because the pin would otherwise float.
- LDO Nitehawk-36: Extruder definition in `nitehawk-36.cfg`; toolhead LEDs and bed neopixel configured in `printer.cfg`.

## Patterns & Conventions
- Always prefer `_CG28` for conditional homing; `homing.cfg` wraps raw `G28` with current/LED/fan handling.
- Save/restore state around disruptive ops: `SAVE_GCODE_STATE`/`RESTORE_GCODE_STATE` (with `MOVE=1` when needed).
- Check optional hardware/macros before calling:
  `{% if printer['gcode_macro AFC_BRUSH'] is defined %} AFC_BRUSH {% endif %}`
- Status signaling: call `STATUS_*` before long-running actions and `RESET_STATUS`/`STATUS_READY` when done.
- Delayed actions: use `[delayed_gcode ...]` with `UPDATE_DELAYED_GCODE` (see `status_macros.cfg` notify heaters).

## Integration Points
- `nerdygriffin-macros` includes (in `printer.cfg`): auto PID, beeper, client hooks, filament mgmt, heat soak, maintenance, rename/save_config, shaketune, shutdown, tacho, utilities.
- KAMP ([Klipper Adaptive Meshing & Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)) includes: `Line_Purge.cfg`, `Smart_Park.cfg` for intelligent bed mesh and purge positioning based on print geometry (Adaptive_Meshing/Voron_Purge are optional/commented).
- Nozzle wiper: `CLEAN_NOZZLE` maps to `NW_CLEAN_NOZZLE` in `nerdygriffin-macros/nozzle_wiper.cfg`. The
  `[servo wipeServo]` definition (`gpio29`) and the `_NW_BRUSH_VARS` / `NW_BUCKET_POS` overrides live in
  `nerdygriffin-macros.cfg` — `printer.cfg` has an older commented-out servo block, and
  `references/nozzlewiper.cfg` is archived.

## Hardware & Limits (V0.3048)
- Serial Number: V0.3048
- Hostname: V0-3048
- Motion: CoreXY. Tuned limits (`max_velocity`, `max_accel`, `max_z_velocity`, `max_z_accel`) live in `[printer]` in `printer.cfg`.
- Build volume: 120×120×120 (Z endstop calibrated; see SAVE_CONFIG block).
- Bed heater: Pico `heater_bed` on `gpio21`; chamber thermistor on `gpio27` (`temperature_sensor chamber`).
- LEDs: `bed_light` (1 GRBW), toolhead pixels on Nitehawk; panel LEDs are placeholders.
- Beeper: `[pwm_cycle_time beeper]` pin override in `nerdygriffin-macros.cfg`.
- Chamber heating: via bed+hotend assist in `HEAT_SOAK`. The ceiling is `variable_max_chamber_target` in `nerdygriffin-macros.cfg`, set from a measured maximum.
  - The chamber is poorly insulated, so the max chamber target is limited to avoid `TEMPERATURE_WAIT` for a temperature that it can never reach.
- **Note**: See the authoritative values in `printer.cfg` for current motion limits, speeds, and calibrated offsets. Do not rely on hard-coded values in documentation; always check the actual config.

## Typical Overrides (Local only)
- All shared-macro overrides live in `nerdygriffin-macros.cfg`, **not** `printer.cfg`, grouped by upstream
  filename: `HEAT_SOAK` (`max_chamber_target`, `ext_assist_multiplier`), `_BELT_TENSION_VARS`
  (`y_calibrated`), `_HOME_VARS`, `_LED_VARS`, `_NW_BRUSH_VARS` / `NW_BUCKET_POS`, and the
  LOAD/UNLOAD/PURGE distances. Read current values there rather than trusting numbers in this file.
- KAMP behavior: tweak Smart Park/Purge via `KAMP_Settings.cfg`.

## Workflows
- Restart + tail logs after config edits:
  ```bash
  curl -s -X POST "http://localhost:7125/printer/gcode/script?script=FIRMWARE_RESTART" && sleep 2 && tail -n 60 ~/printer_data/logs/klippy.log
  ```
- Test macros quickly:
  ```bash
  curl -s -X POST "http://localhost:7125/printer/gcode/script?script=CLEAN_NOZZLE"
  curl -s -X POST "http://localhost:7125/printer/gcode/script?script=HEAT_SOAK%20CHAMBER=45%20DURATION=5"
  ```
- Sensorless tuning (XY):
  ```gcode
  TEST_SENSORLESS_HOME_X TEST_SGTHRS=255
  TEST_SENSORLESS_HOME_Y TEST_SGTHRS=255
  ```
- Input shaper: ADXL345 (`[resonance_tester]` in `printer.cfg`).

## Terminal Command Best Practices
- **Always use verbose flags** (`-v` or `--verbose`) with file operations for visual confirmation:
  - `cp -v` instead of `cp`
  - `mv -v` instead of `mv`
  - `rm -v` instead of `rm`
  - `rmdir -v` instead of `rmdir`
  - `ln -sfv` instead of `ln -sf`
- This provides immediate feedback and helps catch errors early.

## Cautions
- Do not edit symlinked `nerdygriffin-macros/` or `KAMP/`; override via local macros/variables after includes.
- Panel LED macros assume `panel_left/right` exist; leave placeholders or add actual neopixel definitions before enabling.
- AFC macros are conditionally referenced in `idle_timeout` but not required on this printer.

## Deprecated Files
- Files in `deprecated/` are archived configs kept for reference only.
- Do not document or suggest using deprecated files.
- These files are tracked for historical reference, but they are not actively maintained.

## File Landmarks
- Entry: `printer.cfg`
- Homing: `homing.cfg`
- Status LEDs: `status_macros.cfg`
- Start/End: `print_macros.cfg`
- Wiper: `nerdygriffin-macros/nozzle_wiper.cfg` (overrides in `nerdygriffin-macros.cfg`)
- Shared macros: `nerdygriffin-macros/`

## Ease of use
- If I repeated request actions that contradict these instructions, propose ways to improve these instructions.
- **Important:** This file (`AGENTS.md`) is the vendor-neutral source of truth for AI-agent guidance —
  surfaced to Claude Code via `CLAUDE.md` (`@AGENTS.md` import) and to GitHub Copilot via the
  `.github/copilot-instructions.md` symlink. Edit `AGENTS.md`, not the pointers. Do not reference it
  from user-facing docs (README.md, docs/*.md); those must be self-contained.
