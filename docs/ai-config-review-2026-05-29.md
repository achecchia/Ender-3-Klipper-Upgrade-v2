# AI Config Review - 2026-05-29

This page records the configuration changes that were uploaded after reviewing Project Leonardo with a Klipper/3D-printer-specific AI assistant.

This is documentation only. No printer configuration files should be edited from this page.

---

## Scope Reviewed

The uploaded backup advanced the repo beyond the previous documentation state and changed several files under:

```text
printer_data/config/
```

Compared with the prior docs-protection baseline, the current backup shows the following broad changes:

```text
Added:
- printer_data/config/README_PROJECT_LEONARDO.md
- printer_data/config/mainsail_overrides.cfg

Removed:
- printer_data/config/KlipperScreen.conf
- printer_data/config/ai_README_FIRST.txt
- printer_data/config/ai_adxlmcu.cfg
- printer_data/config/ai_macros.cfg
- printer_data/config/ai_mainsail.cfg
- printer_data/config/ai_printer.cfg
- printer_data/config/ai_shell_command.cfg
- printer_data/config/crowsnest.conf.2026-05-23-1732

Modified:
- printer_data/config/adxlmcu.cfg
- printer_data/config/crowsnest.conf
- printer_data/config/macros.cfg
- printer_data/config/mobileraker.conf
- printer_data/config/moonraker.conf
- printer_data/config/octoeverywhere-system.cfg
- printer_data/config/octoeverywhere.conf
- printer_data/config/printer.cfg
- printer_data/config/shell_command.cfg
- printer_data/config/sonar.conf
```

---

## New Printer-Side Notes File

A new local printer note file was added:

```text
printer_data/config/README_PROJECT_LEONARDO.md
```

Purpose:

```text
A printer-local sanity map for the Project Leonardo Klipper setup.
```

It documents:

- active config files
- config ownership rules
- include checklist
- OrcaSlicer start/end G-code
- bed mesh policy
- CHC/hotend upgrade reminders
- first power-on checklist
- first Klipper motion/probe checks
- tuning log placeholders
- backup/save-point notes
- duplicate-section troubleshooting notes

Important: this file is not Klipper config. It is a documentation/reference file stored beside the configs.

---

## Config Ownership Rules Captured by the AI Pass

The new printer-side notes define clearer ownership boundaries:

```text
printer.cfg owns hardware and core machine configuration.
macros.cfg owns print/tuning/helper macros.
mainsail.cfg owns Mainsail pause/resume/cancel/display plumbing.
shell_command.cfg owns shell command helpers.
adxlmcu.cfg is temporary input-shaper equipment config.
```

The key rule is:

```text
Do not define the same Klipper section in more than one included config file.
```

Common duplicate-risk sections include:

```ini
[pause_resume]
[display_status]
[respond]
[exclude_object]
[gcode_macro PAUSE]
[gcode_macro RESUME]
[gcode_macro CANCEL_PRINT]
```

---

## printer.cfg Changes to Document

`printer.cfg` was modified during the AI-assisted cleanup.

Known important change:

```ini
[exclude_object]
```

was added to `printer.cfg`.

The stated config ownership rule is that `[exclude_object]` belongs in `printer.cfg`, not in `macros.cfg`.

This matters because object exclusion and any adaptive mesh/purge behavior depend on Moonraker/Klipper object data, but duplicate `[exclude_object]` sections can break Klipper startup.

Existing known machine facts still apply:

```text
- Creality 4.2.2 non-silent board
- Physical Vref stepper current adjustment
- No TMC/UART sections
- BLTouch installed and working
- Direct-drive BMG-style extruder
- Headless printer; stock screen removed from normal/final setup
```

---

## macros.cfg Changes to Document

`macros.cfg` had the largest change set.

Important documented/observed macro changes include:

```text
PRINT_END retract changed from a large Bowden-style retract to a small direct-drive-friendly E-3 retract.
```

Reason:

```text
Leonardo E3 is now a direct-drive machine, so large end-of-print retracts are not appropriate.
```

Known helper/tuning macro themes from the AI pass:

- print start/end cleanup
- pause/resume/cancel helper cleanup
- parking helpers
- bed mesh helpers
- PID helper macros
- probe and safe movement test helpers
- duplicate-section cleanup

Review note:

The AI-generated printer-local notes say the desired policy is that normal prints should load a saved `default` bed mesh rather than creating a fresh mesh every print. Before relying on that behavior, confirm the live `PRINT_START` macro in `macros.cfg` matches that policy.

Desired normal-print mesh policy:

```text
Normal print:
1. Heat bed
2. Home
3. Load saved mesh profile default if it exists
4. Continue without mesh if no saved profile exists yet
5. Heat nozzle
6. Prime
7. Print
```

Manual mesh creation policy:

```gcode
CREATE_BED_MESH TEMP=60 SOAK=300 SAVE=1
```

Use that after major hardware changes or when refreshing the mesh deliberately.

---

## ADXL / Input-Shaper Config

`adxlmcu.cfg` was cleaned up and documented as temporary test equipment.

Current intended behavior:

```text
Normal printing:
- Keep adxlmcu.cfg available on the Pi if desired.
- Keep the include commented out in printer.cfg.

Input-shaper testing:
- Plug in the accelerometer MCU.
- Uncomment the include.
- Restart Klipper.
- Run resonance tests.
- Save resulting input_shaper values.
- Comment the include back out afterward.
```

Reason:

```text
If the ADXL MCU is unplugged while adxlmcu.cfg is included, Klipper may fail to connect.
```

Known ADXL config intent:

```ini
[mcu adxl]
[adxl345]
[resonance_tester]
```

Probe point documented for an Ender 3-size bed:

```text
125,125,20
```

---

## Shell Command / Mainsail Button Status

`shell_command.cfg` is still the home for Mainsail shell helpers.

Known important macros/buttons:

```text
update_git
PULL_FROM_GIT
```

Workflow remains:

```text
If GitHub/docs changed:
1. Press PULL_FROM_GIT
2. Press update_git

If only printer config changed locally:
1. Press update_git
```

Do not merge pull and push behavior into the same macro unless there is a very good reason.

---

## Removed AI Draft Files

The AI draft files were removed from the backup:

```text
ai_README_FIRST.txt
ai_adxlmcu.cfg
ai_macros.cfg
ai_mainsail.cfg
ai_printer.cfg
ai_shell_command.cfg
```

This suggests the AI-generated proposed files were promoted, consolidated, or replaced by live config files and the new printer-local `README_PROJECT_LEONARDO.md`.

Going forward:

```text
Do not assume ai_ files exist.
Use the live config files and README_PROJECT_LEONARDO.md as the current reference.
```

---

## KlipperScreen Config Removed

`printer_data/config/KlipperScreen.conf` was removed.

This matches the current physical setup:

```text
Leonardo E3 is currently a headless printer.
The stock screen is removed for the normal/final build.
Primary control is Mainsail/Moonraker.
```

The stock screen may be reinstalled temporarily if it is ever genuinely useful for troubleshooting/calibration, but it is not part of the intended final configuration.

---

## Web / Remote Services

The AI pass also modified service-related config files:

```text
crowsnest.conf
mobileraker.conf
moonraker.conf
octoeverywhere-system.cfg
octoeverywhere.conf
sonar.conf
```

Documentation-level meaning:

```text
The remote/monitoring stack is still actively configured and should be treated as part of the project, not as random leftover files.
```

Current intended roles remain:

```text
Mainsail       = local Klipper web UI
Moonraker      = Klipper API/backend
UniFi VPN      = secure full remote admin access
OctoEverywhere = cloud access, notifications, webcam/failure monitoring
GitHub         = config history and documentation
```

Crowsnest/webcam behavior should be rechecked when the camera mount and final camera are installed.

---

## Items to Verify Before Calling This Final

The AI pass appears useful, but the following should be verified on the actual printer before treating the configuration as final:

```text
1. Klipper restarts cleanly.
2. No duplicate section errors.
3. Mainsail macros show only the expected buttons.
4. PULL_FROM_GIT still works.
5. update_git still works.
6. PRINT_START behavior matches the intended bed-mesh policy.
7. PRINT_END direct-drive retract behavior is correct.
8. BLTouch still homes/probes correctly.
9. ADXL include remains commented out for normal printing.
10. Headless operation remains normal; no screen dependency is introduced.
```

---

## Documentation Impact

This AI-assisted config pass changes the documentation baseline.

Relevant docs now need to treat the following as current project facts:

```text
- Live configs were consolidated/cleaned by a Klipper-focused AI assistant.
- README_PROJECT_LEONARDO.md is now the printer-local config map.
- ai_ draft files are no longer part of the active repo.
- KlipperScreen config was removed; Leonardo E3 remains headless.
- [exclude_object] belongs in printer.cfg.
- ADXL config is temporary test equipment config.
- Small direct-drive PRINT_END retract is the intended behavior.
- PULL_FROM_GIT remains the required first step after GitHub-side doc/config changes.
```

---

## Reminder

After this documentation page is added from GitHub, press:

```text
PULL_FROM_GIT
```

in Mainsail so the Pi pulls this page into `~/config_backup` before the next `update_git` backup.
