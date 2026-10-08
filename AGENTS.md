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
- Local overrides: `nerdygriffin-macros.cfg`, included _last_ (after every `nerdygriffin-macros/*.cfg`) and organized by upstream filename. Holds the beeper pin, `[servo wipeServo]`, and every shared-macro variable override.
- Optional probe: `Klicky-Probe/` present but commented out; Z homes to endstop centrally without Beacon.

## Key Subsystems

- Status LEDs (`nerdygriffin-macros/status_macros.cfg`): `STATUS_*` patterns drive `bed_light`, Nitehawk `toolhead` pixels and the
  `chamber_left`/`chamber_right` strips (`chamber_leds.cfg`). The name-to-group mapping (`logo`/`nozzle`/`chamber`) is overridden in
  `nerdygriffin-macros.cfg` (`_LED_VARS`).
- Homing — **two files that extend each other, no overlap**:
  - `nerdygriffin-macros/homing.cfg` (shared): `_CG28`, `_HOME_VARS`, `_HOME_PRE_AXIS`/`_HOME_POST_AXIS`,
    `_HOME_EDGE_CLEARANCE`, `TEST_SENSORLESS_HOME_*`.
  - `homing_override.cfg` (local, included straight after): `_HOME_X`/`_HOME_Y`/`_HOME_Z` and
    `[homing_override]`. Load-bearing — `[homing_override]` is what routes `G28` to those macros.
    Named `homing_override.cfg` rather than `homing.cfg` to keep it distinct from the shared file.
  - Sensorless XY via TMC2209 with pre/post current changes; Z homes at bed center using a switch endstop (the centre is hard-coded
    `60,60` in `_HOME_Z` in `homing_override.cfg`).
  - VT-1548 has no local counterpart: `[beacon]` supplies the `homing_override` equivalent there
    (`beacon.cfg` line 40: `# [homing_override] ## DELETE THIS, it is handled by the [beacon] section`).
- Print flow (`nerdygriffin-macros/print_macros.cfg`):
  - `PRINT_START`: `CLEAR_PAUSE` (stale-pause guard) first, then heat soak based on bed temp, hot scrub via `CLEAN_NOZZLE`, re-home Z,
    `SMART_PARK` (KAMP), then purge: `SQUIGGLY_PURGE` when objects allow it, else `LINE_PURGE`.
  - `PRINT_END`: retract, park at the rear-right corner (Y rear-10, then X right-10), disable sensors, delayed save/shutdown.
- Filament sensors (three defined, two active, on different MCUs):
  - `encoder_sensor` (`filament_motion_sensor`, SKR Pico `^gpio16`): flow/clog detection. Enabled after
    start, disabled on end and idle timeout. Its `runout_gcode` calls `_PAUSE_IF_PRINTING`. A
    `switch_sensor` block sits commented out on the _same_ pin in `printer.cfg` — the two are mutually
    exclusive. That switch is physically installed but not wired: the Pico has no free gpio for it.
  - `extruder_tool_start` (`filament_switch_sensor`, `^nhk:gpio3` in `nitehawk-36.cfg`): toolhead
    switch at the extruder inlet. `_PAUSE_IF_PRINTING` is deliberately commented out of its
    `runout_gcode` (with `RESET_STATUS` in its place) — the switch mount / ball bearing / filament
    path tolerances mean filament must be pushed to one side to trigger it. That is a **mechanical**
    fault, not an electrical one; the switch reports consistently when pressed. Toolhead was rebuilt
    with new switches and ball 2026-10 and needs long-print testing before PAUSE is re-enabled. Its
    `insert_gcode` runs `_AUTO_LOAD_FILAMENT`.
    - Deliberate split: the encoder pauses, the toolhead switch guards RESUME. `_CLIENT_VARIABLE` `runout_sensor` in `printer.cfg`
      is `filament_switch_sensor extruder_tool_start`, so Mainsail `RESUME` is blocked while that switch reads empty.
  - `extruder_tool_end` (`filament_switch_sensor`, `^nhk:gpio13`): placeholder after the extruder;
    reports only. Used by `_AUTO_LOAD_FILAMENT` to confirm the load reached the nozzle.
  - The `^` pullup is optional here: the Nitehawk-36 V1.4 filament header already has a 10K pull-up to
    3V3 (R16), a 100R series resistor to the MCU pin (R19), and a 100nF cap to GND (C27) forming an
    ~1ms RC debounce. It is set for parity with VT-1548, not because the pin would otherwise float.
  - **Never put a bare `PAUSE` in a `runout_gcode`.** Use `_PAUSE_IF_PRINTING`
    (`nerdygriffin-macros/filament_management.cfg`). `pause_on_runout: False` plus a gated PAUSE in
    `runout_gcode` is intentional. Stock Klipper `PAUSE` sets `pause_resume.is_paused` even with no job
    running and `SDCARD_PRINT_FILE` never clears it, so an idle-time runout leaks `is_paused=True` into
    the next print, where `NOZZLE_STANDBY_COOLDOWN` (`client.cfg`) sees it and issues `M104 S150`
    mid-print. That caused a 2026-10-07 abort (`Extrude below minimum temp` at 168 °C) that looked like
    a heater failure. `PRINT_START` now runs `CLEAR_PAUSE` as a second line of defence. The helper keys
    on `print_stats.state == "printing"` (written only by `virtual_sdcard`), not `idle_timeout.state`,
    which reports `Printing` during any manual move.
- LDO Nitehawk-36: `[extruder]`, `[neopixel toolhead]`, `[adxl345]` and the toolhead filament switches live in `nitehawk-36.cfg`;
  `bed_light` is in `printer.cfg` and the chamber strips in `chamber_leds.cfg`.

## Patterns & Conventions

- Always prefer `_CG28` for conditional homing; `homing_override.cfg` wraps raw `G28` with current/LED/fan handling.
- Save/restore state around disruptive ops: `SAVE_GCODE_STATE`/`RESTORE_GCODE_STATE` (with `MOVE=1` when needed).
- Check optional hardware/macros before calling:
  `{% if printer['gcode_macro AFC_BRUSH'] is defined %} AFC_BRUSH {% endif %}`
- Status signaling: call `STATUS_*` before long-running actions and `RESET_STATUS`/`STATUS_READY` when done.
- Delayed actions: use `[delayed_gcode ...]` with `UPDATE_DELAYED_GCODE` (see `nerdygriffin-macros/status_macros.cfg` notify heaters).

## Integration Points

- `nerdygriffin-macros` includes: see the `[include nerdygriffin-macros/*]` block in `printer.cfg` for the authoritative list (plus
  `homing.cfg` and `idle_timeout.cfg`, included earlier in the file).
- KAMP ([Klipper Adaptive Meshing & Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)): `printer.cfg` includes
  `KAMP/Line_Purge.cfg` and `KAMP/Smart_Park.cfg` directly (Adaptive_Meshing/Voron_Purge commented out). `KAMP_Settings.cfg` holds
  only the `_KAMP_Settings` variables; its own include lines are commented.
- Nozzle wiper: `CLEAN_NOZZLE` maps to `NW_CLEAN_NOZZLE` in `nerdygriffin-macros/nozzle_wiper.cfg`. The
  `[servo wipeServo]` definition (`gpio29`) and the `_NW_BRUSH_VARS` / `NW_BUCKET_POS` overrides live in
  `nerdygriffin-macros.cfg`; `references/nozzlewiper.cfg` is archived.

## Hardware & Limits (V0.3048)

- Serial Number: V0.3048
- Hostname: V0-3048
- Motion: CoreXY. Tuned limits (`max_velocity`, `max_accel`, `max_z_velocity`, `max_z_accel`) live in `[printer]` in `printer.cfg`.
- Build volume: nominal 120³ V0 bed; actual travel limits are in `[stepper_*]` in `printer.cfg` (X/Y adjusted for the offset
  toolhead mount; Z endstop calibrated in the SAVE_CONFIG block).
- Bed heater: Pico `heater_bed` on `gpio21`; chamber thermistor on `gpio27` (`temperature_sensor chamber`).
- Hotend/extruder: see `[extruder]` in `nitehawk-36.cfg` (Galileo 2, Phaetus Dragon UHF, PT1000). 100 W heater fitted during the
  2026-09/10 toolhead rebuild; PID re-tuned 2026-09-30 in commit `fe2e1c3`.
- Nevermore: `[heater_fan filter_fan]` in `printer.cfg`, keyed to both `extruder` and `heater_bed` temps.
- LEDs: `bed_light` (`printer.cfg`), `toolhead` pixels (`nitehawk-36.cfg`), `chamber_left`/`chamber_right` strips (`chamber_leds.cfg`,
  included from `printer.cfg`); all mapped to groups in `_LED_VARS` (`nerdygriffin-macros.cfg`).
- Beeper: `[pwm_cycle_time beeper]` pin override in `nerdygriffin-macros.cfg`.
- Chamber heating: via bed+hotend assist in `HEAT_SOAK`. The ceiling is `variable_max_chamber_target` in `nerdygriffin-macros.cfg`, set from a measured maximum.
  - The chamber is poorly insulated, so the max chamber target is limited to avoid `TEMPERATURE_WAIT` for a temperature that it can never reach.
- **Note**: See the authoritative values in `printer.cfg` for current motion limits, speeds, and calibrated offsets. Do not rely on hard-coded values in documentation; always check the actual config.

## Typical Overrides (Local only)

- All shared-macro overrides live in `nerdygriffin-macros.cfg`, **not** `printer.cfg`, grouped by upstream
  filename: `HEAT_SOAK` (`max_chamber_target`, `ext_assist_multiplier`), `_BELT_TENSION_VARS`
  (`belt_span_length`, `y_calibrated`), `_HOME_VARS`, `_LED_VARS`, `_NW_BRUSH_VARS` / `NW_BUCKET_POS`, the
  LOAD/UNLOAD/PURGE distances and `_AUTO_LOAD_FILAMENT` (`load_distance`). Read current values there rather than trusting numbers in this file.
- KAMP behavior: tweak Smart Park/Purge variables in `_KAMP_Settings` (`KAMP_Settings.cfg`); enable/disable KAMP modules via the
  `[include ./KAMP/*]` lines in `printer.cfg`, not `KAMP_Settings.cfg`.

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
  The persistent threshold is `sg4_thrs` in `autotune.cfg` (TMC autotune); `driver_SGTHRS` is commented out in `printer.cfg`.
- Input shaper: `[adxl345]` on the Nitehawk (`nitehawk-36.cfg`); `[resonance_tester]` tuning in `printer.cfg`.

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
- `_LED_VARS` names must match real `[neopixel ...]` sections (`bed_light`, `toolhead`, `chamber_left`, `chamber_right`); add the
  neopixel definition before adding a name to a group.
- AFC macros are conditionally referenced in `idle_timeout.cfg`, `client.cfg` and `PRINT_START`/`PRINT_END` but not required on this printer.

## Deprecated Files

- Files in `deprecated/` are archived configs kept for reference only.
- Do not document or suggest using deprecated files.
- These files are tracked for historical reference, but they are not actively maintained.

## File Landmarks

- Entry: `printer.cfg`
- Homing: `nerdygriffin-macros/homing.cfg` (shared helpers) + `homing_override.cfg` (local `_HOME_*`)
- Status LEDs: `nerdygriffin-macros/status_macros.cfg`
- Start/End: `nerdygriffin-macros/print_macros.cfg`
- Wiper: `nerdygriffin-macros/nozzle_wiper.cfg` (overrides in `nerdygriffin-macros.cfg`)
- Shared macros: `nerdygriffin-macros/`
- Other includes from `printer.cfg`: `mainsail.cfg` (symlink; provides `[pause_resume]`), `moonraker_obico_macros.cfg` and
  `timelapse.cfg` (symlinks), `TEST_SPEED.cfg`, `chamber_leds.cfg`, `autotune.cfg`, `KAMP_Settings.cfg`
- Boot hook: `[delayed_gcode MACHINE_STARTUP]` in `printer.cfg` runs `NW_RETRACT` + `_STARTUP_BEEP` at startup

## Ease of use

- If I repeated request actions that contradict these instructions, propose ways to improve these instructions.
- **Important:** This file (`AGENTS.md`) is the vendor-neutral source of truth for AI-agent guidance —
  surfaced to Claude Code via `CLAUDE.md` (`@AGENTS.md` import) and to GitHub Copilot via the
  `.github/copilot-instructions.md` symlink. Edit `AGENTS.md`, not the pointers. Do not reference it
  from user-facing docs (README.md); those must be self-contained.
