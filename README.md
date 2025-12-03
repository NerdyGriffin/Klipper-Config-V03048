# Klipper config for V0.3048

This repository contains the Klipper configuration for a Voron 0.2 (V0.3048) using a BTT SKR Pico V1.0 mainboard and an LDO Nitehawk-36 toolboard, plus shared macros via `nerdygriffin-macros` and KAMP.

## Architecture Overview
- Main entry: `printer.cfg` includes subsystems and plugins.
- Symlinked plugins: `nerdygriffin-macros/` (shared macros), `KAMP/` (Adaptive Purge/Smart Park). Do not modify symlinked content; override locally.
- Hardware-specific: `nitehawk-36.cfg` (extruder, LEDs), `nozzlewiper.cfg` (servo + clean), `status-macros.cfg`.

## Workflows
- Restart + tail logs after config edits:
	```bash
	sudo systemctl restart klipper
	tail -f ~/printer_data/logs/klippy.log
	```
- Test macros quickly:
	```bash
	curl -s -X POST "http://localhost:7125/printer/gcode/script?script=CLEAN_NOZZLE"
	curl -s -X POST "http://localhost:7125/printer/gcode/script?script=HEAT_SOAK%20CHAMBER=45%20DURATION=5"
	```

## Notes
- Symlinked directories (`KAMP`, `nerdygriffin-macros`) contain files not tracked in this repo. Override behavior in local config; do not edit symlinked content.

This is the contents of the `/home/pi/printer_data/config/` directory [NOTE: this line may be redundant. consider removing it]

```
$ tree
.
├── .github
│   └── [copilot instructions & prompts]
├── autotune.cfg
├── chamber_leds.cfg
├── crowsnest.conf
├── deprecated
│   └── [archived configs]
├── docs
│   └── [documentation files]
├── homing.cfg
├── KAMP -> [symlink to KAMP installation]
├── KAMP_Settings.cfg
├── Klicky-Probe
│   └── [optional probe macros]
├── nerdygriffin-macros -> [symlink to shared macros]
├── nitehawk-36.cfg
├── nozzlewiper.cfg
├── printer.cfg
├── print_macros.cfg
├── status-macros.cfg
└── TEST_SPEED.cfg

Note: Symlinked directories (KAMP, nerdygriffin-macros) contain files not tracked in this repo.
See git history for complete file listing.
