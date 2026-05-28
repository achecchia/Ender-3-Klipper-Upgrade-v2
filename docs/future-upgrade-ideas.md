# Future Upgrade Ideas

These are ideas worth keeping track of, but they are not part of the current active upgrade phase.

The current priority is to finish the hotend, cooling, and bed-stability upgrades, then commission and tune the printer cleanly.

---

## Important Future Goal

### Quiet Operation / Silent Electronics

Status: Important future goal.

Reason:

This Ender 3 has a Creality v4.2.2 mainboard, but it is the **non-silent** version. Reducing machine noise is an important long-term goal for this build.

Potential paths to quieter operation:

- Future silent mainboard upgrade, likely BTT SKR Mini E3-style or similar
- Quieter hotend, part-cooling, electronics, and PSU fans where appropriate
- Better fan control where supported
- Motion tuning that avoids unnecessary harshness without sacrificing reliability

Current decision:

Do not replace the board during the active hotend/cooling hardware phase. Keep the current 4.2.2 board until the printer is mechanically and thermally stable, then revisit quiet-operation upgrades as a focused phase.

---

## Low Priority / Optional

### Z-Axis Height Extender for Direct-Drive Clearance

Link: https://www.printables.com/model/178265-ender-3pro-z-axis-height-extender-for-direct-drive

Status: Not important right now.

Reason:
This may be useful later if the direct-drive setup, top-mounted filament sensor, or cable routing reduces usable Z height more than expected.

Current plan:
Do not install this preemptively. First complete the hotend/cooling upgrades, then physically test maximum safe Z travel.

Preferred first solution:
Reduce `position_max` in `printer.cfg` and match the reduced printable height in the dedicated OrcaSlicer profile.

Revisit only if:

- Safe usable Z height drops below an acceptable amount
- The direct-drive extruder or filament path interferes near the top of travel
- Tall prints become important for this machine

---

### BTT Mainboard Upgrade

Status: Future modernization idea, not part of the current phase.

Potential reasons to revisit later:

- Quieter stepper operation compared with the current non-silent Creality v4.2.2 board
- Software stepper-current control
- Cleaner fan control
- Independent dual-Z support
- More expansion ports
- Cleaner Klipper integration
- Better future wiring and serviceability

Preferred category if revisited:

- BTT SKR Mini E3-style board

Current decision:
Keep the Creality 4.2.2 board for now. It is working and is not the current bottleneck. Revisit later as part of a dedicated quiet-operation/electronics modernization phase.

---

### Dual-Z Upgrade

Status: Future possible upgrade, not soon.

Preferred direction:
A mechanically synchronized dual-Z setup may be the best fit for reliability and simplicity.

Revisit after:

- Hotend/cooling upgrades are complete
- Printer is tuned and reliable
- Any Z-related issue is confirmed as a real limitation

---

### WLED Printer Lighting

Status: Good future quality-of-life upgrade.

Reason:
Better lighting will improve webcam visibility and OctoEverywhere failure/spaghetti detection.

Possible presets:

- `PRINT_START` -> bright white / camera mode
- `PRINT_END` -> complete color or soft white
- `PAUSE` -> warning color
- `CANCEL_PRINT` -> red/error color
- `IDLE` -> dim/off

Current decision:
Do this later after the mechanical and thermal upgrade path is stable.

---

### Mainboard Power Switching

Status: Future refinement.

Goal:
Keep the Raspberry Pi powered on all the time while allowing the Creality mainboard and printer electronics to be switched off from Mainsail.

Preferred architecture:

```text
24V PSU always on
├── buck converter -> Raspberry Pi always on
└── DC-rated relay/MOSFET -> Creality mainboard 24V input
```

Important note:
A USB power blocker or data-only USB cable will be required to prevent the Raspberry Pi from backfeeding the mainboard over USB when the mainboard power is off.

Current decision:
Do this later, not during the hotend/cooling upgrade phase.
