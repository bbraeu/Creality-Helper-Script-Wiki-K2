# Helper Script for Creality K2

The Creality K2 Helper Script is a menu-driven shell script for the **Creality K2 series**: K2 / K2 Combo (with CFS), K2 Pro and K2 Plus. It installs features by writing Klipper `.cfg` files to `/mnt/UDISK/printer_data/config/`, extending the stock Moonraker, and adding a few services — see [Install Helper Script](helper-script-installation.md) for the installation commands.

It is forked from [sw3defy's K2 Plus Helper Script](https://github.com/sw3defy/Creality-Helper-Script-K2-Plus), which is based on [Guilouz's Creality Helper Script](https://github.com/Guilouz/Creality-Helper-Script).

Repository: [github.com/bbraeu/Creality-Helper-Script-K2](https://github.com/bbraeu/Creality-Helper-Script-K2)

---

## Hardware Detection

The K2 family (K2 / K2 Pro / K2 Plus) shares firmware and stock macros but differs in bed size, chamber heater and fans. Instead of assuming one model, the script reads the hardware from your stock config every time it starts. Only files Klipper actually loads are read (`printer.cfg` and its `[include]` tree) — SAVE_CONFIG backups such as `printer-<date>.cfg` in the same folder are ignored:

| Detected | From |
|---|---|
| Model (K2 / K2 Pro / K2 Plus) | `position_max` of `[stepper_x]` (262 mm on the K2) |
| Usable bed area | `mesh_min` / `mesh_max` of `[bed_mesh]` — the travel limits reach past the bed (X −12…262, Y up to 296.5 on the K2) |
| Chamber heater | a `[heater_generic ...chamber...]` section |
| Chamber fan | a `[temperature_fan ...chamber...]` section |
| Chamber sensor | the chamber heater, else a `[temperature_sensor ...chamber...]` (`chamber_temp` on the K2), else the chamber fan |
| Part cooling enable / aux fan | `[output_pin fan0_en]` (a `static_digital_output` on the K2, so it is left alone), `[output_pin fan2]` |
| Fan PWM scale | `scale` of `[output_pin fan0]` / `[output_pin fan2]` (255 on the K2) |
| Z tilt | a `[z_tilt]` section |

The menu header shows the result, for example:

```
   Detected: K2 (no chamber heater, bed mesh 20,20 to 255,255)
```

Generated macros only contain what your hardware has. On a K2 base model this means: no `CHAMBER_HEAT` or `PID_CHAMBER` (there is no chamber heater), `CHAMBER_STATUS` reads the existing chamber sensor, `CHAMBER_COOL` only if a chamber fan exists, `Z_TILT_CALIBRATE` only with a `[z_tilt]` section, and `WARMUP` stays inside the bed mesh area. The same script still works on the K2 Pro and K2 Plus.

---

## Menu Overview

| Option | Feature | Page |
|---|---|---|
| 1 | Moonraker (extensions / updatable Moonraker) | [Moonraker](moonraker-k2.md) |
| 2 | Fans Control Macros | [Fans Control Macros](fans-control-macros-k2.md) |
| 3 | Useful Macros (PID, leveling, warmup) | [Useful Macros](useful-macros-k2.md) |
| 4 | Save Z-Offset Macros | [Save Z-Offset Macros](save-z-offset-macros-k2.md) |
| 5 | M600 Filament Change Support | [M600 Support](m600-support-k2.md) |
| 6 | Klipper Adaptive Meshing & Purging (KAMP-K2) | [KAMP-K2](klipper-adaptive-meshing-and-purging-k2.md) |
| 7 | Improved Shapers Calibrations | [Improved Shapers](improved-shapers-calibrations-k2.md) |
| 8 | Fluidd (install/update/repair — port 4408) | [Fluidd](fluidd-k2.md) |
| 9 | Mainsail (install/update/repair — port 4409) | [Mainsail](mainsail-k2.md) |
| 10 | Moonraker Timelapse | [Moonraker Timelapse](moonraker-timelapse-k2.md) |
| 11 | Camera Support (Fluidd, Mainsail, Mobileraker) | [Camera Support](../configurations/configure-camera.md) |
| 12 | HelixScreen (touchscreen UI) | [HelixScreen](helixscreen-k2.md) |
| 13 | Entware Package Manager (needed by 14–17) | [Entware](entware-k2.md) |
| 14 | OctoEverywhere | [OctoEverywhere](octoeverywhere.md) |
| 15 | Mobileraker Companion | [Mobileraker Companion](mobileraker-companion-k2.md) |
| 16 | OctoApp Companion | [OctoApp Companion](octoapp-companion.md) |
| 17 | Git Backup | [Git Backup](git-backup.md) |
| 18 | Klipper Gcode Shell Command | [Klipper Gcode Shell Command](gcode-shell-command.md) |
| 19 | Auto Power Off (smart plug after the print) | [Auto Power Off](https://github.com/bbraeu/Creality-Helper-Script-K2#auto-power-off) |
| 20 | Remove a feature (1–19, same order) | [Install Menu](install-menu/index.md#remove-menu) |
| 21–24 | Backup / restore Klipper configuration and Moonraker database | [Backup & Restore Menu](backup-menu/index.md) |
| 25–34 | Restarts, logs, installed features, Entware update, clear logs & cache, system information, update helper script | [Tools Menu](tools-menu/index.md) |

---

## Requirements

- Creality K2 / K2 Combo (also K2 Pro, K2 Plus)
- Root SSH access enabled — see [Enable Root Access](../firmwares/install-and-update-rooted-firmware-k2.md#enable-root-access)
- An SSH client (e.g. MobaXterm or PuTTY on Windows)
- [Entware](entware-k2.md) (option 13) for OctoEverywhere, Mobileraker Companion, OctoApp Companion and Git Backup
