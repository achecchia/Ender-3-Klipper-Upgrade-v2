# Installed Parts Inventory

This document tracks parts that are already installed on the Ender 3 Klipper build.

The goal is to keep a clear record of what is physically on the machine, where it came from, and what future tuning/config decisions it may affect.

---

## Base Printer

Status: Installed / original platform.

Source link:
https://www.creality3dofficial.com/products/official-creality-ender-3-3d-printer?srsltid=AfmBOooijRqKdK1gHTpAJgoAMWA8nOw_ZDymNjQwWoCMLoLqmY01tuOh

Printer location / function:

- Base machine for this project.
- Original Creality Ender 3 platform.

Important machine-specific note:

- This printer has a Creality v4.2.2 mainboard.
- The installed v4.2.2 board is the **non-silent** version.
- Quiet operation is an important future goal for this build.

Config / tuning notes:

- Current Klipper config and future tuning should assume the Creality 4.2.2 board unless/until the board is upgraded.
- Stepper current is handled by physical Vref potentiometers, not software UART current control.
- A future silent-board upgrade may become worthwhile for noise reduction, even if the current board remains functional.

---

## Filament Runout Sensor

Status: Installed.

Source link:
https://www.amazon.com/dp/B07V8TVK6N?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_43

Printer location / function:

- Mounted at the top of the printer.
- Detects filament presence/runout.
- May affect usable Z height because of sensor position and filament path.

Related documentation:

- [`docs/hardware-notes-audited.md`](hardware-notes-audited.md)
- [`docs/completed-upgrades.md`](completed-upgrades.md)

Config / tuning notes:

- Earlier notes identify `PA4` as a likely Creality 4.2.2 DET/runout pin, but this still needs validation.
- Final sensor behavior should be tested before relying on it during long prints.
- Final safe Z height must be reflected in both `printer.cfg` and the dedicated OrcaSlicer profile.

---

## BLTouch

Status: Installed and working.

Source link:
https://www.amazon.com/dp/B0CGLM4QK8?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_44

Printer location / function:

- Mounted on the toolhead.
- Used for Z homing and bed probing.

Related documentation:

- [`docs/hardware-notes-audited.md`](hardware-notes-audited.md)
- [`docs/completed-upgrades.md`](completed-upgrades.md)

Config / tuning notes:

Current working probe offset assumptions:

```ini
x_offset: 0
y_offset: -28
```

Important historical note:

- A previous Z-homing crash was resolved by correcting BLTouch wiring orientation.
- Known working wiring note: white is signal, black is ground.
- Do not change BLTouch behavior unless there is a confirmed problem.

---

## Magnetic PEI Build Plate

Status: Installed.

Source link:
https://www.amazon.com/dp/B07XBM24HN?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_45&th=1

Printer location / function:

- Installed as the current removable print surface.
- Used as the baseline build plate for future slicer profile work.

Related documentation:

- [`docs/completed-upgrades.md`](completed-upgrades.md)

Config / tuning notes:

- Future OrcaSlicer profile should assume a PEI build surface.
- First-layer tuning and bed temperature assumptions should be based on this surface.
- Bed mesh should be regenerated after silicone bed mounts and final hotend/toolhead work are complete.

---

## Belt Tensioner

Status: Installed.

Source link:
https://www.amazon.com/dp/B096M2HNP9?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_46

Printer location / function:

- Installed to improve belt tension adjustment.
- Exact axis/location should be confirmed and documented later if needed.

Config / tuning notes:

- Belt tension affects motion quality, ringing, and input shaping results.
- Re-check belt tension before final speed/acceleration tuning.
- Revalidate input shaping later if belt tension or toolhead mass changes significantly.

---

## Documentation Notes

When more installed-part links are added, each entry should include:

- part name
- source link
- installed status
- printer location / function
- fitment notes
- config/tuning impact
- related documentation links
