# TODO

## Consolidation Project
- [ ] Consodiate shared macros and configs between VT1548 and V03048 into a custom plugin that can be updated via moonraker
  - Goal: to reduce duplication and ensure consistency between both printers
  - Can be symlinked into each printer's config directory for easy inclusion, in the style of KAMP
  - [ ] Research best practices for Klipper plugin development

### Remaining / Deferred Targets
- See `docs/consolidation_roadmap.md` for full analysis and decision matrix.
- Near-term candidate (after LED design): `status-macros.cfg` (requires LED group abstraction first).
- Future design task: LED group variable mapping (`_LED_VARS`) before broad status macro consolidation.
- Re-evaluate `homing.cfg` for partial abstraction (sensorless current + probe sequencing) once other high-value targets complete.
