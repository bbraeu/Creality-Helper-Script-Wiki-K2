# OctoEverywhere — K2

OctoEverywhere provides free remote access to your printer from anywhere in the world — including live webcam, full Fluidd/Mainsail control, AI print failure detection and notifications.

!!! note "Entware required"
    Install [Entware](entware-k2.md) (option 13) first.

---

## Installation

From the Helper Script menu, select **option 14 — OctoEverywhere**.

The helper script:

1. Installs `git`, `git-http` and `bash` with Entware.
2. Clones the [OctoEverywhere plugin](https://github.com/QuinnDamerell/OctoPrint-OctoEverywhere) to `/mnt/UDISK/octoeverywhere`.
3. Runs the upstream `install.sh`. The upstream installer detects the K2 itself and installs its Python dependencies and ffmpeg via Entware.

Selecting option 14 again runs the upstream installer again (update / repair).

---

## Linking Your Account

At the end, the installer prints a link. **Open that link** in your browser and sign in (or create a free account at [octoeverywhere.com](https://octoeverywhere.com)) to link the printer to your account. Afterwards, access the printer at [octoeverywhere.com](https://octoeverywhere.com) or via the OctoEverywhere app.

---

## Features

- **Remote access** — full Fluidd or Mainsail interface from any device
- **Live webcam** — view your print from anywhere
- **AI failure detection** — Gadget AI monitors your print and pauses on spaghetti detection
- **Notifications** — print start, finish, pause, and failure alerts
- **Secure** — encrypted, no port forwarding required

!!! tip "Webcam remotely in Mobileraker"
    Mobileraker can use OctoEverywhere for remote access. Remotely, only the **MJPEG** webcam works, not WebRTC — answer `y` to *Add MJPEG camera?* when installing [Camera Support](../configurations/configure-camera.md).

---

## Removing

Remove menu (option 20) → **14) OctoEverywhere**. The helper runs the upstream `uninstall.sh` and deletes `/mnt/UDISK/octoeverywhere`.
