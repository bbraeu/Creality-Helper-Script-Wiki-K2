# Creality K2 - Helper Script Wiki

The wiki for the Creality K2 Helper Script, for the **Creality K2** (base model, 260 × 260 mm) and **K2 Combo (with CFS)**.
The helper script also works on the K2 Pro and K2 Plus: it reads the hardware from your stock `printer.cfg`
instead of assuming one model.

Forked from [sw3defy's K2 Plus Helper Script Wiki](https://github.com/sw3defy/Creality-Helper-Script-Wiki-K2-Plus), which is based on the excellent [Guilouz Creality Helper Script Wiki](https://github.com/Guilouz/Creality-Helper-Script-Wiki).

## Wiki

**[https://bbraeu.github.io/Creality-Helper-Script-Wiki-K2-Base/](https://bbraeu.github.io/Creality-Helper-Script-Wiki-K2-Base/)**

## Helper Script

**[https://github.com/bbraeu/Creality-Helper-Script-K2-Base](https://github.com/bbraeu/Creality-Helper-Script-K2-Base)**

## About

The K2 has a fundamentally different architecture from the K1 Series:
- Different filesystem paths (`/mnt/UDISK/` instead of `/usr/data/`)
- Different service manager (OpenWrt rc.d / procd instead of supervisorctl)
- Pre-installed Fluidd on port 4408
- CoreXY, 260 × 260 × 260 mm build volume
- **No chamber heater** (chamber temperature sensor and chamber fan only)
- CFS (Color Filament System) multi-material support on the K2 Combo
- LIS2DW accelerometer (not ADXL345)

This wiki is adapted from the K2 Plus wiki. Content shared across the K2 family (paths, services, root access,
helper script behaviour) carries over; K2-base-specific verification on real hardware is ongoing, and pages with
unverified hardware details say so.

## Features

- Hardware detection (`Detected: K2 (260x260 bed, no chamber heater)`) and macros generated for the detected hardware
- Moonraker Extensions & Update Manager
- Fluidd & Mainsail web interfaces
- Fans Control, Useful Macros, Z-Offset, M600 support
- KAMP-K2 (Klipper Adaptive Meshing & Purging, K2 port)
- Improved Shapers Calibrations
- Moonraker Timelapse
- Camera Support for Fluidd, Mainsail and Mobileraker (WebRTC + optional MJPEG via go2rtc)
- HelixScreen (modern touchscreen UI)
- Entware, OctoEverywhere, Mobileraker Companion, OctoApp Companion, Git Backup, Klipper Gcode Shell Command
- Backup & Restore of Klipper configuration and Moonraker database
- Tools: restarts, logs, Entware update, clear logs & cache, system information

## Discussions

[Join the discussion](https://github.com/bbraeu/Creality-Helper-Script-Wiki-K2-Base/discussions)

## Support

If this project saved you some time, you can buy me a coffee:

<a href="https://buymeacoffee.com/braeubernhk"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" height="40"></a>

## Credits

- [sw3defy](https://github.com/sw3defy/Creality-Helper-Script-Wiki-K2-Plus) - K2 Plus Helper Script and Wiki this fork is based on
- [Guilouz](https://github.com/Guilouz) - Original Creality Helper Script and Wiki for K1 Series
- [grant0013/KAMP-K2](https://github.com/grant0013/KAMP-K2) - KAMP port for the K2 (GPL-3.0)
- [DnG-Crafts](https://github.com/DnG-Crafts/K2-Camera) - K2 camera WebRTC discovery
- [vsevolod-volkov](https://github.com/vsevolod-volkov/K2Plus-entware) - Entware on K2 (wget shim solution)
- [AlexxIT/go2rtc](https://github.com/AlexxIT/go2rtc) - Stream conversion software
- [Arksine](https://github.com/Arksine) - Moonraker, gcode_shell_command.py
- [Clon1998/mobileraker_companion](https://github.com/Clon1998/mobileraker_companion) - Mobileraker Companion
- [prestonbrown/HelixScreen](https://github.com/prestonbrown/helixscreen) - Modern touchscreen UI for Klipper
- [Klipper](https://www.klipper3d.org), [Moonraker](https://moonraker.readthedocs.io), [Fluidd](https://docs.fluidd.xyz), [Mainsail](https://docs.mainsail.xyz)
- [KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) - Adaptive meshing
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) - Wiki theme
