# Files Location — K2

All persistent data on the K2 lives under `/mnt/UDISK/`. There is no `/usr/data/` path on the K2 — this differs from the K1 Series.

Connect via [SSH](../firmwares/ssh-connection.md) to access these locations.

!!! note "Collected on the K2 Plus"
    These paths were collected on the K2 Plus. The `printer_data` paths and services are shared across the K2 family; the Creality system data paths have not all been verified on the K2 base model.

---

## Klipper & Printer Data

| Resource | Path |
|---|---|
| Klipper configuration files | `/mnt/UDISK/printer_data/config/` |
| GCode files | `/mnt/UDISK/printer_data/gcodes/` |
| Klipper log | `/mnt/UDISK/printer_data/logs/klippy.log` |
| Moonraker log | `/mnt/UDISK/printer_data/logs/moonraker.log` |
| Moonraker database | `/mnt/UDISK/printer_data/database/` |
| Moonraker timelapse videos | `/mnt/UDISK/printer_data/timelapse/` |

---

## Helper Script Data

| Resource | Path |
|---|---|
| Helper script, go2rtc, camera watchdog | `/mnt/UDISK/helper-script/` |
| Helper backups (config, Moonraker database) | `/mnt/UDISK/helper-script/backups/` |
| Installed features list | `/mnt/UDISK/helper-script/.installed` |
| Git Backup token | `/mnt/UDISK/helper-script/.git-credentials` |
| Mobileraker Companion | `/mnt/UDISK/mobileraker_companion/`, `/mnt/UDISK/mobileraker-env/` |
| OctoEverywhere / OctoApp | `/mnt/UDISK/octoeverywhere/`, `/mnt/UDISK/octoapp/` |
| Entware | `/opt/` |
| Helper procd services | `/etc/init.d/mobileraker`, `/etc/init.d/git-backup` |
| Camera bridge startup | `/etc/rc.d/S99camera` |

---

## Creality System Data

| Resource | Path |
|---|---|
| System config (self-check flag, etc.) | `/mnt/UDISK/creality/userdata/config/system_config.json` |
| Creality timelapse videos | `/mnt/UDISK/creality/userdata/delay_image/video/` |
| AI image data | `/mnt/UDISK/ai_image/` |
| Layer image data | `/mnt/UDISK/layers_image/` |

---

## Read-Only System Paths

These paths live on a read-only partition and **cannot be edited in place**. Modifying them requires overlaying or replacing the binary/file via the helper script approach.

| Resource | Path |
|---|---|
| Klipper Python env | `/usr/share/klippy-env/bin/python` |
| Klipper source | `/usr/share/klipper/klippy/klippy.py` |
| Klipper config templates | `/usr/share/klipper/config/<board-id>/` — `F008_CR0CN240319C13_1` on the K2 Plus; Creality's open-source repo names the K2 directory `F021_CR0CN200400C10`. Check with `ls /usr/share/klipper/config/` |
| Moonraker Python env | `/usr/share/moonraker-env/bin/python` |
| Moonraker source | `/usr/share/moonraker/` |
| Moonraker config | `/usr/share/moonraker/moonraker.conf` |
| Fluidd static files | `/usr/share/fluidd/` |
| Nginx config | `/etc/nginx/nginx.conf` |

---

## Service Scripts

The K2 uses OpenWrt-style rc.d init scripts. There is no `systemd` or `supervisord`.

| Service | Start script | Stop script |
|---|---|---|
| klipper MCU bridge | `/etc/init.d/S54klipper_mcu` | `/etc/rc.d/K54klipper_mcu` (if present) |
| Klipper (klippy) | `/etc/init.d/S55klipper` | — |
| Moonraker | `/etc/init.d/S56moonraker` | — |
| Nginx | `/etc/init.d/S80nginx` | — |
| WebRTC (camera) | `/etc/init.d/S97webrtc` | `/etc/rc.d/K97webrtc` |

To restart a service over SSH:

```bash
/etc/init.d/S55klipper restart
/etc/init.d/S56moonraker restart
/etc/init.d/S80nginx restart
```

!!! note "No supervisorctl on K2"
    Unlike the K1 Series, the K2 does not use Supervisor Lite. The `supervisorctl` command is not available. Use the rc.d init scripts above instead.
