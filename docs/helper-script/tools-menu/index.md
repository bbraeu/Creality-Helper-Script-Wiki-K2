# Tools Menu

The `[Tools] Menu` (options 24–32) collects maintenance and diagnostic helpers.

| Option | Tool | What it does |
|---|---|---|
| 24 | Restart Klipper | `/etc/rc.d/S55klipper restart`, then checks that `klippy.py` is running |
| 25 | Restart Moonraker | Stops all running Moonraker instances and starts Moonraker again |
| 26 | Restart Nginx | `/etc/rc.d/S80nginx restart` |
| 27 | View Klipper log | Last 50 lines of `/mnt/UDISK/printer_data/logs/klippy.log` |
| 28 | View Moonraker log | Last 50 lines of `/mnt/UDISK/printer_data/logs/moonraker.log` |
| 29 | Show installed features | Lists the features recorded in `/mnt/UDISK/helper-script/.installed` |
| 30 | Update Entware packages | `opkg update && opkg upgrade` (requires [Entware](../entware-k2.md)) |
| 31 | Clear logs & cache | Frees space on `/mnt/UDISK` (see below) |
| 32 | System information | Printer and system overview (see below) |

---

## Clear Logs & Cache (option 31)

After a confirmation, the script:

- deletes rotated logs (`*.log.*`, `*.gz`) in `/mnt/UDISK/printer_data/logs/`
- **truncates** the live logs (`*.log`) instead of deleting them — Klipper and Moonraker keep them open and would stop logging until restarted
- removes the camera bridge logs in `/tmp`
- clears `/root/.cache` and `/mnt/UDISK/.cache`
- purges the pip cache (if Entware's `pip3` is installed)

It then reports how many MB were freed on `/mnt/UDISK`.

!!! tip
    Download `klippy.log` / `moonraker.log` first if you still need them for a bug report.

---

## System Information (option 32)

Shows:

| Field | Source |
|---|---|
| Printer | Detected model, bed size and chamber heater, e.g. `K2 (no chamber heater, bed mesh 20,20 to 255,255)` |
| Hostname | `/proc/sys/kernel/hostname` |
| IP address | Current source IP of the default route |
| OS | `/etc/openwrt_release` |
| Kernel | `uname -r` and architecture |
| Uptime / Load | `uptime`, `/proc/loadavg` |
| Memory | Used / total RAM |
| UDISK | Used / total space on `/mnt/UDISK` |
| Entware | Installed or not |

---

## Related

- [Restore a Previous Firmware](../restore-a-previous-firmware-k2.md) — not a menu option; links to the manual procedure
- [Reset Factory Settings](../../firmwares/reset-factory-settings.md)
- [SSH Connection](../../firmwares/ssh-connection.md)
