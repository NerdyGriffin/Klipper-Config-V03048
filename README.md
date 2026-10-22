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

### BTT Smart Filament Sensor V2.0

- **Source**: [bigtreetech/smart-filament-detection-module](https://github.com/bigtreetech/smart-filament-detection-module)
- **Description**: Runout switch plus a motion encoder for jam/clog detection, on the SKR Pico with the stock SFS V2.0
  split cable — no re-pinning
- **Wiring**:

  | SFS connector    | SKR Pico header             | Klipper pin | Config section                            |
  | ---------------- | --------------------------- | ----------- | ----------------------------------------- |
  | Switch / — / VCC | E0-STOP (IO16 / GND / 5V)   | `^gpio16`   | `[filament_switch_sensor switch_sensor]`  |
  | Motion / GND / — | WD-DET (P_S / GND / 12/24V) | `gpio22`    | `[filament_motion_sensor encoder_sensor]` |

- **Board jumpers** (both required for the WD-DET input):
  - **P21**: jumper `P_S` to `IO22`, otherwise the encoder isn't connected to the MCU.
  - **JP2**: leave **off**. ⚠️ JP2 connects a 10K pull-up between `P_S` and the 12/24V supply, which could fry the SFS with 24V (rated 5V max).
- **Why WD-DET instead of PROBE?** The motion sensor output is 5V. The endstop headers (`X-STOP`, `Y-STOP`,
  `Z-STOP`, `E0-STOP`) pass through a `74LVC2G34DW` buffer, so the RP2040 sees the signal as 3.3V.
  `PROBE` has no buffer: the motion output there would put 5V straight onto the RP2040's `gpio22` and
  could damage it. WD-DET isolates the signal with an optocoupler instead, so the RP2040 sees the signal as 3.3V and `gpio22` needs no `^`
  pull-up.
- **Behaviour**: `switch_sensor` only reports; it never pauses. `encoder_sensor` pauses on a jam, and only
  while the switch still sees filament. When the switch is clear, the tail-end of the filament has left the sensor, so the
  encoder just posts a message and lets the tail run down to the toolhead sensor (`extruder_tool_start`),
  which does the runout pause. Pauses go through `_PAUSE_IF_PRINTING`, so a runout while idle never leaves
  a stale pause behind.

---

## Credits

- [chirpy2605](https://github.com/chirpy2605) - Creator of many of my favorite Voron mods, including DragonBurner and Nozzle Wiper v2.
- [MapleLeafMakers](https://github.com/MapleLeafMakers) - Matchstick Diffusers and more
  - https://mods.vorondesign.com/details/SMqxDRtUkiizoQXq5nhQQ
- [LDO Nitehawk-36 Toolboard](https://github.com/MotorDynamicsLab/Nitehawk-36) - RP2040-based USB toolboard with integrated TMC2209, accelerometer, and tachometer-enabled fans
