# Config and Workflow Notes

This document keeps the detailed operational notes that used to live in the README.

The README should stay short and act as the front page. This file holds the deeper workflow, config-layout, backup, remote-access, and tuning-process details.

---

## Current Printer Platform

Base printer:

- Creality Ender 3
- Creality 4.2.2 mainboard, non-silent version
- Klipper firmware
- Raspberry Pi 3 B+ host connected to the printer mainboard over USB
- Fresh Raspberry Pi SD card rebuild after the original card failed
- Mainsail web interface
- Moonraker backend
- GitHub-backed Klipper config history

Mainboard / stepper behavior:

- Creality 4.2.2 board, non-silent version
- Stepper current is adjusted physically with Vref potentiometers on the board
- No active Klipper UART/TMC driver control is currently configured
- No `[tmc2209]` or equivalent driver sections are currently active in the live config
- Quiet operation is an important future goal, so a silent mainboard upgrade may be worth revisiting after the current hardware phase is stable

Firmware status:

- The Creality 4.2.2 MCU firmware was recompiled and reflashed during the 2026 resurrection phase.
- This cleared the Klipper `STEPPER_STEP_BOTH_EDGE` deprecated-code warning.
- See [`project-history.md`](project-history.md) for the recovery timeline.

---

## Raspberry Pi Power Setup

The Raspberry Pi is powered from the printer power system:

```text
Printer 24V PSU
    ↓
Buck converter stepped down to approximately 5.1V
    ↓
USB cable
    ↓
Raspberry Pi micro USB power input
```

This was done intentionally so the Pi is powered from the printer while still feeding power through the Pi's normal protected power input path, including the Pi's built-in fuse protection.

Future checks:

- Verify stable 5V behavior under heater/load transients
- Watch for Pi undervoltage warnings
- Watch for USB disconnects or webcam instability if a camera is added

---

## Filament Runout Sensor / Z Height Note

The filament runout sensor is mounted at the top of the printer.

Because of that, usable Z height may need to be reduced later to avoid physical interference near the top of travel. This is acceptable because this Ender 3 is not the primary printer.

Earlier AI-assisted troubleshooting notes identified `PA4` as the likely DET/runout pin, but this still needs validation against the live config, a trusted board pinout for the exact board revision, or direct Klipper sensor testing.

When final tuning is complete:

- Verify true safe maximum Z height
- Update `[stepper_z] position_max` in `printer.cfg` if needed
- Create a dedicated OrcaSlicer profile for this specific Ender 3
- Match OrcaSlicer printable volume to the safe firmware volume

---

## Configuration File Layout

The active Klipper files are backed up under:

```text
printer_data/config/
```

Important active files:

```text
printer_data/config/printer.cfg
printer_data/config/macros.cfg
printer_data/config/shell_command.cfg
printer_data/config/adxlmcu.cfg
```

`mainsail.cfg` may appear as a symlink on the printer:

```text
mainsail.cfg -> /home/pi/mainsail-config/mainsail.cfg
```

That file is part of the Mainsail install structure and may not be backed up like a normal local config file. This is expected.

---

## `ai_` File Convention

Files beginning with `ai_` are AI-generated proposed replacement files.

They are not active unless they are explicitly included by the live `printer.cfg`.

Current convention:

```text
printer.cfg        = active live file
ai_printer.cfg     = proposed AI-generated replacement
old_printer_*.cfg  = previous archived version
```

Before promoting an `ai_` file:

1. Run a backup
2. Rename the current active file to a dated `old_` version
3. Rename the `ai_` file to the active filename
4. Restart Klipper
5. Confirm the printer loads cleanly
6. Run another backup after validation

Recommended backup command before file swaps:

```gcode
update_git MESSAGE="backup before promoting ai config files"
```

---

## Backup Workflow

Klipper-Backup is installed and connected to this GitHub repository.

Manual backup is available from the Mainsail dashboard through the `update_git` macro/button.

Manual backup from Mainsail console:

```gcode
update_git
```

Manual backup with commit message:

```gcode
update_git MESSAGE="backup before CHC hotend install"
```

Manual backup from SSH:

```bash
~/klipper-backup/script.sh
```

The manual backup button works through:

```text
Mainsail button
    ↓
update_git macro
    ↓
gcode_shell_command
    ↓
Klipper-Backup script
    ↓
GitHub repository
```

---

## Remote Access / Monitoring Stack

Current remote and monitoring strategy:

```text
Mainsail       = local Klipper web UI
Moonraker      = Klipper API/backend
UniFi VPN      = secure full remote admin access
OctoEverywhere = cloud access, notifications, webcam/failure monitoring
GitHub         = config history and documentation
```

Use OctoEverywhere for:

- Print status
- Notifications
- Failure/spaghetti detection
- Quick remote viewing

Use UniFi VPN for higher-risk/admin tasks:

- SSH access
- Editing config files
- Updating Klipper/Moonraker
- Restarting services
- Troubleshooting network/system issues

Avoid exposing Moonraker or Mainsail directly to the public internet.

Optional local physical interface:

- An old Samsung phone may be mounted later.
- Preferred interface is Mainsail as a full-screen web app or Mobileraker.
- KlipperScreen was tested and removed because the phone/XServer path was unreliable and unnecessary for the Pi 3 B+.

---

## Current Tuning Philosophy

The printer is currently in a hardware transition phase.

Do not perform final tuning until all pending hardware upgrades are installed.

Do not finalize yet:

- Pressure advance
- Retraction
- Max volumetric flow
- Acceleration limits
- Speed limits
- OrcaSlicer performance profile
- Final input shaper validation

Reason:

The hotend, heatbreak, nozzle, cooling, and bed support changes will alter the printer's actual thermal and mechanical behavior. Tuning before those parts are installed would create temporary numbers that will need to be redone.

Current priority:

```text
Finish hardware installation
    ↓
Verify safe operation
    ↓
PID tune
    ↓
Extrusion sanity check
    ↓
Retraction tune
    ↓
Pressure advance tune
    ↓
Max flow characterization
    ↓
Speed/acceleration tuning
    ↓
OrcaSlicer profile creation
```

See [`punch-list.md`](punch-list.md) for the full staged checklist.

---

## Known Live Config Notes

The currently active `printer.cfg` is the working baseline, not necessarily the final cleaned version.

Known items to address later:

- Duplicate `[safe_z_home]` section should be cleaned
- `[gcode_arcs]` should use Klipper-style `resolution: 1.0` syntax
- Hotend `min_temp: -30` should be changed to a safer value such as `0`
- Macro stack should eventually be simplified
- Existing adaptive mesh/purge macro behavior should be reviewed before relying on it
- Final CHC hotend config will require PID tuning and likely max temp/thermistor review

These should be changed deliberately, with backups before and after.

---

## Hardware Fact-Check Notes

Key reminders from the AI-assisted audit:

- The final verified pancake stepper order is `Black, Green, Red, Blue`.
- The earlier `Black, Red, Blue, Green` order failed.
- Treat the runout sensor `PA4` note as a candidate until tested.
- Verify fan voltage before buying or wiring.
- Do not power a 12V fan directly from a Raspberry Pi 5V path.

More details are in [`hardware-notes-audited.md`](hardware-notes-audited.md).

---

## Related Printers / Context

Other printers available for comparison and production use:

- Elegoo Centauri Carbon
- Anycubic Kobra S1 with two ACE units

This Ender 3 is being maintained as a custom, known, long-term machine rather than the only production printer.
