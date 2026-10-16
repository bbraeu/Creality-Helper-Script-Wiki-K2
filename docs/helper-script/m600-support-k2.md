# M600 Support — K2

M600 is the standard filament change G-code command. On the K2, M600 behavior differs depending on whether you are using single-filament mode or the CFS.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 5 — M600 Filament Change Support**.

---

## Single-Filament Mode

In single-filament mode, `M600` triggers a filament change pause:

1. The filament is retracted by 5 mm
2. The toolhead lifts 10 mm (limited to the Z maximum) and parks at X10 Y10
3. The nozzle target is lowered by 20°C (not below 160°C) to limit ooze while keeping it warm
4. The print pauses and the console shows *"M600: Filament change paused. Swap filament then RESUME."*
5. You swap the filament manually (the macros `FILAMENT_UNLOAD` / `FILAMENT_LOAD TEMP=...` help), then resume with `RESUME`

Park position and retract length are variables of the macro (`variable_park_x`, `variable_park_y`, `variable_retract` in `m600.cfg`).

---

## CFS Mode (K2 Combo)

With the CFS, colour changes in multi-material prints are tool changes (`T<n>`) handled by the stock CFS macros — the M600 macro is meant for single-filament manual changes. See [CFS — Color Filament System](cfs-k2.md).

---

## Config Location

The M600 macro is written to `/mnt/UDISK/printer_data/config/m600.cfg` and included from `printer.cfg`.

!!! note "Path difference from K1"
    On K1, config lives under `/usr/data/printer_data/config/`. On K2 it is `/mnt/UDISK/printer_data/config/`.
