# Project Leonardo

**Leonardo E3** is my long-term Creality Ender 3 Klipper rebuild.

The goal is to keep this printer reliable, understandable, serviceable, reasonably fast, and eventually quieter while upgrading it in deliberate stages. This is a project-car style machine, not the primary production printer.

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
- OctoEverywhere for remote status/notifications/failure monitoring
- UniFi VPN for secure full remote admin access

See [Upgrades](docs/upgrades.md) for installed part links, completed upgrades, and near-term planned hardware.

See [Configuration](docs/configuration.md) for machine architecture, wiring/config notes, software services, backup workflow, remote access, and calibration status.

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
- The top-mounted filament runout sensor may reduce usable Z height. Final safe Z height must be matched in both `printer.cfg` and the dedicated OrcaSlicer profile.
- The runout sensor `PA4` note is a candidate until validated against the live config, trusted board pinout, or direct Klipper sensor testing.
- The pancake extruder stepper's final verified wire order is documented in [Configuration](docs/configuration.md).
- Files beginning with `ai_` are proposed AI-generated replacements and are not active unless explicitly included by the live config.

---

## General Rule for This Build

Back up before changing anything important.

Do not chase tuning numbers until the hardware is stable.

Keep Leonardo E3 understandable, reversible, quiet where practical, and maintainable.
