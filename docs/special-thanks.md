# Special Thanks

This project would not exist without the work of many talented developers and the open source community.

---

## sw3defy — K2 Plus Helper Script and Wiki

[sw3defy](https://github.com/sw3defy) created the [K2 Plus Helper Script](https://github.com/sw3defy/Creality-Helper-Script-K2-Plus) and [K2 Plus Wiki](https://github.com/sw3defy/Creality-Helper-Script-Wiki-K2-Plus), rebuilt from the ground up for the K2 architecture. This K2 (base model) script and wiki are forked from that work.

- [K2 Plus Helper Script](https://github.com/sw3defy/Creality-Helper-Script-K2-Plus)
- [K2 Plus Wiki](https://sw3defy.github.io/Creality-Helper-Script-Wiki-K2-Plus/)

---

## Guilouz — Original Creality Helper Script Wiki

[Guilouz](https://github.com/Guilouz) created the original [Creality Helper Script](https://github.com/Guilouz/Creality-Helper-Script) and [Creality Helper Script Wiki](https://github.com/Guilouz/Creality-Helper-Script-Wiki) for K1 Series printers, which the K2 Plus project was based on.

- [Guilouz GitHub](https://github.com/Guilouz)
- [Original Wiki](https://guilouz.github.io/Creality-Helper-Script-Wiki/)

---

## DnG-Crafts — K2 Camera Fix

[DnG-Crafts](https://github.com/DnG-Crafts/K2-Camera) discovered and documented the K2 camera WebRTC signaling protocol and created the original HTML-based camera workaround for Fluidd. Their work made it possible to reverse-engineer the custom base64 JSON WebRTC format used by the K2 camera, leading to the full go2rtc integration.

- [K2-Camera GitHub](https://github.com/DnG-Crafts/K2-Camera)

---

## AlexxIT — go2rtc

[go2rtc](https://github.com/AlexxIT/go2rtc) by AlexxIT is the stream conversion software that bridges the K2's proprietary WebRTC camera to Fluidd, Mainsail and Mobileraker, including its official `#format=creality` handler and the optional MJPEG transcoding.

- [go2rtc GitHub](https://github.com/AlexxIT/go2rtc)

---

## prestonbrown — HelixScreen

[prestonbrown](https://github.com/prestonbrown) created [HelixScreen](https://helixscreen.org), a modern Klipper touchscreen UI with explicit K2 series support including prebuilt armhf binaries and CFS protocol documentation.

- [HelixScreen Website](https://helixscreen.org)
- [HelixScreen GitHub](https://github.com/prestonbrown/helixscreen)
- [K2 Series Documentation](https://helixscreen.org/dev/printers/creality-k2/)

---

## vsevolod-volkov — Entware on the K2 family

[vsevolod-volkov](https://github.com/vsevolod-volkov) discovered the solution to install Entware on the K2 Plus (and with it the K2 family) by creating a Python-based dummy wget script to bootstrap the Entware installer, bypassing the ARM ABI incompatibility.

- [K2Plus-entware GitHub](https://github.com/vsevolod-volkov/K2Plus-entware)

---

## Klipper

[Klipper](https://www.klipper3d.org) is the open source 3D printer firmware that powers the K2. Without Klipper, none of this would be possible.

- [Klipper Website](https://www.klipper3d.org)
- [Klipper GitHub](https://github.com/Klipper3d/klipper)

---

## Moonraker

[Moonraker](https://moonraker.readthedocs.io) is the API server that bridges Klipper with web interfaces like Fluidd and Mainsail.

- [Moonraker Documentation](https://moonraker.readthedocs.io)
- [Moonraker GitHub](https://github.com/Arksine/moonraker)

---

## Fluidd

[Fluidd](https://docs.fluidd.xyz) is the web interface pre-installed on the K2 on port 4408.

- [Fluidd Website](https://docs.fluidd.xyz)
- [Fluidd GitHub](https://github.com/fluidd-core/fluidd)

---

## Mainsail

[Mainsail](https://docs.mainsail.xyz) is an alternative web interface for Klipper, available on port 4409 after installation.

- [Mainsail Website](https://docs.mainsail.xyz)
- [Mainsail GitHub](https://github.com/mainsail-crew/mainsail)

---

## KAMP — Klipper Adaptive Meshing & Purging

[KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) by kyleisah enables adaptive bed meshing and purging for Klipper printers.

- [KAMP GitHub](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)

---

## grant0013 — KAMP-K2

[grant0013](https://github.com/grant0013) created [KAMP-K2](https://github.com/grant0013/KAMP-K2) (GPL-3.0), the K2 port of KAMP. It restores Klipper's own `BED_MESH_CALIBRATE` over Creality's `prtouch_v3` handler (`restore_bed_mesh.py`), defers the `master-server` mesh and runs the adaptive mesh in `START_PRINT`. The helper script's KAMP option installs KAMP-K2 v1.1.0 and ports its config patching to run on the printer.

- [KAMP-K2 GitHub](https://github.com/grant0013/KAMP-K2)

---

## Clon1998 — Mobileraker

[Clon1998](https://github.com/Clon1998) develops the [Mobileraker](https://mobileraker.com) app and [Mobileraker Companion](https://github.com/Clon1998/mobileraker_companion), which the helper script installs for push notifications.

- [Mobileraker Companion GitHub](https://github.com/Clon1998/mobileraker_companion)
- [Mobileraker App GitHub](https://github.com/Clon1998/mobileraker)

---

## Arksine — Moonraker and gcode_shell_command

[Arksine](https://github.com/Arksine) is the author of Moonraker and of `gcode_shell_command.py`, the Klipper extra installed by the helper script's *Klipper Gcode Shell Command* option.

- [Arksine GitHub](https://github.com/Arksine)

---

## QuinnDamerell and crysxd — OctoEverywhere and OctoApp

The [OctoEverywhere](https://github.com/QuinnDamerell/OctoPrint-OctoEverywhere) and [OctoApp](https://github.com/crysxd/OctoApp-Plugin) installers support the K2 directly; the helper script runs them after installing their prerequisites with Entware.

- [OctoEverywhere GitHub](https://github.com/QuinnDamerell/OctoPrint-OctoEverywhere)
- [OctoApp Plugin GitHub](https://github.com/crysxd/OctoApp-Plugin)

---

## Material for MkDocs

This wiki is built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) by squidfunk.

- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)

---

## Support the Original Authors

This fork builds on sw3defy's K2 Plus work. If you find it useful, consider supporting sw3defy:

<a href="https://buymeacoffee.com/sw3defy"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" height="40"></a>

<a href="https://ko-fi.com/sw3defy"><img src="https://ko-fi.com/img/githubbutton_sm.svg" height="40"></a>
