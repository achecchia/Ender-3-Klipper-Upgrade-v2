# Project Leonardo

**Leonardo E3** is my long-term Creality Ender 3 Klipper rebuild.

The goal is to keep this printer reliable, understandable, serviceable, reasonably fast, and eventually quieter while upgrading it in deliberate stages. Think of it more like an old control system / rack rebuild than a one-off printer mod: clean up the architecture, document the topology, preserve rollback points, and commission it properly after the hardware is stable.

---

## Project Snapshot

Printer name:

- **Leonardo E3**

Base machine:

- Creality Ender 3
- Creality v4.2.2 mainboard, **non-silent version**
- Raspberry Pi 3 B+ running Klipper / Mainsail / Moonraker
- GitHub-backed Klipper config history

Current phase:

- Hardware transition / commissioning phase
- Do not perform final tuning until the pending hotend, cooling, and bed-stability upgrades are installed and validated

---

## Quick Links

- [Punch list](docs/punch-list.md)
- [Upgrades](docs/upgrades.md)
- [Configuration](docs/configuration.md)
- [AI config review - 2026-05-29](docs/ai-config-review-2026-05-29.md)
- [Klipper-Backup docs protection](docs/klipper-backup-docs-protection.md)
- [Project history](docs/project-history.md)
- [Future ideas](docs/future-ideas.md)
- Active Klipper configs: [`printer_data/config/`](printer_data/config/)

---

## Project Goals

- Keep the printer reliable and maintainable
- Improve performance beyond stock Ender 3 behavior
- Make the machine quieter over time
- Avoid unnecessary complexity
- Preserve known-good configuration history
- Tune only after hardware changes are complete
- Create a dedicated OrcaSlicer profile after final hardware commissioning

---

## Current Baseline

Already installed or configured:

- Klipper, Mainsail, and Moonraker
- Raspberry Pi printer-powered setup through a buck converter
- Creality v4.2.2 non-silent mainboard running Klipper firmware
- BLTouch
- BMG-style direct-drive extruder conversion
- Pancake extruder stepper motor with corrected wiring
- Ductinator cooling duct
- Magnetic PEI build plate
- Top-mounted filament runout sensor
- Belt tensioner
- Saved bed mesh and input shaping baseline data
- Klipper-Backup with working `update_git` macro/button
- `PULL_FROM_GIT` Mainsail macro/button for pulling GitHub-side docs/config changes down to the Pi before backups
- OctoEverywhere for remote status/notifications/failure monitoring
- UniFi VPN for secure full remote admin access

See [Upgrades](docs/upgrades.md) for installed part links, completed upgrades, and near-term planned hardware.

See [Configuration](docs/configuration.md) for machine architecture, wiring/config notes, software services, backup workflow, remote access, and calibration status.

See [AI Config Review - 2026-05-29](docs/ai-config-review-2026-05-29.md) for the latest Klipper-focused AI config cleanup summary.

---

## Pending Hardware Before Final Tuning

Parts planned for installation before final tuning:

- Trianglelab 115W CHC Pro ceramic heating core kit
- Mellow all-metal bi-metal heatbreak
- Mellow Gold DLC hardened steel / copper bimetal nozzle
- Dual 24V part-cooling fans for the Ductinator
- Silicone bed mounts/spacers to replace the bed springs

After those are installed, the printer should go through PID tuning, extrusion checks, bed mesh, retraction tuning, pressure advance, max-flow testing, speed/acceleration tuning, and OrcaSlicer profile creation.

See [Punch List](docs/punch-list.md) for the staged process.

---

## Future Plans

Longer-term ideas live in [Future Ideas](docs/future-ideas.md).

Examples include quiet-operation upgrades, lighting ideas, motion-system ideas, power-switching ideas, cosmetic ideas, and anything else worth tracking later.

---

## Important Reminders

- The installed Creality v4.2.2 board is the **non-silent** version, so noise reduction is an important future goal.
- Leonardo E3 is currently headless. The stock screen is removed for the normal/final build, with Mainsail/Moonraker as the primary interface.
- The stock screen may be reinstalled temporarily for service/troubleshooting if truly useful, but it is not part of the intended final setup.
- The top-mounted filament runout sensor may reduce usable Z height. Final safe Z height must be matched in both `printer.cfg` and the dedicated OrcaSlicer profile.
- The runout sensor `PA4` note is a candidate until validated against the live config, trusted board pinout, or direct Klipper sensor testing.
- The pancake extruder stepper's final verified wire order is documented in [Configuration](docs/configuration.md).
- The old `ai_` draft config files were removed after the AI-assisted config consolidation. Treat the live config files and `printer_data/config/README_PROJECT_LEONARDO.md` as the current printer-side references.
- If GitHub-side docs/config files are changed, press `PULL_FROM_GIT` in Mainsail before pressing `update_git`.

---

## General Rule for This Build

Back up before changing anything important.

Do not chase tuning numbers until the hardware is stable.

Keep Leonardo E3 understandable, reversible, quiet where practical, and maintainable.
