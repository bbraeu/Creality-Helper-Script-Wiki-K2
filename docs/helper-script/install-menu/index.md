# Install Menu

The `[Install] Menu` (options 1–18) is grouped in the order the features should be installed. Select a number, confirm, and the script does the rest. Selecting an option again for an installed feature offers to reinstall, update or repair it (or, for Git Backup and Entware, opens a small management menu).

Make sure you have followed the [Install Helper Script](../helper-script-installation.md) section first.

---

## Step 1: Foundation

| Option | Feature | Notes |
|---|---|---|
| 1 | [Moonraker](../moonraker-k2.md) — 1) Extensions for Creality's Moonraker, 2) [Updatable Moonraker](../moonraker-k2.md#updatable-moonraker-upstream) | Install first. Persistent `moonraker.conf` in `/mnt/UDISK`. |

## Step 2: Print Macros

| Option | Feature | Notes |
|---|---|---|
| 2 | [Fans Control Macros](../fans-control-macros-k2.md) | Generated for the detected fans / chamber hardware |
| 3 | [Useful Macros (PID, leveling, warmup)](../useful-macros-k2.md) | `PID_HOTEND`, `PID_BED`, `BED_LEVELING`, `WARMUP`, chamber macros |
| 4 | [Save Z-Offset Macros](../save-z-offset-macros-k2.md) | |
| 5 | [M600 Filament Change Support](../m600-support-k2.md) | Single-filament manual changes |

## Step 3: Leveling & Calibration

| Option | Feature | Notes |
|---|---|---|
| 6 | [Klipper Adaptive Meshing & Purging (KAMP-K2)](../klipper-adaptive-meshing-and-purging-k2.md) | Needs a slicer start G-code change |
| 7 | [Improved Shapers Calibrations](../improved-shapers-calibrations-k2.md) | LIS2DW accelerometer |

## Step 4: Web Interface & Camera

| Option | Feature | Notes |
|---|---|---|
| 8 | [Fluidd](../fluidd-k2.md) | Pre-installed; update / repair on port 4408 |
| 9 | [Mainsail](../mainsail-k2.md) | Port 4409 |
| 10 | [Moonraker Timelapse](../moonraker-timelapse-k2.md) | Needs Camera Support for frames |
| 11 | [Camera Support (Fluidd, Mainsail, Mobileraker)](../../configurations/configure-camera.md) | go2rtc bridge, optional MJPEG camera |
| 12 | [HelixScreen](../helixscreen-k2.md) | Replaces the stock touchscreen UI |

## Step 5: Remote Access, Notifications & Backup

| Option | Feature | Notes |
|---|---|---|
| 13 | [Entware Package Manager](../entware-k2.md) | Needed by options 14–17 |
| 14 | [OctoEverywhere](../octoeverywhere.md) | Requires Entware |
| 15 | [Mobileraker Companion](../mobileraker-companion-k2.md) | Requires Entware |
| 16 | [OctoApp Companion](../octoapp-companion.md) | Requires Entware |
| 17 | [Git Backup](../git-backup.md) | Requires Entware; install option 18 first for macros |
| 18 | [Klipper Gcode Shell Command](../gcode-shell-command.md) | `RUN_SHELL_COMMAND` for your own macros |
| 19 | [Auto Power Off](https://github.com/bbraeu/Creality-Helper-Script-K2-Base#auto-power-off) | Switches the printer off with a smart plug once the print has cooled down |

Not in the menu, but documented here:

- [SimplyPrint](../simplyprint-k2.md) — installed with SimplyPrint's own K2 guide
- [CFS — Color Filament System](../cfs-k2.md) — stock K2 Combo hardware and macros

---

## Remove Menu

Option **20 — Remove a feature** opens the `[Remove] Menu`. It lists the same features as options 1–19, in the same order:

| Option | Removes |
|---|---|
| 1 | Moonraker Extensions (restores the stock Moonraker startup, archives `moonraker.conf`) |
| 2 | Fans Control Macros |
| 3 | Useful Macros |
| 4 | Save Z-Offset Macros |
| 5 | M600 Support |
| 6 | KAMP-K2 (restores `gcode_macro.cfg` from before the install) |
| 7 | Improved Shapers Calibrations |
| 8 | Restore stock Fluidd (nginx block) |
| 9 | Mainsail |
| 10 | Moonraker Timelapse |
| 11 | Camera Support |
| 12 | HelixScreen |
| 13 | Entware Package Manager (removes `/opt` and all packages) |
| 14 | OctoEverywhere (runs the upstream `uninstall.sh`) |
| 15 | Mobileraker Companion |
| 16 | OctoApp Companion (runs the upstream `uninstall.sh`) |
| 17 | Git Backup (your GitHub repository is not touched) |
| 18 | Klipper Gcode Shell Command (refused while a config still uses `[gcode_shell_command]`) |
| 0 | Back to main menu |
