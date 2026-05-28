# Hardware Notes

This file captures hardware-specific details that are easy to forget but important for future troubleshooting.

---

## Pancake Extruder Stepper Wiring

Status: Completed and working.

Related upgrade:

- BMG-style direct-drive conversion
- Pancake extruder stepper motor
- Creality 4.2.2 mainboard

### Why this note matters

The pancake extruder motor did not work correctly with the first assumed wire order. During testing, the motor made a thump/noise but did not rotate correctly. That behavior pointed to the stepper phases being wired incorrectly for the Creality 4.2.2 E-stepper port.

The motor has two coil/phase pairs:

```text
Phase A: Black + Green
Phase B: Red + Blue
```

The Creality 4.2.2 E-stepper port expects the phases grouped together as an inline layout:

```text
Pin 1: A+
Pin 2: A-
Pin 3: B+
Pin 4: B-
```

### Final working wire order

Looking at the connector from left to right, the final working order is:

```text
Black, Green, Red, Blue
```

Mapped electrically:

```text
Pin 1: Black  -> Phase A
Pin 2: Green  -> Phase A
Pin 3: Red    -> Phase B
Pin 4: Blue   -> Phase B
```

### Failed / problematic behavior to remember

The wrong wiring produced symptoms like:

- A single thump
- Motor noise without useful movement
- No smooth extruder motion
- Phase mismatch behavior during `STEPPER_BUZZ STEPPER=extruder`

Do not blindly swap stepper wires while powered. Power down before moving pins.

### Validation command

Use Klipper's stepper test command after verifying the connector wiring:

```gcode
STEPPER_BUZZ STEPPER=extruder
```

Expected behavior:

- The motor should physically pulse/jiggle cleanly.
- A thump or hum without movement suggests a phase/wiring issue.

### Notes for future work

- The Creality 4.2.2 board uses physical Vref potentiometers for stepper current.
- Do not add `[tmc2209 extruder]` or software `run_current` settings unless the electronics are upgraded to a board with UART-controlled stepper drivers.
- If the extruder runs backward after wiring is correct, fix direction in Klipper with the `dir_pin` invert flag rather than rewiring the phases.
- After the final hotend is installed, re-check extrusion behavior under heat and load.

---

## BLTouch Wiring Note

Status: Completed and working.

A previous Z-homing crash was resolved by correcting BLTouch wiring polarity at the mainboard.

Important note:

```text
White = signal
Black = ground
```

The BLTouch is currently working and should not be rewired unless there is a confirmed fault.

---

## Filament Runout Sensor / DET Port

Status: Installed / to be validated during final commissioning.

The Creality 4.2.2 board has a 3-pin filament detection port labeled `DET`.

The DET port maps to this Klipper pin:

```ini
switch_pin: PA4
```

Typical Klipper config:

```ini
[filament_switch_sensor runout_sensor]
pause_on_runout: True
switch_pin: PA4
runout_gcode:
    M117 Filament runout detected!
    PAUSE
insert_gcode:
    M117 Filament inserted
```

If the sensor logic is backwards, invert the pin:

```ini
switch_pin: !PA4
```

Notes:

- The runout sensor is mounted at the top of the printer.
- Final Z-height clearance must be checked because the sensor and filament path may reduce safe usable Z travel.
- Match final firmware Z max in OrcaSlicer after validation.

---

## Mainboard Enclosure / Grounding Note

Status: Important safety context.

A 3D-printed plastic mainboard enclosure is acceptable for housing the Creality 4.2.2 board, but it does not provide chassis grounding by itself.

Important grounding rules:

1. The green/yellow AC earth ground wire must remain securely attached to the PSU earth-ground terminal.
2. The metal PSU chassis must remain properly bonded/bolted to the printer frame.
3. Do not rely on the printed electronics enclosure for electrical grounding.
4. Do not add random ground wires unless there is a specific measured reason and a safe grounding plan.

Why this matters:

- The PSU/frame earth bond is the important safety ground path.
- A plastic enclosure can protect the board mechanically, but it is not a substitute for the printer's earth grounding.

---

## Fan / Voltage Verification Rule

Status: Ongoing rule.

Do not assume fan voltage based on size or connector style.

Before wiring any fan, verify:

- Fan voltage rating
- Power source voltage
- Current draw
- Control method: always-on, PWM, MOSFET-switched, GPIO-controlled, or board-controlled
- Grounding/common-ground requirements

Important lesson:

A 12V fan should not be treated as a correct choice for the Raspberry Pi's 5V power/GPIO path. If a fan is powered from the Pi side, it needs to be 5V-rated or driven through an appropriate external circuit.

---

## Safety Reminder

When working on stepper wiring, BLTouch wiring, heater wiring, fan wiring, or mains-side wiring:

1. Power the printer off.
2. Unplug AC power if working inside the electronics bay.
3. Confirm wiring before powering back on.
4. Test one subsystem at a time.
5. Back up config before and after any related firmware/config change.
