# OctoApp Companion — K2

The [OctoApp Companion](https://github.com/crysxd/OctoApp-Plugin) adds push notifications and live activities for the OctoApp mobile app.

!!! note "Entware required"
    Install [Entware](entware-k2.md) (option 13) first.

---

## Installation

From the Helper Script menu, select **option 16 — OctoApp Companion**.

The helper script:

1. Installs `git`, `git-http` and `bash` with Entware.
2. Clones the [OctoApp plugin](https://github.com/crysxd/OctoApp-Plugin) to `/mnt/UDISK/octoapp`.
3. Runs the upstream `install.sh`. The upstream installer detects the K2 itself and installs its Python dependencies and ffmpeg via Entware.

Selecting option 16 again runs the upstream installer again (update / repair).

---

## Linking the App

At the end, the installer prints a link — follow it to connect the companion with your OctoApp account / app.

---

## Removing

Remove menu (option 20) → **16) OctoApp Companion**. The helper runs the upstream `uninstall.sh` and deletes `/mnt/UDISK/octoapp`.
