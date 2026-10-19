# Creality K2 — Helper Script Wiki

This wiki covers the complete process to root the **Creality K2 series** — K2, K2 Combo (with CFS), K2 Pro and K2 Plus — and add features using the Creality K2 Helper Script.

The advantage of rooting your printer is having full access to the firmware and configuration files, enabling you to install community tools, improve print quality, and extend your printer far beyond its factory defaults.

!!! danger "Read before proceeding"
    If you don't know what you're doing, I don't recommend following this guide.
    Modifying system files can cause issues if done incorrectly. Always back up your configuration before making changes.

!!! note "This wiki is for the whole K2 series"
    The helper script runs on the **K2 / K2 Combo**, **K2 Pro** and **K2 Plus**: it detects the hardware (bed size, chamber heater, chamber fan / sensor, fans, Z-tilt) from your `printer.cfg` instead of assuming one model.
    Hardware examples on these pages (build volume, chamber, pins) were verified on the K2 base model (260 × 260 × 260 mm) unless a page says otherwise.
    For K1 Series, K1C, K1 Max, or Ender-3 V3 Series — see the [original Guilouz wiki](https://guilouz.github.io/Creality-Helper-Script-Wiki/).

!!! warning "Verification status"
    This wiki is adapted from the K2 Plus wiki. Everything that is shared across the K2 family (paths under `/mnt/UDISK/`, services, root access, Fluidd on port 4408) carries over unchanged. K2-base-specific verification on real hardware is ongoing — pages that still contain unverified details say so at the top.

---

## About This Wiki

This project is a fork of [sw3defy's K2 Plus Helper Script Wiki](https://github.com/sw3defy/Creality-Helper-Script-Wiki-K2-Plus), which itself is based on the excellent [Guilouz Creality Helper Script Wiki](https://github.com/Guilouz/Creality-Helper-Script-Wiki). The K2 family has a fundamentally different architecture from the K1 Series — different filesystem paths, service manager, pre-installed software, and the CFS multi-material system. See [Special Thanks](special-thanks.md) for full attribution.

---

## Key Differences from K1 Series

Understanding these differences is important before following any guide. K1-targeted instructions will not work on the K2 without modification.

| Feature | K1 Series | K2 |
|---|---|---|
| Persistent data path | `/usr/data/` | `/mnt/UDISK/` |
| Root password | `creality_2023` | **`creality_2024`** |
| Service manager | Supervisor Lite (`supervisorctl`) | OpenWrt rc.d / procd (`/etc/init.d/`) |
| Restart Klipper | `supervisorctl restart klipper` | `/etc/init.d/S55klipper restart` |
| Restart Moonraker | `supervisorctl restart moonraker` | `/etc/init.d/S56moonraker restart` |
| Fluidd pre-installed | No | **Yes** (port 4408) |
| Moonraker pre-installed | No | **Yes** (read-only at `/usr/share/`) |
| Chamber heater | No | **No** (chamber temperature sensor only) |
| Multi-material (CFS) | No | **Yes** (Combo model, `box.cfg`) |
| Accelerometer | ADXL345 | **LIS2DW** (SPI on nozzle_mcu) |
| Kinematics | CoreXY | CoreXY |
| Print volume | 220×220×250mm | **260×260×260mm** |
| Probe type | Inductive | **prtouch_v3 (strain gauge)** |
| OS | Tina 4.x / OpenWrt | **Tina 5.0 / OpenWrt 21.02** |

## K2 Family Overview

| | K2 | K2 Pro | K2 Plus |
|---|---|---|---|
| Build volume | 260 × 260 × 260 mm | 300 × 300 × 300 mm | 350 × 350 × 350 mm |
| Chamber heater | **no** (sensor only) | yes (60 °C) | yes |

K2 base specifications: max speed 600 mm/s, max acceleration 20,000 mm/s², nozzle up to 300 °C, bed up to 100 °C, no RFID reader. The **K2 Combo** is the K2 bundled with the CFS — everything on this wiki that mentions the CFS applies to the Combo.

---

## Quick Start

1. [Enable root access and connect via SSH](firmwares/install-and-update-rooted-firmware-k2.md#enable-root-access)
2. [Install the Helper Script](helper-script/helper-script-installation.md)
3. Install features in the [recommended order](helper-script/helper-script-installation.md#recommended-installation-order)
4. [Configure OrcaSlicer](slicers/orcaslicer.md) with the correct K2 start/end G-code

---

## Wiki

Full wiki available at: [bbraeu.github.io/Creality-Helper-Script-Wiki-K2](https://bbraeu.github.io/Creality-Helper-Script-Wiki-K2/)

Helper Script repository: [github.com/bbraeu/Creality-Helper-Script-K2](https://github.com/bbraeu/Creality-Helper-Script-K2)

---

## Support This Project

If this K2 helper script and wiki saved you some time, you can buy me a coffee:

<a href="https://buymeacoffee.com/braeubernhk"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" height="40"></a>
