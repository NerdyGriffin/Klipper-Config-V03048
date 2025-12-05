# Klipper config for V0.3048

This repository contains the Klipper configuration for a Voron 0.2 (V0.3048) using a BTT SKR Pico V1.0 mainboard and an LDO Nitehawk-36 toolboard, plus shared macros via `nerdygriffin-macros` and KAMP.

## Architecture Overview
- Main entry: `printer.cfg` includes subsystems and plugins.
- Symlinked plugins: `nerdygriffin-macros/` (shared macros), `KAMP/` (Adaptive Purge/Smart Park). Do not modify symlinked content; override locally.
- Hardware-specific: `nitehawk-36.cfg` (toolhead).

## Installed Klipper Plugins

- [crowsnest](https://github.com/mainsail-crew/crowsnest)
- [katapult](https://github.com/Arksine/katapult)
- [Klippain-ShakeTune](https://github.com/Frix-x/klippain-shaketune)
- [Klipper-Adaptive-Meshing-Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) (KAMP)
- [klipper-nerdygriffin-macros](https://github.com/NerdyGriffin/klipper-nerdygriffin-macros)
- [klipper_tmc_autotune](https://github.com/andrewmcgr/klipper_tmc_autotune)
- [KlipperScreen](https://github.com/KlipperScreen/KlipperScreen)
- [moonraker-obico](https://github.com/TheSpaghettiDetective/moonraker-obico)
- [timelapse](https://github.com/mainsail-crew/moonraker-timelapse)

---

## Hardware Mods

### Nozzle Wiper v2
- **Source**: [chirpy2605/voron - V0/NozzleWiper](https://github.com/chirpy2605/voron/tree/main/V0/NozzleWiper)
- **Description**: Servo-actuated nozzle purge bucket and brush system for automated nozzle cleaning
- **Features**:
  - SG90 servo-controlled deployment (extends 90°, retracts when not in use)
  - Integrated purge bucket and brush
  - Hot-to-cold cleaning capability (wipes while cooling to minimize oozing)
  - Randomized wipe patterns across brush depth
  - Configurable wipe count and speeds
- **Configuration**: See `nerdygriffin-macros/nozzle_wiper.cfg` for setup instructions
- **Macro**: `CLEAN_NOZZLE`

---

## Credits

- [chirpy2605](https://github.com/chirpy2605) - Creator of many of my favorite Voron mods, including DragonBurner and Nozzle Wiper v2.
- [MapleLeafMakers](https://github.com/MapleLeafMakers) - Matchstick Diffusers and more
  - https://mods.vorondesign.com/details/SMqxDRtUkiizoQXq5nhQQ
- [LDO Nitehawk-36 Toolboard](https://github.com/MotorDynamicsLab/Nitehawk-36) - RP2040-based USB toolboard with integrated TMC2209, accelerometer, and tachometer-enabled fans
