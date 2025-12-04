# Klipper config for V0.3048

This repository contains the Klipper configuration for a Voron 0.2 (V0.3048) using a BTT SKR Pico V1.0 mainboard and an LDO Nitehawk-36 toolboard, plus shared macros via `nerdygriffin-macros` and KAMP.

## Architecture Overview
- Main entry: `printer.cfg` includes subsystems and plugins.
- Symlinked plugins: `nerdygriffin-macros/` (shared macros), `KAMP/` (Adaptive Purge/Smart Park). Do not modify symlinked content; override locally.
- Hardware-specific: `nitehawk-36.cfg` (toolhead), `nozzlewiper.cfg` (servo + clean).

## Shared Macros Integration

This config uses the [klipper-nerdygriffin-macros](https://github.com/NerdyGriffin/klipper-nerdygriffin-macros) plugin for hardware-agnostic macro support:
- 13 consolidated macros included via `nerdygriffin-macros/` symlink
- Automatic AFC detection and fallback support
- Per-printer customization via variable overrides in `printer.cfg`

Macros included:
- `auto_pid.cfg` - Automated PID tuning
- `beeper.cfg` - M300 beeper support
- `client_macros.cfg` - Pause/Resume/Cancel hooks
- `filament_management.cfg` - Load/Unload/Purge operations
- `heat_soak.cfg` - Chamber preheating with LED animations
- `maintenance_macros.cfg` - Maintenance helpers (SETTLE_BELT_TENSION, etc.)
- `rename_existing.cfg` - Enhanced G-code overrides
- `save_config.cfg` - Safe config saving
- `shaketune.cfg` - Shake&Tune wrappers
- `shutdown.cfg` - Safe shutdown operations
- `status_macros.cfg` - Status LED patterns
- `tacho_macros.cfg` - Fan preflight checks
- `utility_macros.cfg` - Utility helpers

## Notes
- Symlinked directories (`KAMP`, `nerdygriffin-macros`) contain files not tracked in this repo
- Override behavior in local config; do not edit symlinked content
