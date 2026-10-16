# Moonraker Extensions — K2

!!! warning "K2 differs significantly from K1"
    On the K1 Series, Moonraker and Nginx are installed by the helper script. On the K2, **both are pre-installed from the factory firmware** and run from the read-only `/usr/share/` partition. The approach here is therefore fundamentally different: instead of installing from scratch, you are **extending** the stock installation.

---

## What Ships From the Factory

The K2 comes with the following already running:

| Component | Details |
|---|---|
| Fluidd | Served at `http://<printer-ip>:4408`, static files at `/usr/share/fluidd/` |
| Moonraker | Running at `127.0.0.1:7125`, config at `/usr/share/moonraker/moonraker.conf` |
| Nginx | Config at `/etc/nginx/nginx.conf`, single-file (no `conf.d/` directory) |
| Klipper | Running via `/usr/share/klippy-env/`, config at `/mnt/UDISK/printer_data/config/printer.cfg` |

You can verify all four are running over SSH:

```bash
ps aux | grep -E 'klipper|moonraker|nginx' | grep -v grep
```

---

## Architecture Overview

```
Browser → nginx (port 4408)
              ├── / → /usr/share/fluidd/          (static Fluidd files)
              ├── /websocket → 127.0.0.1:7125      (Moonraker WebSocket)
              ├── /printer|api|access|machine|server → 127.0.0.1:7125  (Moonraker REST)
              └── /webcam/ → 127.0.0.1:8080        (WebRTC/MJPEG streams)

Moonraker (7125) → /tmp/klippy_uds → Klipper (klippy.py)
```

Mainsail is not pre-installed. See [Mainsail](mainsail-k2.md) to add it on port `4409`.

---

## Moonraker Config Location

The stock Moonraker config is at `/usr/share/moonraker/moonraker.conf` on the **read-only system partition**.

!!! danger "Do not edit /usr/share/moonraker/moonraker.conf directly"
    This file is on the read-only system partition and will be **overwritten on every firmware update**. Any changes made directly to this file will be lost. Use the helper script approach to apply persistent changes.

The stock config includes:

- `[machine]` with `provider: none` — meaning Moonraker has no system service management capability. Reboot/shutdown buttons in Fluidd may not work (see below).
- No `[update_manager]` section — the Update Manager panel in Fluidd is empty by default.
- No `[timelapse]` component — see [Moonraker Timelapse](moonraker-timelapse-k2.md) to add it.

---

## Extending Moonraker with Include Files

Because the stock config is read-only, the helper script adds a persistent include directive by patching the moonraker startup. Additional config is written to:

```
/mnt/UDISK/printer_data/config/moonraker.conf
```

This file survives firmware updates because it lives on `/mnt/UDISK/`.

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the script's `[Install] Menu` select **option 1 — Moonraker Extensions & Update Manager**.

The script will:

1. Write `/mnt/UDISK/printer_data/config/moonraker.conf`, starting with `[include /usr/share/moonraker/moonraker.conf]` so the stock settings are still loaded.
2. Patch the Moonraker startup script (`/etc/rc.d/S56moonraker`, original saved as `S56moonraker.orig`) to start Moonraker with this file.
3. Restart Moonraker.

Every start of the helper script (and every boot) additionally makes sure the stock `/usr/share/moonraker/moonraker.conf` includes the UDISK file and sets `queue_gcode_uploads: True` — otherwise `[timelapse]` and `[update_manager]` sections in the UDISK file were silently ignored, and the web UIs flooded Moonraker with "Metadata not available" requests.

Other features add their sections to this file: Moonraker Timelapse (`[timelapse]`), KAMP-K2 (`[file_manager] enable_object_processing: True`) and Mobileraker Companion (`[update_manager mobileraker]`).

To remove it, use the Remove menu (option 19 → 1): the stock startup is restored and `moonraker.conf` is archived as `moonraker.conf.removed.<date>`.

---

## Restarting Services

The K2 uses OpenWrt rc.d init scripts. There is **no `supervisorctl`** command (unlike K1).

```bash
# Restart Moonraker
/etc/init.d/S56moonraker restart

# Restart Klipper
/etc/init.d/S55klipper restart

# Restart Nginx
/etc/init.d/S80nginx restart

# Restart the low-level MCU bridge
/etc/init.d/S54klipper_mcu restart
```

The helper script's Tools menu has shortcuts: options 24 (Klipper), 25 (Moonraker) and 26 (Nginx).

---

## Reboot / Shutdown Buttons

The stock `[machine]` section uses `provider: none`, so the Reboot and Shutdown buttons in Fluidd/Mainsail may not work. The K2 helper script does not install a host-control shim (the K1 helper script's *Host Control Support* is not part of it). Use the touchscreen or SSH (`reboot`).

---

## Adding Mainsail

Mainsail is not pre-installed. To add it on port `4409`, see [Mainsail](mainsail-k2.md). The helper script adds a second `server` block to the nginx config for Mainsail alongside the existing Fluidd block.

---

## Update Manager

The **Software Updates** panel in Fluidd / Mainsail lists every component that has an `[update_manager ...]` section in a loaded Moonraker config. The helper script adds one for **Mobileraker Companion**; HelixScreen's own update entry is removed because it is not supported on the K2. The helper script itself is updated by re-running the installation commands — see [Install Helper Script → Update](helper-script-installation.md#update).

Updates performed via the Update Manager do **not** affect the firmware itself or the read-only `/usr/share/` partitions.
