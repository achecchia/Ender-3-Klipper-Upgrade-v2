# Project Leonardo Printer Notes

This file is the clean, consolidated note stash for Anthony's Ender 3 / Project Leonardo build.

Keep this on the printer in:

```text
~/printer_data/config/README_PROJECT_LEONARDO.md
```

These notes are not Klipper config. They are here as a sanity map so future-you does not have to dig through old chat logs like a raccoon in a firmware dumpster.

---

## Active Config Files

- `printer.cfg` — main Klipper hardware config.
- `macros.cfg` — print start/end, manual mesh helper, prime line, PID helpers, parking, and test macros.
- `mainsail.cfg` — Mainsail client helpers, pause/resume/cancel plumbing.
- `moonraker.conf` — Moonraker API and service configuration.
- `crowsnest.conf` — webcam service configuration.
- `sonar.conf` — WiFi keepalive configuration.
- `shell_command.cfg` — shell command helpers if used.
- `adxlmcu.cfg` — ADXL/input-shaper helper config; include only while using the accelerometer.

---

## Config Ownership Rules

This is the “do not make firmware lasagna” section.

### `printer.cfg` owns:

- MCU serial.
- Printer motion limits.
- Stepper pins.
- Extruder hardware.
- Bed heater hardware.
- BLTouch config.
- Bed mesh definition.
- Screws tilt adjust.
- Fan pins.
- LCD/display pins.
- `[exclude_object]`.

### `macros.cfg` owns:

- `PRINT_START`
- `PRINT_END`
- `CREATE_BED_MESH`
- `G29`
- `PRIME_LINE`
- `PID_HOTEND`
- `PID_BED`
- `HEAT_SOAK_BED`
- Parking helpers.
- Motion/probe test helpers.

### `mainsail.cfg` owns:

- `[pause_resume]`
- `[display_status]`
- `[respond]`
- `CANCEL_PRINT`
- `PAUSE`
- `RESUME`
- `_TOOLHEAD_PARK_PAUSE_CANCEL`

Do not define the same section in two files. Klipper hates duplicate sections, and honestly, fair enough.

---

## Important Design Choices

- `[exclude_object]` lives in `printer.cfg`.
- `macros.cfg` does **not** define `[exclude_object]`.
- `PRINT_START` does **not** run a bed mesh every print.
- `PRINT_START` tries to load saved mesh profile `default` if it exists.
- If no saved mesh exists yet, `PRINT_START` continues without mesh.
- Adaptive mesh is intentionally disabled for reliability.
- Adaptive purge is intentionally disabled for reliability.
- Mesh creation is a manual/monthly tuning task.
- `PRINT_END` uses a small `E-3` retract for the direct-drive setup.
- Creality 4.2.2 board is treated as standalone/Vref-adjusted driver mode.
- TMC/UART sections are intentionally not added.
- Hotend fan is stock-style 24V always-on 4010 axial unless upgraded later.

---

## Before Replacing Config Files on the Pi

1. Back up the current config folder.
2. Upload replacement files to:

```text
~/printer_data/config
```

3. Restart Klipper.
4. If Klipper errors, restore the backup and copy the exact error text.

Recommended backup command from SSH:

```bash
cd ~/printer_data
tar -czf ~/printer_config_backup_$(date +%Y-%m-%d_%H-%M).tar.gz config
```

---

## Include Checklist

`printer.cfg` should include:

```ini
[include mainsail.cfg]
[include shell_command.cfg]
[include macros.cfg]
```

Optional ADXL include only while using the accelerometer:

```ini
#[include adxlmcu.cfg]
```

Leave it commented out for normal printing unless you are actively doing input-shaper work.

---

## OrcaSlicer G-code

### Start G-code

```gcode
M117
M190 S0
M109 S0
PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]
```

### End G-code

```gcode
PRINT_END
```

---

## Bed Mesh Policy

The printer should **not** create a bed mesh on every print.

Normal prints should:

1. Heat bed.
2. Home.
3. Load saved mesh profile `default` if it exists.
4. Continue without mesh if no saved profile exists yet.
5. Heat nozzle.
6. Prime.
7. Print.

### Create the first mesh after hardware work

Once the printer is physically finished and safe:

```gcode
CREATE_BED_MESH TEMP=60 SOAK=300 SAVE=1
```

This should:

- Heat bed to 60°C.
- Heat soak for 300 seconds.
- Home.
- Run `BED_MESH_CALIBRATE`.
- Save the profile as `default`.
- Run `SAVE_CONFIG`.
- Restart Klipper.

### Monthly mesh refresh

Run the same command monthly or whenever major hardware changes happen:

```gcode
CREATE_BED_MESH TEMP=60 SOAK=300 SAVE=1
```

Also refresh the mesh after:

- Changing bed surface.
- Changing probe mount.
- Changing toolhead mount.
- Changing hotend/nozzle setup.
- Changing silicone spacers/bed hardware.
- Any crash or major mechanical adjustment.

---

## CHC / Hotend Upgrade Notes

Before first heat-up after the CHC/hotend install:

- Verify heater cartridge is fully seated.
- Verify thermistor is fully seated.
- Verify hotend fan spins.
- Verify thermistor reads room temperature.
- Confirm the thermistor type.

Current config assumes:

```ini
sensor_type: EPCOS 100K B57560G104F
```

If the new CHC hotend uses a different thermistor, update `sensor_type` before heating.

After CHC/heatbreak/nozzle install, retune:

- Hotend PID.
- Bed PID if bed hardware changed.
- Extruder rotation distance.
- Retraction.
- Pressure advance.
- Max flow.
- Z offset.
- Bed mesh.

---

## Project Leonardo Build Checklist

### Before Power-On After Hardware Work

- [ ] Heater cartridge fully seated.
- [ ] Thermistor fully seated.
- [ ] Hotend fan connected.
- [ ] Part cooling fan connected.
- [ ] BLTouch connected.
- [ ] No bare wires exposed.
- [ ] Toolhead wiring strain relief installed.
- [ ] X carriage moves freely by hand.
- [ ] Bed moves freely by hand.
- [ ] Z axis moves freely by hand.
- [ ] Belt paths are clear.
- [ ] No wires can hit fans, belts, pulleys, bed, or wheels.

### First Power-On

- [ ] Hotend reads room temperature.
- [ ] Bed reads room temperature.
- [ ] Hotend fan spins.
- [ ] Part cooling fan can be controlled.
- [ ] BLTouch self-test passes.
- [ ] No smoke.
- [ ] No burning smell.
- [ ] No buzzing/clicking nonsense.
- [ ] Emergency power-off plan is obvious.

### First Klipper Checks

- [ ] `FIRMWARE_RESTART` is clean.
- [ ] `QUERY_ENDSTOPS` works.
- [ ] X endstop triggers correctly.
- [ ] Y endstop triggers correctly.
- [ ] BLTouch/probe triggers correctly.
- [ ] `STEPPER_BUZZ STEPPER=stepper_x`
- [ ] `STEPPER_BUZZ STEPPER=stepper_y`
- [ ] `STEPPER_BUZZ STEPPER=stepper_z`

---

## First Motion / Probe Tests

Run these carefully after the printer is assembled.

### Endstop check

```gcode
QUERY_ENDSTOPS
```

Manually trigger X/Y/probe and check that Klipper sees the change.

### Stepper buzz

```gcode
STEPPER_BUZZ STEPPER=stepper_x
STEPPER_BUZZ STEPPER=stepper_y
STEPPER_BUZZ STEPPER=stepper_z
```

### Probe test

```gcode
TEST_PROBE
```

### Safe movement test

```gcode
TEST_SPEED_SAFE
```

Do not run aggressive speed tests until belts, wheels, wiring, and limits are verified.

---

## Tuning Log

Use this section as your road notes. Future-you will want these numbers.

### Hotend PID

Date:
Hardware:
Target temp:
Kp:
Ki:
Kd:

Command:

```gcode
PID_HOTEND TEMP=220
SAVE_CONFIG
```

### Bed PID

Date:
Target temp:
Kp:
Ki:
Kd:

Command:

```gcode
PID_BED TEMP=60
SAVE_CONFIG
```

### Probe Accuracy

Date:
Surface:
Samples:
Range:
Std dev:

Command:

```gcode
TEST_PROBE
```

### Z Offset

Date:
Nozzle:
Build plate:
Final z_offset:

Notes:

```text
Redo Z offset after hotend/nozzle/probe mount changes.
```

### Extruder Rotation Distance

Date:
Requested extrusion:
Actual extrusion:
Final rotation_distance:

Notes:

```text
Current baseline is rotation_distance: 7.71
Recheck after final extruder/hotend setup.
```

### Retraction

Date:
Filament:
Nozzle:
Temp:
Retraction distance:
Retraction speed:

### Pressure Advance

Date:
Filament:
Nozzle:
Layer height:
Final pressure_advance:

Notes:

```text
Current baseline is pressure_advance: 0.04
Retune after hotend/nozzle/filament profile is finalized.
```

### Max Flow

Date:
Filament:
Nozzle:
Temp:
Max reliable flow:

### Input Shaper

Date:
Toolhead config:
X shaper:
Y shaper:
X frequency:
Y frequency:

---

## Git / Backup Save Points

After clean config changes:

```bash
cd ~/printer_data/config
git status
git add .
git commit -m "Update Project Leonardo config notes"
git push
```

Before major hardware changes, make a tag:

```bash
git tag stable-before-hotend-upgrade
git push origin stable-before-hotend-upgrade
```

That is your save point before the boss fight.

---

## Deletable AI Draft Files

Once the consolidated live files are installed and Klipper restarts cleanly, these old draft/reference files can be deleted:

- `ai_macros.cfg`
- `ai_mainsail.cfg`
- `ai_printer.cfg`
- `ai_README_FIRST.txt`

Keep this consolidated note file instead.

---

## Quick Troubleshooting Notes

### If Klipper complains about duplicate sections

Look for duplicate definitions across:

- `printer.cfg`
- `macros.cfg`
- `mainsail.cfg`

Common duplicate goblins:

```ini
[pause_resume]
[display_status]
[respond]
[exclude_object]
[gcode_macro PAUSE]
[gcode_macro RESUME]
[gcode_macro CANCEL_PRINT]
```

### If prints start without mesh

That is expected until a saved `default` mesh exists.

Create one with:

```gcode
CREATE_BED_MESH TEMP=60 SOAK=300 SAVE=1
```

### If mesh load fails after deleting old mesh

Run:

```gcode
CREATE_BED_MESH TEMP=60 SOAK=300 SAVE=1
```

### If hotend temperature looks wrong

Stop. Do not heat. Verify thermistor type and wiring first.

---

## Final Philosophy

Reliability first. Fancy later.

Get the machine safe, predictable, and repeatable. Then tune it like a guitar before a show.
