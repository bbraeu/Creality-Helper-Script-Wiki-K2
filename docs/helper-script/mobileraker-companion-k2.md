# Mobileraker Companion — K2

[Mobileraker Companion](https://github.com/Clon1998/mobileraker_companion) sends push notifications (print progress, finish, errors) to the Mobileraker app, even when the app is closed. The upstream installer only knows the K1 and Sonic Pad, so the helper script installs it by hand: Entware Python, a virtualenv and a procd service.

!!! note "Entware required"
    Install [Entware](entware-k2.md) (option 13) first.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 15 — Mobileraker Companion**.

The script:

| Step | Details |
|---|---|
| Packages | `git git-http python3 python3-pip python3-pillow` via Entware (Pillow comes from Entware because there is no armv7 wheel), then `virtualenv` via pip |
| Code | Clones the companion to `/mnt/UDISK/mobileraker_companion` (or updates it) |
| Python environment | Virtualenv at `/mnt/UDISK/mobileraker-env` with the requirements installed (takes a few minutes) |
| Config | Creates `/mnt/UDISK/printer_data/config/mobileraker.conf` if it does not exist. The timezone comes from the printer (`uci`) or is asked for. |
| Service | procd service `/etc/init.d/mobileraker` (starts on boot, restarted if it exits) |
| Log | `/mnt/UDISK/printer_data/logs/mobileraker.log` |
| Updates | Adds `[update_manager mobileraker]` to the UDISK `moonraker.conf`, so Fluidd / Mainsail can update the companion |

Selecting option 15 again offers to update / repair the installation. An existing `mobileraker.conf` is kept.

---

## Connect the App

Open the Mobileraker app and add the printer **by IP address**, not by hostname. The camera URLs registered by Camera Support contain the IP, and Mobileraker maps them through OctoEverywhere for remote viewing only when the host matches the printer address it knows.

---

## Camera in Mobileraker

Install [Camera Support](../configurations/configure-camera.md) (option 11) and answer **`y`** to *Add MJPEG camera?*:

| Webcam | Service | Who can use it |
|---|---|---|
| **K2 Camera** | `webrtc-go2rtc` | Mobileraker **supporters** (paid) only; does not work remotely via OctoEverywhere |
| **K2 Camera MJPEG** | `mjpegstreamer` | **Everyone**; works remotely via OctoEverywhere |

- Notification **snapshots** can only come from an MJPEG webcam. Choosing the snapshot webcam in the app is a supporter feature; otherwise the companion uses `snapshot_uri` from `mobileraker.conf`, which the helper sets to the go2rtc snapshot (`http://127.0.0.1:1984/api/frame.jpeg?src=k2plus`).
- Mobileraker's own `webrtc-creality` camera mode does not work on the K2 ([mobileraker#452](https://github.com/Clon1998/mobileraker/issues/452), open).
- Webcams via Obico are not supported by Mobileraker.

---

## Configuration

The generated `/mnt/UDISK/printer_data/config/mobileraker.conf`:

```ini
[general]
timezone: Europe/Berlin
include_snapshot: True

[printer K2]
moonraker_uri: ws://127.0.0.1:7125/websocket
moonraker_api_key: False
# Fallback when no snapshot webcam is chosen in the app (needs Camera Support)
snapshot_uri: http://127.0.0.1:1984/api/frame.jpeg?src=k2plus
```

See the [Mobileraker Companion documentation](https://github.com/Clon1998/mobileraker_companion) for all options. Restart the service after editing:

```bash
/etc/init.d/mobileraker restart
```

---

## Known Issue — HeaterFan Parse Error (reported on K2 Plus)

!!! warning "Reported on the K2 Plus — check the issue"
    On the K2 Plus, the Mobileraker **app** failed to connect with:

    ```
    Found _$HeaterFanImpl, parentException: null
    PrinterBuilder._updateTemperatureFan
    ```

    The K2 Plus `printer.cfg` uses a fan configuration the app's printer builder did not handle. This is tracked in [Mobileraker issue #429](https://github.com/Clon1998/mobileraker/issues/429) — check the issue for its current status. It has not been verified whether the K2 base model is affected. The companion itself (server side) is not involved.

---

## Removing

Remove menu (option 19) → **15) Mobileraker Companion**. This stops and removes the service, removes `[update_manager mobileraker]`, deletes the code, the virtualenv and `mobileraker.conf`, and restarts Moonraker.
