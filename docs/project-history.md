# Project History

This document captures important historical context from the Ender 3 resurrection and upgrade process.

It is not a step-by-step setup guide. It is a record of what happened, what was fixed, and what should be remembered later.

---

## 2026-05 - Fresh SD Card / Klipper Resurrection

Status: Completed.

The Ender 3 had been flashed with Klipper roughly two years earlier and then sat on the bench for a long period.

When the project restarted, the old Raspberry Pi 3 B+ SD card was dead. A new 64GB SD card was prepared and Klipper/Mainsail were rebuilt on the Raspberry Pi.

Initial problem after the fresh install:

```text
Section 'gcode_shell_command update_git_script' is not a valid config section
```

Cause:

- The restored Klipper config referenced `gcode_shell_command` sections.
- The fresh Raspberry Pi image did not yet have the G-Code Shell Command extension installed.
- Stock Klipper does not recognize `[gcode_shell_command ...]` sections without that extra module.

Temporary workaround used during resurrection:

```ini
# [include shell_command.cfg]
```

This allowed Klipper to boot while the missing extension / backup tooling was sorted out.

Current status:

- G-Code Shell Command is now installed again.
- `shell_command.cfg` is included again.
- The `update_git` Mainsail button is working.

---

## 2026-05 - KIAUH Was Not Present Initially

Status: Resolved.

On the fresh Raspberry Pi image, `~/kiauh` did not exist.

Observed commands:

```bash
cd ~/kiauh && ./kiauh.sh
./kiauh/kiauh.sh
```

Both failed at first because KIAUH was not installed.

KIAUH was later installed and used for extension management.

Why this matters:

- Some old services/extensions were referenced by config files but were not installed on the new SD card.
- Future troubleshooting should not assume every previously installed helper is present after an SD card rebuild.

---

## 2026-05 - MCU Firmware Update

Status: Completed.

After updating the Raspberry Pi host side to newer Klipper, Mainsail reported a warning that the MCU firmware had deprecated code and was missing this feature:

```text
STEPPER_STEP_BOTH_EDGE
```

The machine had:

```text
Mainboard: Creality v4.2.2
Older MCU firmware: v0.12.0-era
Newer Klipper host: v0.13.0-era
```

The v4.2.2 board was recompiled and reflashed using the Creality 32-bit SD-card flashing method.

Important firmware build context for Creality v4.2.2:

```text
Micro-controller Architecture: STMicroelectronics STM32
Processor model: STM32F103
Bootloader offset: 28KiB bootloader
Clock Reference: 8 MHz crystal
Communication interface: Serial on USART1 PA10/PA9
```

Current status:

- The deprecated MCU warning was cleared.
- The Pi and Creality 4.2.2 board are communicating normally.

Notes:

- Do not reflash the MCU just because there is a minor Klipper update.
- Reflash only when Klipper requires it, a protocol mismatch appears, or a required MCU-level feature is missing.

---

## 2026-05 - Moonraker Update Manager Cleanup

Status: Completed.

After the fresh SD card rebuild, Moonraker showed stale update-manager warnings for extensions that were referenced in configuration but missing on disk.

Problem extensions included:

```text
mobileraker
crowsnest
octoeverywhere
```

Example causes:

```text
/home/pi/mobileraker_companion does not exist
/home/pi/crowsnest/tools/pkglist.sh does not exist
/home/pi/octoeverywhere does not exist
```

Resolution:

- Stale `[update_manager ...]` blocks were cleaned/commented/removed from `moonraker.conf`.
- Moonraker was restarted.
- A later log check showed:

```text
startup_warnings: []
```

Current status:

- Moonraker warning state was cleaned after resurrection.
- Extension entries should only exist if the matching extension is actually installed.

---

## 2026-05 - Samsung Phone Interface / KlipperScreen Attempt

Status: KlipperScreen removed; phone interface idea remains optional.

A Samsung phone was considered as a printer-mounted interface to replace the look/function of the stock Ender screen.

Paths considered:

- Mainsail/Fluidd mobile web UI as a full-screen web app
- Mobileraker app
- KlipperScreen streamed over the network through XServer/XSDL

What happened:

- KlipperScreen was installed during testing.
- XServer/XSDL on the Samsung phone was unstable and crashed.
- KlipperScreen was removed to keep the Raspberry Pi 3 B+ lightweight.

Current preferred approach if a phone is mounted later:

- Use Mainsail as a full-screen PWA / home-screen web app, or
- Use Mobileraker directly against Moonraker.

Do not reinstall KlipperScreen unless there is a real dedicated display plan.

---

## 2026-05 - Direct Drive / Pancake Stepper Wiring

Status: Completed and documented.

During the direct-drive conversion, the pancake extruder stepper did not work correctly with the first assumed wire order.

Important final wiring result:

```text
Phase A: Black + Green
Phase B: Red + Blue
Final connector order: Black, Green, Red, Blue
```

The bad wiring behavior was a thump/noise without smooth movement, indicating a phase mismatch.

Detailed notes are in:

```text
docs/configuration.md
```

---

## 2026-05 - Temporary Extrusion Baseline Before Final Hotend

Status: Temporary baseline only.

Before installing the final CHC hotend / bi-metal heatbreak / upgraded nozzle, extrusion was tested on the current hotend setup.

Observed behavior:

```text
10 mm/s extrusion command: severe skipping
2 mm/s extrusion command: smooth, continuous, click-free extrusion
```

How to treat this:

- This proves the wiring and basic extrusion path can work.
- This is not a final max-flow result.
- Retest after the CHC hotend, heatbreak, nozzle, cooling, and bed mounts are installed.

---

## 2026-05 - AI-Assisted Voltage Verification Lesson

Status: Ongoing rule.

During earlier AI-assisted work, bad advice was given around fan voltage: a 12V fan was discussed in the context of being powered from Raspberry Pi 5V power.

Project rule going forward:

```text
Verify voltage, current, connector pinout, and control method before buying or wiring any fan, heater, buck converter, sensor, or controller.
```

For anything tied to the Raspberry Pi GPIO or 5V rail:

- Use 5V-rated devices only, or
- Use a proper driver/MOSFET/relay/buck setup with common ground where appropriate.

Do not assume a fan is correct based on size alone.

---

## Current Historical Baseline

After the resurrection phase, the important baseline was:

- Fresh Raspberry Pi SD card installed after old card failure
- Klipper host rebuilt
- Creality v4.2.2 MCU firmware updated to match modern Klipper host expectations
- Moonraker warnings cleaned
- G-Code Shell Command restored
- Klipper-Backup restored
- `update_git` button working
- Direct drive hardware installed
- Pancake stepper wiring corrected
- BLTouch working after wiring correction
- Top-mounted filament runout sensor installed/planned around DET/PA4 behavior

This is the foundation for the next phase: finishing the hotend, cooling, and bed-mount hardware upgrades before final tuning.
