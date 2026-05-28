# Completed Upgrades

This document tracks the parts, upgrades, services, and baseline calibration milestones that are already installed or configured on the Ender 3 Klipper build.

Items listed here are considered part of the current machine baseline unless otherwise noted.

---

## Base Printer

Status: Installed / original platform.

Source link:
https://www.creality3dofficial.com/products/official-creality-ender-3-3d-printer?srsltid=AfmBOooijRqKdK1gHTpAJgoAMWA8nOw_ZDymNjQwWoCMLoLqmY01tuOh

Notes:

- Base machine for this project.
- Original Creality Ender 3 platform.
- This printer has a Creality v4.2.2 mainboard.
- The installed v4.2.2 board is the **non-silent** version.
- Quiet operation is an important future goal for this build.
- Current Klipper config and future tuning should assume the Creality 4.2.2 board unless/until the board is upgraded.
- Stepper current is handled by physical Vref potentiometers, not software UART current control.
- A future silent-board upgrade may become worthwhile for noise reduction, even if the current board remains functional.

---

## Hardware Upgrades / Installed Parts

### BMG-Style Direct Drive Extruder

Status: Installed.

Source link:
https://www.aliexpress.us/item/3256805805447850.html?spm=a2g0o.order_list.order_list_main.59.b0e318020SxXM1&gatewayAdapt=glo2usa

Notes:

- Replaced the stock Bowden-style extrusion setup with a BMG-style dual-gear direct drive setup.
- Current live config uses a BMG-style `rotation_distance` baseline of `7.71`.
- Final extrusion calibration should be revisited after the hotend/cooling upgrades are complete.

---

### Pancake Extruder Stepper Motor

Status: Installed and wired correctly.

Source link:
https://www.amazon.com/dp/B0FHHXHHRW?ref=ppx_yo2ov_dt_b_fed_asin_title

Notes:

- Installed as part of the direct-drive conversion.
- The pancake stepper required repinning/rewiring to work correctly with the Creality 4.2.2 E-stepper port.
- Final verified connector order, left to right, is:

```text
Black, Green, Red, Blue
```

- The incorrect order `Black, Red, Blue, Green` caused a thump/noise without smooth movement, indicating a phase mismatch.
- Motor current is controlled by the physical Vref potentiometer on the Creality 4.2.2 board.
- No Klipper UART/TMC driver current control is currently active.
- Do not add `[tmc2209 extruder]` or software `run_current` settings unless the mainboard is upgraded to one with UART-controlled drivers.
- Motor temperature and torque should be evaluated after the final hotend is installed and extrusion testing begins.

More detail:

- See [`hardware-notes-audited.md`](hardware-notes-audited.md)

---

### Ductinator Cooling Duct

Status: Installed / in use for the current toolhead plan.

Source link:
https://www.thingiverse.com/thing:6939003

Notes:

- Current BLTouch offset assumptions are based around the Ductinator-style duct placement.
- Current probe offsets in the live config are:

```ini
x_offset: 0
y_offset: -28
```

- These should be left alone unless the physical probe location changes.

---

### Magnetic PEI Build Plate

Status: Installed.

Source link:
https://www.amazon.com/dp/B07XBM24HN?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_45&th=1

Notes:

- Installed as the current removable print surface.
- Used as the baseline build plate for future slicer profile work.
- Future OrcaSlicer profile should assume a PEI build surface.
- First-layer tuning and bed temperature assumptions should be based on this surface.
- Bed mesh should be regenerated after silicone bed mounts and final hotend/toolhead work are complete.

---

### Top-Mounted Filament Runout Sensor

Status: Installed physically; final firmware behavior still needs validation.

Sensor source link:
https://www.amazon.com/dp/B07V8TVK6N?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_43

Mount resource link:
https://www.printables.com/model/368594-mount-for-the-filament-sensor-for-ender-3-v2

Notes:

- Filament sensor is mounted at the top of the printer.
- Detects filament presence/runout.
- This may reduce safe usable Z height depending on filament path, toolhead clearance, and cable routing.
- User is okay with reducing Z height if needed.
- Final safe Z height must be matched in both `printer.cfg` and the dedicated OrcaSlicer profile.
- Earlier notes identify `PA4` as a likely Creality 4.2.2 DET/runout pin, but this still needs validation.
- Final sensor behavior should be tested before relying on it during long prints.

More detail:

- See [`hardware-notes-audited.md`](hardware-notes-audited.md)

---

### BLTouch

Status: Installed and working.

Source link:
https://www.amazon.com/dp/B0CGLM4QK8?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_44

Notes:

- BLTouch is configured as the Z virtual endstop.
- Mounted on the toolhead.
- Used for Z homing and bed probing.
- A previous Z-axis homing crash was resolved by correcting the BLTouch signal/ground wiring orientation.
- Known working wiring note: white is signal, black is ground.
- Current working BLTouch-related configuration includes:

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

More detail:

- See [`hardware-notes-audited.md`](hardware-notes-audited.md)

---

### Belt Tensioner

Status: Installed.

Source link:
https://www.amazon.com/dp/B096M2HNP9?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_46

Notes:

- Installed to improve belt tension adjustment.
- Exact axis/location should be confirmed and documented later if needed.
- Belt tension affects motion quality, ringing, and input shaping results.
- Re-check belt tension before final speed/acceleration tuning.
- Revalidate input shaping later if belt tension or toolhead mass changes significantly.

---

## Power / Electronics Setup

### Raspberry Pi Printer-Powered Setup

Status: Installed.

Power path:

```text
Printer 24V PSU
    ↓
Buck converter stepped down to approximately 5.1V
    ↓
USB cable
    ↓
Raspberry Pi micro USB power input
```

Notes:

- The Pi is powered from the printer's 24V PSU through a buck converter.
- Power is fed into the Pi through the normal micro USB power input so the Pi's built-in fuse/protection path is retained.
- Future mainboard power-switching plans should keep the Pi powered independently of the Creality mainboard.

---

### Creality 4.2.2 Mainboard / Klipper USB Connection

Status: Installed and working.

Notes:

- The Raspberry Pi communicates with the Creality 4.2.2 board over USB.
- The installed 4.2.2 board is the non-silent version.
- Stepper currents are adjusted physically on the board through Vref potentiometers.
- No active UART-controlled TMC configuration is currently used in Klipper.
- A quieter electronics phase may eventually include a silent board upgrade.

---

## Firmware / Software / Remote Access

### Klipper + Mainsail

Status: Installed and working.

Notes:

- Klipper is running on the Raspberry Pi.
- Mainsail is the local web interface.
- Moonraker is the backend/API layer.

---

### Klipper-Backup

Status: Installed and working.

Notes:

- Klipper configuration is backed up to this GitHub repository.
- Manual backup is available through the Mainsail `update_git` button.
- Manual console backup command:

```gcode
update_git
```

- Manual console backup with message:

```gcode
update_git MESSAGE="backup message here"
```

---

### G-Code Shell Command Extension

Status: Installed and working.

Notes:

- Required for the Mainsail `update_git` macro/button to trigger Klipper-Backup from the UI.
- `shell_command.cfg` is now included by the active `printer.cfg`.

---

### OctoEverywhere for Klipper

Status: Installed.

Notes:

- Installed for cloud access, print notifications, webcam/failure monitoring, and spaghetti detection support.
- UniFi VPN remains the preferred path for higher-risk admin tasks like SSH, config edits, and Klipper/Moonraker updates.

---

### GitHub Documentation / Project Tracking

Status: Active.

Repository link:
https://github.com/achecchia/Ender-3-Klipper-Upgrade-v2

Notes:

- README is the project overview.
- `docs/punch-list.md` tracks the staged upgrade process.
- `docs/future-upgrade-ideas.md` tracks optional future ideas.
- `docs/hardware-notes-audited.md` tracks hardware-specific wiring and troubleshooting notes.
- `docs/config-and-workflow-notes.md` tracks workflow, config, backup, and remote-access notes.
- This file tracks completed/installed upgrades.

---

## Config / Tuning Milestones Already Completed

### Bed Mesh

Status: Completed baseline mesh exists.

Notes:

- A saved bed mesh exists in the live `printer.cfg` `SAVE_CONFIG` block.
- Mesh should be regenerated after silicone bed mounts and hotend/toolhead hardware changes are complete.

---

### Input Shaping

Status: Completed baseline input shaping data exists.

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

## Related Links for Parts Not Yet Completed

These links were provided during the project discussion but are not listed as completed upgrades yet.

They should remain tracked in the punch list and/or future upgrade documents until installed and validated.

### Trianglelab 115W CHC Pro Hotend Kit

Status: Pending install.

https://www.aliexpress.us/item/3256804038017822.html?spm=a2g0o.order_list.order_list_main.11.b0e318020SxXM1&gatewayAdapt=glo2usa

### Mellow All-Metal Bi-Metal CR10/Crazy Heatbreak

Status: Pending install.

https://www.aliexpress.us/item/3256802721411891.html?spm=a2g0o.order_list.order_list_main.23.b0e318020SxXM1&gatewayAdapt=glo2usa

### Mellow Gold DLC Hardened Steel / Copper Bimetal GHC V6 Nozzle

Status: Pending install.

https://www.aliexpress.us/item/3256809178438391.html?spm=a2g0o.order_list.order_list_main.29.b0e318020SxXM1

### Dual Blower Fans for Duct Setup

Status: Pending install / planned with duct setup.

https://www.aliexpress.us/item/3256808842298737.html?spm=a2g0o.order_list.order_list_main.35.3c1818020NUC69&gatewayAdapt=glo2usa

### Silicone Bed Mounts / Spring Replacements

Status: Pending install.

https://www.aliexpress.com/item/3256808932604349.html?spm=a2g0o.order_list.order_list_main.17.3c1818020NUC69

---

## Documentation Notes

When more completed parts or upgrades are added, each entry should include:

- part or upgrade name
- source link when available
- installed status
- printer location / function
- fitment notes
- config/tuning impact
- related documentation links
