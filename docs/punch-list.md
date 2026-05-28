# Ender 3 Klipper Upgrade Punch List

This punch list tracks the upgrade path for the Ender 3 Klipper build.

The guiding rule for this machine:

```text
One controlled change, one backup, one known-good state at a time.
```

---

## Phase 0 - Current Stable Baseline

Goal: keep the printer stable while waiting for hardware parts. Do not tune yet.

### Done / Mostly Done

- Klipper installed and running
- Mainsail installed
- Raspberry Pi powered from printer 24V PSU through buck converter to about 5.1V
- Raspberry Pi powered through micro USB input to retain Pi fuse protection
- Creality 4.2.2 board connected to Pi over USB
- BLTouch installed and working
- BMG-style direct drive installed
- Pancake extruder motor installed
- Ductinator duct installed/planned
- Magnetic PEI sheet installed
- Filament runout sensor mounted at top of printer
- GitHub backup repo created
- Klipper-Backup installed
- `update_git` Mainsail button working
- OctoEverywhere installed
- README documentation started
- `ai_` config files staged as inactive future replacements

### Do Not Tune Yet

Do not finalize any of these until the hotend/cooling hardware is installed:

- Pressure advance
- Retraction
- Max volumetric flow
- Speed limits
- Acceleration limits
- OrcaSlicer profile
- Input shaper revalidation

---

## Phase 1 - Before Installing New Hardware

Goal: preserve current state and prepare for clean rollback.

### High Priority

1. Run backup before touching hardware.

```gcode
update_git MESSAGE="backup before CHC hotend and cooling hardware install"
```

2. Take photos before disassembly.

Photograph:

- Hotend wiring
- BLTouch wiring
- Fan wiring
- Mainboard wiring
- Toolhead layout
- Cable routing
- Current duct/fan arrangement

3. Label wires before replacing hotend/fans.

Especially:

- Heater cartridge wires
- Thermistor wires
- Hotend fan
- Part cooling fan/fans
- BLTouch wires
- Extruder motor cable

4. Confirm backup repo has newest files.

Check GitHub after pressing `update_git`.

---

## Phase 2 - Install Pending Hardware

Goal: complete all physical upgrades before tuning.

### High Priority Hardware

#### CHC Hotend / Heater System

Pending part:

- Trianglelab 115W High Power CHC Pro kit

Important checks:

- Correct heater wiring
- Correct thermistor connection
- No exposed heater leads
- Proper strain relief
- No wires touching heater block
- Hotend fan aimed correctly
- Silicone sock installed if applicable

#### Bi-Metal Heatbreak

Pending part:

- Mellow all-metal bi-metal CR10/Crazy heatbreak

Important checks:

- Proper hot-tightening procedure
- No nozzle/heatbreak gap
- No filament leak path
- Heatbreak seated correctly
- PTFE no longer treated like a stock PTFE-lined hotend

#### Upgraded Nozzle

Pending part:

- Mellow Gold DLC hardened steel/copper bimetal GHC V6 nozzle

Important checks:

- Correct nozzle type/length for hotend
- Proper hot tightening
- Verify nozzle does not crash into duct or affect probe clearance

#### Dual Blower Fans / Duct

Important checks:

- Correct voltage
- Correct polarity
- Fan airflow direction
- No fan blade rubbing
- Wires secured away from hotend and motion
- Part cooling responds to `M106`

#### Silicone Bed Mounts

Important checks:

- Bed not over-compressed
- Bed cable strain relief still good
- Screws secure
- BLTouch can still probe safely
- Re-run screw tilt adjust afterward

---

## Phase 3 - First Power-On After Hardware Install

Goal: safely verify the printer before heating aggressively.

### Critical Safety Checks

1. Power on with heaters off.

Confirm:

- Pi boots
- Mainsail connects
- MCU connects
- BLTouch initializes
- Fans behave as expected
- No smoke/smell/overheating
- No unexpected hotend heating

2. Verify thermistor reading at room temp.

Before heating, hotend should read roughly room temperature.

Stop and fix before heating if hotend reads something absurd like:

- `-30`
- `0`
- `300`
- fluctuating wildly

3. Fix unsafe config before heating.

In `printer.cfg`, change:

```ini
min_temp: -30
```

to:

```ini
min_temp: 0
```

Also clean duplicate safe Z home:

```ini
[safe_z_home]
[safe_z_home]
```

to a single:

```ini
[safe_z_home]
```

And change:

```ini
[gcode_arcs]
resolution = 1.0
```

to:

```ini
[gcode_arcs]
resolution: 1.0
```

4. Backup after basic config cleanup.

```gcode
update_git MESSAGE="config cleanup before first CHC heat test"
```

---

## Phase 4 - First Heat Test

Goal: confirm thermal control before printing.

### Critical

1. Heat slowly and watch behavior.

Start with a moderate temperature:

```gcode
M104 S150
```

Watch:

- Temperature rises smoothly
- No runaway
- No sudden spikes
- No heater errors
- No smoke/smell
- Hotend fan is running

Then cool down:

```gcode
M104 S0
```

2. PID tune hotend.

```gcode
PID_CALIBRATE HEATER=extruder TARGET=220
SAVE_CONFIG
```

Then back up:

```gcode
update_git MESSAGE="hotend PID tuned after CHC install"
```

3. PID tune bed if bed hardware was disturbed.

```gcode
PID_CALIBRATE HEATER=heater_bed TARGET=60
SAVE_CONFIG
```

Then back up:

```gcode
update_git MESSAGE="bed PID checked after silicone mount install"
```

---

## Phase 5 - Mechanical Validation

Goal: make sure the machine moves and probes safely.

### High Priority

1. Home all axes.

```gcode
G28
```

Watch carefully:

- X homes correctly
- Y homes correctly
- BLTouch deploys
- Z homes safely
- No nozzle crash
- Probe hits expected location

2. Check BLTouch offsets and clearance.

Current known offsets:

```ini
x_offset: 0
y_offset: -28
```

Leave them alone unless the new duct/hotend physically changes probe position.

3. Run screws tilt adjust.

```gcode
SCREWS_TILT_CALCULATE
```

4. Run new bed mesh.

```gcode
BED_MESH_CALIBRATE
SAVE_CONFIG
```

Back up:

```gcode
update_git MESSAGE="new bed mesh after hardware install"
```

---

## Phase 6 - Extrusion Commissioning

Goal: confirm the new hotend/extruder path works before performance tuning.

### High Priority

1. Load filament manually.

Check:

- Smooth feed into top-mounted runout sensor
- Smooth path into BMG
- No tight bends
- No rubbing
- No filament drag from sensor mount

2. Basic extrusion test.

Heat to PLA temperature, likely:

```gcode
M109 S220
```

Then extrude slowly:

```gcode
G91
G1 E25 F60
G1 E25 F120
G90
```

Watch for:

- Smooth extrusion
- No clicking
- No grinding
- No inconsistent flow
- No leaking around nozzle/heatbreak

3. Verify extruder motor temperature.

After testing or printing later:

- Warm is fine
- Too hot to touch is not fine
- Skipping means Vref, tension, or hotend resistance may need review

Do not adjust Vref unless symptoms appear.

---

## Phase 7 - Filament Sensor / Z Clearance

Goal: confirm the top-mounted runout sensor does not interfere with usable Z.

### Important

1. Manually inspect upper Z travel.

Move Z upward carefully and watch:

- Filament sensor mount clearance
- Filament path clearance
- Bowden/PTFE guide clearance if used
- Toolhead cable clearance
- Gantry clearance

2. Set real safe Z max.

Current:

```ini
position_max: 250
```

If needed, reduce to something safe like:

```ini
position_max: 235
```

or whatever the real measured clearance allows.

3. Document final Z max.

This must later match OrcaSlicer.

Back up:

```gcode
update_git MESSAGE="set safe Z max after filament sensor clearance check"
```

---

## Phase 8 - Slicer Baseline

Goal: get a conservative OrcaSlicer profile working before speed tuning.

Create a dedicated OrcaSlicer printer profile for this exact machine.

Initial conservative values:

- Direct drive retraction baseline: `0.6-1.0 mm`
- Retraction speed: `25-40 mm/s`
- Conservative print speed first
- Conservative acceleration first
- Correct bed size
- Correct reduced Z height if needed
- Klipper-style start/end g-code
- Use `PRINT_START` / `PRINT_END` only after macros are cleaned and trusted

Do not copy a Bowden Ender 3 profile directly.

---

## Phase 9 - Tuning Phase

Goal: tune in the correct order after hardware is final.

Correct order:

1. Rotation distance sanity check
2. Retraction tuning
3. Pressure advance tuning
4. Max volumetric flow testing
5. Speed and acceleration tuning
6. Input shaper revalidation if toolhead mass changed significantly

Current temporary values:

```ini
rotation_distance: 7.71
pressure_advance: 0.04
```

Current saved input shaper values:

```ini
shaper_type_y = mzv
shaper_freq_y = 33.4
shaper_type_x = mzv
shaper_freq_x = 74.0
```

Target later, only after validation:

```text
Acceleration: 3000-6000 mm/s² realistic bedslinger range
Print speeds: 150-250 mm/s depending material, line width, and flow
```

---

## Phase 10 - Config Cleanup

Goal: simplify the live config so it is maintainable.

### Promote Cleaned `ai_` Files Carefully

Before promoting:

```gcode
update_git MESSAGE="backup before promoting ai config files"
```

Rename current active file:

```text
printer.cfg -> old_printer_YYYY-MM-DD_before_cleanup.cfg
```

Then:

```text
ai_printer.cfg -> printer.cfg
```

Restart Klipper, verify, then:

```gcode
update_git MESSAGE="promoted cleaned printer config"
```

### Simplify Macro Stack

Preferred final macro set:

- `PRINT_START`
- `PRINT_END`
- Simple purge routine
- Safe pause/resume/cancel
- Optional `G29`
- Optional light/WLED macros later

Avoid unnecessary Voron/Klicky/QGL-style macro logic.

---

## Phase 11 - Remote / Quality-of-Life Enhancements

Already done:

- OctoEverywhere installed
- UniFi VPN available
- GitHub backup working
- Mainsail backup button working

Later optional enhancements:

- WLED printer lighting
- Mainsail-controlled mainboard power switching
- Dual-Z upgrade
- More refined macros
- Full tuning log

---

## Priority Summary

### Critical Before Printing After Hardware Install

1. Backup current state
2. Install hardware carefully
3. Verify thermistor at room temp
4. Fix `min_temp: -30`
5. PID tune hotend
6. Confirm BLTouch/home safety
7. Confirm extrusion path
8. Run bed mesh

### Important Before Tuning

1. Silicone bed mount leveling
2. Z height clearance check
3. Dedicated OrcaSlicer baseline profile
4. Retraction tuning
5. Pressure advance tuning
6. Max flow testing

### Nice-To-Have Later

1. WLED lighting
2. Mainsail-controlled mainboard power
3. Dual-Z
4. More refined macros
5. Full documentation/tuning log

### Do Not Do Yet

- Final speed tuning
- Final flow testing
- Dual-Z
- Mainboard power switching
- WLED automation
- Aggressive macro changes during hardware transition

Current best move:

```text
Finish hardware -> verify safety -> tune methodically -> document/backup each step
```
