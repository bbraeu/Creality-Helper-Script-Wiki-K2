# Moonraker Timelapse — K2

Moonraker Timelapse is a third-party Moonraker component that creates timelapse recordings of prints by capturing a frame at each layer change.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 10 — Moonraker Timelapse**.

The script downloads [moonraker-timelapse](https://github.com/mainsail-crew/moonraker-timelapse), copies the component to `/usr/share/moonraker/components/timelapse.py` and the macros to `/mnt/UDISK/printer_data/config/timelapse.cfg`, adds `[timelapse]` to `/mnt/UDISK/printer_data/config/moonraker.conf` and includes `timelapse.cfg` from `printer.cfg`. Moonraker Extensions are installed first if missing.

!!! note "Frame capture needs wget"
    The `[timelapse]` component grabs each frame with `wget`. The stock firmware has none; the helper links `/usr/bin/wget` to Entware's `/opt/bin/wget` when Entware is installed. Install [Entware](entware-k2.md) if no video is produced.

---

## Timelapse Video Location

Completed timelapse videos are saved to:

```
/mnt/UDISK/printer_data/timelapse/
```

!!! note "Path difference from K1"
    On K1, timelapse videos are saved to `/usr/data/printer_data/timelapse/`. On K2 the path is `/mnt/UDISK/printer_data/timelapse/`.

Videos can be downloaded directly from Fluidd's file manager under the **Timelapse** section, or via SCP:

```bash
scp root@<printer-ip>:/mnt/UDISK/printer_data/timelapse/my_print.mp4 ./
```

---

## Slicer Setup

Add the following to your slicer's layer change G-code:

```gcode
TIMELAPSE_TAKE_FRAME
```

And add this to your `END_PRINT` macro or slicer end G-code:

```gcode
TIMELAPSE_RENDER
```

---

## Camera

Moonraker Timelapse uses the snapshot URL from the camera configuration to capture frames. With Camera Support installed, the snapshot URL is (the stream name `k2plus` is used on every K2 model):

```
http://<printer-ip>:4408/go2rtc/api/frame.jpeg?src=k2plus
```

Ensure [Camera Support](../configurations/configure-camera.md) is installed before enabling timelapse.
