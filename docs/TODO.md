# TODO

- [ ] Consodiate shared macros and configs between VT1548 and V03048 into a custom plugin that can be updated via moonraker
  - Goal: to reduce duplication and ensure consistency between both printers
  - Can be symlinked into each printer's config directory for easy inclusion, in the style of KAMP
  - [ ] Research best practices for Klipper plugin development
- [X] Standardize HEAT_SOAK display formatting in plugin
  - Replace duplicate `SET_DISPLAY_TEXT` lines with a single canonical form
    - Preferred: `SET_DISPLAY_TEXT MSG={"Heatsoak: %02d:%02d"|format(MINUTES_FORMATED, SECONDS_FORMATED)}`
  - Remove any mixed quote/brace variants and ensure consistency across macros
  - Quick audit other macros for similar string-formatting patterns
- [ ] Consider removing temperature parameter from LOAD/UNLOAD filament macros.
  - The latest versions of mainsail, KlipperScreen, or AFC seem to bypass this behavior or require the extruder above min_print_temp before the macro is allowed to run
