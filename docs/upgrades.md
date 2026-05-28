# Upgrades

This document tracks Leonardo E3's upgrade path.

It is split into completed upgrades, near-term planned upgrades, and ordered/on-hand parts that are not installed yet. Longer-term ideas belong in [`future-ideas.md`](future-ideas.md).

---

## Completed Upgrades / Installed Parts

### Base Printer

Status: Installed / original platform.

Source link:
https://www.creality3dofficial.com/products/official-creality-ender-3-3d-printer?srsltid=AfmBOooijRqKdK1gHTpAJgoAMWA8nOw_ZDymNjQwWoCMLoLqmY01tuOh

Notes:

- Original Creality Ender 3 platform.
- Creality v4.2.2 mainboard, non-silent version.
- Quiet operation is an important future goal.
- Current tuning and configuration should assume the Creality 4.2.2 board unless/until the board is upgraded.

---

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
- Required corrected wiring to work properly with the Creality 4.2.2 E-stepper port.
- Final verified connector order is documented in [`configuration.md`](configuration.md).
- Motor temperature and torque should be evaluated after the final hotend is installed and extrusion testing begins.

---

### PTFE Tube Between Extruder and Hotend

Status: Installed and working.

Source link:
https://www.amazon.com/dp/B0F1TVH4TW?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1

Notes:

- Used to connect the direct-drive extruder output to the hotend input.
- Current setup is working.
- Re-check tube length/fitment after final hotend, heatbreak, and cooling hardware are installed.

---

### Ductinator Cooling Duct

Status: Installed / in use for the current toolhead plan.

Source link:
https://www.thingiverse.com/thing:6939003

Notes:

- Current BLTouch offset assumptions are based around the Ductinator-style duct placement.
- Final part-cooling performance should be tested after the two 24V duct fans are installed.

---

### Magnetic PEI Build Plate

Status: Installed.

Source link:
https://www.amazon.com/dp/B07XBM24HN?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_45&th=1

Notes:

- Installed as the current removable print surface.
- Used as the baseline build plate for future slicer profile work.
- Future OrcaSlicer profile should assume a PEI build surface.
- Bed mesh should be regenerated after silicone bed mounts and final hotend/toolhead work are complete.

---

### Top-Mounted Filament Runout Sensor

Status: Installed physically; final firmware behavior still needs validation.

Sensor source link:
https://www.amazon.com/dp/B07V8TVK6N?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_43

Mount resource link:
https://www.printables.com/model/368594-mount-for-the-filament-sensor-for-ender-3-v2

Notes:

- Mounted at the top of the printer.
- May reduce safe usable Z height depending on filament path, toolhead clearance, and cable routing.
- Final safe Z height must be matched in both `printer.cfg` and the dedicated OrcaSlicer profile.
- Firmware behavior and pin assignment notes are tracked in [`configuration.md`](configuration.md).

---

### BLTouch

Status: Installed and working.

Source link:
https://www.amazon.com/dp/B0CGLM4QK8?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_44

Notes:

- Used for Z homing and bed probing.
- Current working probe offsets are based on the current toolhead/duct layout.
- Wiring/config notes are tracked in [`configuration.md`](configuration.md).

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

---

### Buck Converter / Raspberry Pi Printer-Powered Setup

Status: Installed and working.

Buck converter source link:
https://www.amazon.com/dp/B08NV3JCBC?ref=ppx_yo2ov_dt_b_fed_asin_title

Notes:

- Buck converter powers the Raspberry Pi from the printer's 24V PSU.
- Power is fed into the Pi through the normal micro USB power input so the Pi's built-in fuse/protection path is retained.
- Power architecture and workflow notes are tracked in [`configuration.md`](configuration.md).

---

### Miscellaneous Hardware Assortments

Status: On hand / used as needed.

Source links:

- https://www.amazon.com/dp/B0D14BC8QS?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1
- https://www.amazon.com/dp/B0D1KQCBMT?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_4&th=1
- https://www.amazon.com/dp/B0CTMP1BFM?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_5&th=1
- https://www.amazon.com/dp/B0CTMBW8H6?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_6&th=1
- https://www.amazon.com/dp/B07VZTSHB4?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_15

Notes:

- Miscellaneous fasteners/hardware are used as needed during the build.
- Track specific usage later only if a part becomes important for fitment, serviceability, or repeatability.

---

## Near-Term Planned Upgrades

These are part of the current active upgrade path. They are not long-term future ideas.

### Trianglelab 115W CHC Pro Hotend Kit

Status: Pending install.

Source link:
https://www.aliexpress.us/item/3256804038017822.html?spm=a2g0o.order_list.order_list_main.11.b0e318020SxXM1&gatewayAdapt=glo2usa

Notes:

- Install before final tuning.
- Requires careful heater/thermistor verification and PID tuning.

---

### Mellow All-Metal Bi-Metal CR10/Crazy Heatbreak

Status: Pending install.

Source link:
https://www.aliexpress.us/item/3256802721411891.html?spm=a2g0o.order_list.order_list_main.23.b0e318020SxXM1&gatewayAdapt=glo2usa

Notes:

- Install with the hotend upgrade.
- Re-check hotend assembly and extrusion path afterward.

---

### Mellow Gold DLC Hardened Steel / Copper Bimetal GHC V6 Nozzle

Status: Pending install.

Source link:
https://www.aliexpress.us/item/3256809178438391.html?spm=a2g0o.order_list.order_list_main.29.b0e318020SxXM1

Notes:

- Install as part of the hotend/nozzle upgrade phase.
- Confirm nozzle fitment and probe clearance before printing.

---

### Silicone Bed Mounts / Spring Replacements

Status: Pending install.

Source link:
https://www.aliexpress.com/item/3256808932604349.html?spm=a2g0o.order_list.order_list_main.17.3c1818020NUC69

Notes:

- Install before final mesh and first-layer tuning.
- Re-run bed mesh and related calibration afterward.

---

## Ordered / On Hand but Not Installed Yet

### Dual 24V Part-Cooling Fans for Ductinator

Status: Ordered / not installed yet.

Source link:
https://www.aliexpress.us/item/3256808842298737.html?spm=a2g0o.order_list.order_list_main.35.2f7d18026auBdG&gatewayAdapt=glo2usa

Notes:

- Two fans will be used on the Ductinator for part cooling.
- User ordered the 24V variant.
- Final fan config and cooling tests should wait until the fans arrive and are installed.

---

### Stepper Motor Heatsinks

Status: Ordered / not installed yet.

Source link:
https://www.aliexpress.us/item/2255800537553298.html?spm=a2g0o.order_list.order_list_main.47.2f7d18026auBdG&gatewayAdapt=glo2usa

Notes:

- Intended for stepper motor cooling/thermal management.
- Not installed yet.
- Revisit after installation if motor temperatures remain a concern.
