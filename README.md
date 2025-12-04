# Klipper config for V0.3048

This repository contains the Klipper configuration for a Voron 0.2 (V0.3048) using a BTT SKR Pico V1.0 mainboard and an LDO Nitehawk-36 toolboard, plus shared macros via `nerdygriffin-macros` and KAMP.

## Architecture Overview
- Main entry: `printer.cfg` includes subsystems and plugins.
- Symlinked plugins: `nerdygriffin-macros/` (shared macros), `KAMP/` (Adaptive Purge/Smart Park). Do not modify symlinked content; override locally.
- Hardware-specific: `nitehawk-36.cfg` (toolhead), `nozzlewiper.cfg` (servo + clean).

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

## Credits

- [LDO Nitehawk-36 Toolboard](https://github.com/MotorDynamicsLab/Nitehawk-36) - RP2040-based USB toolboard with integrated TMC2209, accelerometer, and tachometer-enabled fans
