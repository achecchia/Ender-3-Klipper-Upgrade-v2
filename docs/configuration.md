# Configuration

This document tracks Leonardo E3's system configuration: machine architecture, wiring/config-sensitive hardware notes, software stack, backup workflow, remote access, calibration status, and known live config items.

Physical parts and upgrade status live in [`upgrades.md`](upgrades.md). Long-term ideas live in [`future-ideas.md`](future-ideas.md).

---

## Machine Architecture

Base platform:

- Creality Ender 3
- Creality 4.2.2 mainboard, non-silent version
- Klipper firmware
- Raspberry Pi 3 B+ host connected to the printer mainboard over USB
- Mainsail web interface
- Moonraker backend/API layer
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

Current status:

- Buck converter setup is installed and working.
- Pi is powered through the normal micro USB input.
- Future mainboard power switching should keep the Pi powered independently of the Creality mainboard.

Future checks:

- Verify stable 5V behavior under heater/load transients
- Watch for Pi undervoltage warnings
- Watch for USB disconnects or webcam instability if a camera is added

---

## Hardware-Specific Configuration Notes

### Pancake Extruder Stepper Wiring

Status: completed and working.

Related hardware:

- BMG-style direct-drive conversion
- Pancake extruder stepper motor
- Creality 4.2.2 mainboard

Final verified wire order for the pancake extruder stepper connector:

```text
Black, Green, Red, Blue
```

The earlier order below did not work correctly:

```text
Black, Red, Blue, Green
```

Observed failed behavior:

- thump/noise
- no smooth extruder movement
- symptoms consistent with a phase mismatch

Motor coil pairs:

```text
Phase A: Black + Green
Phase B: Red + Blue
```

Final mapping:

```text
Pin 1: Black
Pin 2: Green
Pin 3: Red
Pin 4: Blue
```

Important note:

The Creality 4.2.2 board uses physical Vref potentiometers for stepper current. Do not add software current-control sections such as `[tmc2209 extruder]` unless the board is upgraded to hardware that supports that.

---

### BLTouch Wiring / Probe Configuration

Status: completed and working.

A previous Z-homing crash was resolved by correcting the BLTouch wiring orientation.

Known working wiring note:

```text
White = signal
Black = ground
```

Current working BLTouch-related configuration includes:

```ini
[bltouch]
sensor_pin: ^PA7
control_pin: PB0
x_offset: 0
y_offset: -28
stow_on_each_sample: True
probe_with_touch_mode: False
pin_up_touch_mode_reports_triggered: False
```

Do not change the BLTouch wiring or behavior unless a real fault is confirmed.

---

### Filament Runout Sensor / DET Port

Status: installed physically; final firmware behavior still needs validation.

Earlier AI-assisted notes identified `PA4` as the likely Creality 4.2.2 `DET` / runout pin:

```ini
switch_pin: PA4
```

Treat `PA4` as a candidate pin, not as fully verified, until confirmed by:

- the live working config
- a trusted pinout for the exact board revision
- direct Klipper sensor testing

Candidate config for later validation:

```ini
[filament_switch_sensor runout_sensor]
pause_on_runout: True
switch_pin: PA4
```

If the logic is backwards after testing, the pin may need inversion:

```ini
switch_pin: !PA4
```

Because the runout sensor is mounted at the top of the printer, final Z-height clearance must be checked before the slicer profile is finalized.

When final tuning is complete:

- Verify true safe maximum Z height
- Update `[stepper_z] position_max` in `printer.cfg` if needed
- Create a dedicated OrcaSlicer profile for this specific Ender 3
- Match OrcaSlicer printable volume to the safe firmware volume

---

### Mainboard Enclosure / Electrical Safety Note

Status: conservative documentation only.

A printed mainboard enclosure may be fine mechanically, but earlier AI-assisted grounding advice should not be treated as authoritative.

Project rule:

- Preserve the stock printer safety design unless there is a measured reason to change it.
- Do not add extra grounding or bonding wires based only on AI advice.
- If anything around the PSU, enclosure, or electronics bay is changed, verify the work before powering the printer.

---

### Fan Voltage Rule

Status: ongoing rule.

Do not assume fan voltage based on size, connector style, or AI recommendation.

Before using a fan, verify:

- voltage rating
- power source voltage
- current draw
- control method

Important lesson:

A 12V fan is not a correct direct replacement for a Raspberry Pi 5V-powered fan path.

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

## Installed Software / Services

### Klipper + Mainsail + Moonraker

Status: installed and working.

Notes:

- Klipper is running on the Raspberry Pi.
- Mainsail is the local web interface.
- Moonraker is the backend/API layer.

---

### Klipper-Backup

Status: installed and working.

Notes:

- Klipper configuration is backed up to this GitHub repository.
- Manual backup is available through the Mainsail `update_git` button.
- The backup script was patched so the Raspberry Pi only stages `printer_data/config/` and does not stage, delete, or overwrite repository documentation.
- Full recovery/prevention notes are documented in [`klipper-backup-docs-protection.md`](klipper-backup-docs-protection.md).

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

Current docs-protection rule:

```text
The Raspberry Pi owns printer_data/config/ backups.
GitHub/ChatGPT owns docs/ and README.md documentation edits.
```

Before reinstalling or updating Klipper-Backup, re-check [`klipper-backup-docs-protection.md`](klipper-backup-docs-protection.md).

---

### G-Code Shell Command Extension

Status: installed and working.

Notes:

- Required for the Mainsail `update_git` macro/button to trigger Klipper-Backup from the UI.
- `shell_command.cfg` is now included by the active `printer.cfg`.

---

### OctoEverywhere for Klipper

Status: installed.

Notes:

- Installed for cloud access, print notifications, webcam/failure monitoring, and spaghetti detection support.
- UniFi VPN remains the preferred path for higher-risk admin tasks like SSH, config edits, and Klipper/Moonraker updates.

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

## Calibration / Commissioning Status

### Bed Mesh

Status: completed baseline mesh exists.

Notes:

- A saved bed mesh exists in the live `printer.cfg` `SAVE_CONFIG` block.
- Mesh should be regenerated after silicone bed mounts and hotend/toolhead hardware changes are complete.

---

### Input Shaping

Status: completed baseline input shaping data exists.

Current saved values in the live config:

```ini
shaper_type_y = mzv
shaper_freq_y = 33.4
shaper_type_x = mzv
shaper_freq_x = 74.0
```

Notes:

- These are believable for the current Ender 3 bedslinger setup.
- Revalidate later only if the final toolhead mass changes significantly.

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

- Final CHC hotend config will require PID tuning and likely max temp/thermistor review
- Macro stack should eventually be simplified
- Existing adaptive mesh/purge macro behavior should be reviewed before relying on it

Items already cleaned:

- Duplicate `[safe_z_home]` section removed
- `[gcode_arcs]` syntax corrected to `resolution: 1.0`
- Hotend `min_temp` changed from unsafe `-30` to `0`
- Z `position_min` formatting cleaned to `position_min: -6`

These should be changed deliberately, with backups before and after.
