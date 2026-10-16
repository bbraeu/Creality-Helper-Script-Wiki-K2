# Install Helper Script — K2

The Creality K2 Helper Script is a native shell-based tool for the K2 and K2 Combo. It installs features by writing Klipper `.cfg` files to `/mnt/UDISK/printer_data/config/` and extending Moonraker, and it detects your printer's hardware from `printer.cfg` (see [Hardware Detection](index.md#hardware-detection)).

!!! warning "K2 family only"
    This helper script is written for the **Creality K2** and **K2 Combo**; it also runs on the K2 Pro and K2 Plus.
    It will not work on K1, K1C, K1 Max, or Ender-3 V3 Series printers.

!!! note "Before you begin"
    - Root access must be enabled. See [Enable Root Access](../firmwares/install-and-update-rooted-firmware-k2.md#enable-root-access).
    - The printer must have completed its startup sequence and be on the home screen.
    - An internet connection is required for features that download additional components (Mainsail, KAMP-K2, Timelapse, go2rtc, Entware packages).

---

## How It Works

| What it does | How |
|---|---|
| Extends Moonraker | Writes `/mnt/UDISK/printer_data/config/moonraker.conf` with `[include /usr/share/moonraker/moonraker.conf]` at the top and points the Moonraker startup script (`S56moonraker`) at it. It also makes sure the stock config includes the UDISK file, so `[timelapse]` and `[update_manager]` sections there are actually loaded. |
| Adds Klipper features | Writes `.cfg` files to `/mnt/UDISK/printer_data/config/` and adds `[include]` lines to `printer.cfg` |
| Adds services | OpenWrt procd services in `/etc/init.d/` (Mobileraker Companion, Git Backup) and an rc.d script for the camera bridge |
| Service restarts | Uses `/etc/rc.d/S55klipper`, Moonraker's init script and `/etc/rc.d/S80nginx` — **not** `supervisorctl` (which does not exist on the K2) |
| Uninstall | Every feature can be removed from the Remove menu (option 19) |
| Stock files | Some stock files **are** patched: macro names in `gcode_macro.cfg` / `box.cfg` (see below), `mesh_min` in `printer.cfg`, `nginx.conf` and Fluidd/Mainsail `index.html` (Camera Support), `gcode_macro.cfg` (KAMP-K2, backed up as `gcode_macro.cfg.pre_kamp`), and Klipper's `extras` folder (KAMP-K2, Gcode Shell Command). |

### What runs every time the script starts

Each start of `helper.sh` (and every boot, via `/etc/rc.local`) re-applies:

- **Stock config fixes** — removes the leading underscore from stock CFS / print macros in `box.cfg` and `gcode_macro.cfg` (they were unreachable, causing CFS load/unload errors), changes a stock `mesh_min: 5,5` to `20,20`, restores `motor_control.cfg` from this model's stock copy in `/rom` if it was truncated, and links `/usr/bin/wget` to Entware's `wget` (needed by Timelapse).
- **Add-ons outside `/mnt/UDISK`** — a firmware update replaces `/usr/share/klipper`, so the KAMP-K2 and Gcode Shell Command Klipper extras and the KAMP-K2 macro patches are re-applied (Klipper is restarted automatically only at boot).
- **Moonraker include** — makes sure the stock Moonraker config loads the UDISK `moonraker.conf`, and enables `queue_gcode_uploads`.

---

## Installation

Connect via SSH (user `root`, password `creality_2024`). See [SSH Connection](../firmwares/ssh-connection.md).

The K2 stock firmware has no `git`, `wget`, or `curl`, so Python 3 is used to download the script. Run:

```sh
python3 -c "import urllib.request; urllib.request.urlretrieve('https://github.com/bbraeu/Creality-Helper-Script-K2-Base/archive/refs/heads/main.zip', '/tmp/helper.zip')"
python3 -c "import shutil; shutil.unpack_archive('/tmp/helper.zip', '/tmp/')"
mkdir -p /mnt/UDISK/helper-script
cp -r /tmp/Creality-Helper-Script-K2-Base-main/* /mnt/UDISK/helper-script/
chmod +x /mnt/UDISK/helper-script/helper.sh
sh /mnt/UDISK/helper-script/helper.sh
```

!!! note "Install path"
    The helper script installs to `/mnt/UDISK/helper-script/` — not `/usr/data/helper-script/` as on K1 Series.

---

## Main Menu

When you run the script you will see:

```
======================================================
   Creality K2 Helper Script
======================================================
   https://bbraeu.github.io/Creality-Helper-Script-Wiki-K2-Base
======================================================
   Detected: K2 (no chamber heater, bed mesh 20,20 to 255,255)

  [Install] Menu
  --- Step 1: Foundation (install first) ---
    1) Moonraker Extensions                  [recommended first]

  --- Step 2: Print macros ---
    2) Fans Control Macros
    3) Useful Macros (PID, leveling, warmup)
    4) Save Z-Offset Macros
    5) M600 Filament Change Support

  --- Step 3: Leveling & calibration ---
    6) Klipper Adaptive Meshing & Purging (KAMP-K2)
    7) Improved Shapers Calibrations

  --- Step 4: Web interface & camera ---
    8) Fluidd (install/update/repair — port 4408)
    9) Mainsail (install/update/repair — port 4409)
   10) Moonraker Timelapse
   11) Camera Support (Fluidd, Mainsail, Mobileraker)
   12) HelixScreen (touchscreen UI)

  --- Step 5: Remote access, notifications & backup ---
   13) Entware Package Manager             [needed by 14-17]
   14) OctoEverywhere
   15) Mobileraker Companion
   16) OctoApp Companion
   17) Git Backup
   18) Klipper Gcode Shell Command

  [Remove] Menu
   19) Remove a feature

  [Backup & Restore] Menu
   20) Backup Klipper configuration
   21) Restore Klipper configuration
   22) Backup Moonraker database
   23) Restore Moonraker database

  [Tools] Menu
   24) Restart Klipper
   25) Restart Moonraker
   26) Restart Nginx
   27) View Klipper log
   28) View Moonraker log
   29) Show installed features
   30) Update Entware packages
   31) Clear logs & cache
   32) System information
   33) Update helper script               [installed: f0577de]

    0) Exit

  Enter choice:
```

---

## Recommended Installation Order

The menu is arranged in dependency order — install top to bottom:

1. **Moonraker Extensions** (option 1) — always first. Creates the persistent Moonraker config that other features (Timelapse, KAMP-K2, Mobileraker Companion) add sections to. KAMP-K2 installs it automatically if it is missing.
2. **Print macros** (options 2–5) — optional, in any order. They do not replace the stock `START_PRINT` / `END_PRINT` (those stay in the stock `gcode_macro.cfg` with full CFS integration).
3. **KAMP-K2** (option 6) — then update your slicer start G-code, see [KAMP-K2](klipper-adaptive-meshing-and-purging-k2.md) and [OrcaSlicer](../slicers/orcaslicer.md).
4. **Improved Shapers** (option 7) — independent, any time.
5. **Fluidd, Mainsail, Timelapse, Camera, HelixScreen** (options 8–12) — any order, any time.
6. **Entware** (option 13) — before options 14–17.
7. **Klipper Gcode Shell Command** (option 18) — install it **before** Git Backup (option 17) if you want the `GIT_BACKUP_*` macros.
8. **OctoEverywhere, Mobileraker Companion, OctoApp Companion, Git Backup** (options 14–17) — any order.

---

## Update

The helper script updates itself. The commit it was installed from is stored in `/mnt/UDISK/helper-script/.version`. Every time `helper.sh` starts, it compares that commit with the latest one on GitHub, lists the new changes and asks **Update now? [Y/n]** — press Enter to update; the helper then restarts with the new files.

Other ways to update:

- **Tools → 33) Update helper script** (shows the installed version)
- Over SSH, without prompts:

    ```sh
    sh /mnt/UDISK/helper-script/helper.sh --update
    ```

Only the script files are replaced. Your installed-feature list (`.installed`), backups, the Git Backup token and the generated camera files (`go2rtc`, `go2rtc.yaml`, `camera_watchdog.py`) are kept. To skip the check (e.g. without internet), start with `HELPER_SKIP_UPDATE=1 sh /mnt/UDISK/helper-script/helper.sh`.

After an update, reinstall a feature (select it again) to pick up changes to the files it generates (macros, camera setup).

---

## Uninstall Everything

1. Run the script and use the **Remove menu** (option 19) to remove each installed feature (Moonraker Extensions last — removing it restores the stock Moonraker startup).
2. Remove the helper script directory:

    ```bash
    rm -rf /mnt/UDISK/helper-script
    ```

3. Remove the lines the script added to `/etc/rc.local` (the `patch_stock_configs` and `install_moonraker_include` calls) — otherwise they fail harmlessly on every boot.

---

## Installed Features Location

| Feature | Files | Location |
|---|---|---|
| Moonraker Extensions | `moonraker.conf` | `/mnt/UDISK/printer_data/config/` |
| Fans Control Macros | `fans_control.cfg` | `/mnt/UDISK/printer_data/config/` |
| Useful Macros | `useful_macros.cfg` | `/mnt/UDISK/printer_data/config/` |
| Save Z-Offset Macros | `z_offset_macros.cfg` | `/mnt/UDISK/printer_data/config/` |
| M600 Support | `m600.cfg` | `/mnt/UDISK/printer_data/config/` |
| KAMP-K2 | `KAMP/` (+ `restore_bed_mesh.py` in Klipper extras) | `/mnt/UDISK/printer_data/config/` |
| Improved Shapers | `shapers_calibration.cfg` | `/mnt/UDISK/printer_data/config/` |
| Moonraker Timelapse | `timelapse.cfg` (+ `timelapse.py` in `/usr/share/moonraker/components/`) | `/mnt/UDISK/printer_data/config/` |
| Mainsail | static files | `/usr/share/mainsail/` |
| Camera Support | `go2rtc`, `go2rtc.yaml`, `camera_watchdog.py` | `/mnt/UDISK/helper-script/` |
| Mobileraker Companion | `mobileraker_companion/`, `mobileraker-env/`, `mobileraker.conf` | `/mnt/UDISK/`, config folder |
| OctoEverywhere / OctoApp | `octoeverywhere/`, `octoapp/` | `/mnt/UDISK/` |
| Git Backup | `git_backup.cfg` (macros), `.git-credentials` (token) | config folder, `/mnt/UDISK/helper-script/` |
| Klipper Gcode Shell Command | `gcode_shell_command.py` | `/usr/share/klipper/klippy/extras/` |
| Backups | `klipper_config_*.tar.gz`, `moonraker_database_*.tar.gz` | `/mnt/UDISK/helper-script/backups/` |
| Installed feature log | `.installed` | `/mnt/UDISK/helper-script/` |

---

## Troubleshooting

**Klipper fails to start after installing a feature**

Check the log (or use option 27):
```bash
tail -50 /mnt/UDISK/printer_data/logs/klippy.log
```
Most errors are caused by a problem in a new `.cfg` file. The log shows the exact line.

**Moonraker fails to start after installing extensions**

Check the log (or use option 28):
```bash
tail -50 /mnt/UDISK/printer_data/logs/moonraker.log
```

**Script says "printer.cfg not found"**

The printer has not finished booting. Wait until you can see the Fluidd interface, then run the script again.

**"Entware is required"**

Options 14–17 need Entware. Install it first with option 13.

**Permission denied running helper.sh**

```bash
chmod +x /mnt/UDISK/helper-script/helper.sh
sh /mnt/UDISK/helper-script/helper.sh
```
