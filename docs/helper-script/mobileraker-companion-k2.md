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
| Updates | Select option 15 again, or — with the [Updatable Moonraker](moonraker-k2.md#updatable-moonraker-upstream) — Fluidd → Settings → Software Updates |

Selecting option 15 again offers to update / repair the installation. An existing `mobileraker.conf` is kept.

---

## Connect the App

Open the Mobileraker app and add the printer **by IP address**, not by hostname. The camera URLs registered by Camera Support contain the IP, and Mobileraker maps them through OctoEverywhere for remote viewing only when the host matches the printer address it knows.

---

## Camera in Mobileraker — Step by Step

Mobileraker reads the webcams from Moonraker, so there is nothing to type in the app — the work is on the printer side.

**1. Install Entware** (option 13), if not done yet. It provides ffmpeg 5, which the live MJPEG camera needs.

**2. Install Camera Support** (option 11) and answer:

| Question | Answer |
|---|---|
| *Continue?* | `y` |
| *Add MJPEG camera?* | **`y`** — this is the camera the free Mobileraker version can show |
| *Install ffmpeg 5 from Entware?* | **`y`** — live MJPEG (about 10 fps). With `n` the MJPEG camera runs in snapshot mode (about 1 fps). |
| *Reboot now?* | `y` |

The camera needs 60–90 seconds after the reboot.

**3. Test it on your phone.** Open this in the phone's browser (same Wi-Fi as the printer) — you should see live video:

```
http://<printer-ip>:4408/go2rtc/api/stream.mjpeg?src=k2plus_mjpeg
```

If this stays black or fails, see [Camera Support → Troubleshooting](../configurations/configure-camera.md#troubleshooting) before going on.

**4. Add the printer in the Mobileraker app as `<printer-ip>:4408`** (e.g. `192.168.1.202:4408`) — the IP address, not the hostname, and port **4408** (Fluidd's web server, which also forwards to Moonraker).

- With `:4408`, the app's **default webcam values work as they are** — if you add a webcam yourself in the app, keep *MJPEG* with stream `/webcam/?action=stream` and snapshot `/webcam/?action=snapshot`; the helper maps these classic mjpg-streamer URLs to the K2 camera.
- Plain `192.168.1.202` (Moonraker's port 7125) works too for the webcams the helper registered, but relative URLs like the app defaults then point to port 80, which is Creality's own web server.
- The IP matters for remote viewing: Mobileraker only maps webcam URLs through OctoEverywhere when their host matches the printer address it knows.

**5. Pick the webcam.** The dashboard's webcam card lists the webcams registered in Moonraker:

| Webcam | Service | Who can use it |
|---|---|---|
| **K2 Camera MJPEG** | `mjpegstreamer` (`mjpegstreamer-adaptive` in snapshot mode) | **Everyone**; also works remotely via OctoEverywhere |
| **K2 Camera** | `webrtc-go2rtc` | Mobileraker **supporters** (paid) only; best quality, least printer CPU; local network only |

Select **K2 Camera MJPEG** (or **K2 Camera** if you are a supporter).

**6. Pictures in notifications** — install Mobileraker Companion (option 15, this page). Notification snapshots can only come from an MJPEG webcam:

- **Without supporter subscription:** nothing to do — the companion uses `snapshot_uri` from `mobileraker.conf`, which the helper already points at the go2rtc snapshot.
- **Supporters:** in the app's printer settings, choose **K2 Camera MJPEG** as the snapshot webcam.

**7. Remote viewing (optional)** — install [OctoEverywhere](octoeverywhere.md) (option 14) and link it in Mobileraker. Remotely, **only the MJPEG camera works**; WebRTC needs the local network.

!!! tip "CPU usage"
    Live MJPEG is transcoded by ffmpeg **only while someone watches it** (about one third of the printer's CPU). Fluidd shows the WebRTC camera instead, so an open Fluidd tab costs nothing extra.

Notes:

- Mobileraker's own `webrtc-creality` camera mode does not work on the K2 ([mobileraker#452](https://github.com/Clon1998/mobileraker/issues/452), open) — use the webcams above.
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

Remove menu (option 20) → **15) Mobileraker Companion**. This stops and removes the service, removes a leftover `[update_manager mobileraker]` from older helper versions, deletes the code, the virtualenv and `mobileraker.conf`, and restarts Moonraker.
