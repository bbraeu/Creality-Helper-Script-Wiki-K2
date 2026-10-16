# Backup & Restore Klipper Configuration Files — K2

Regular backups protect your Klipper configuration from accidental deletion, factory resets, and firmware updates.

---

## Backup from Helper Script

From the helper script menu, select **option 21 — Backup Klipper configuration**.

The script creates a compressed archive of your entire `/mnt/UDISK/printer_data/config/` directory and saves it as `/mnt/UDISK/helper-script/backups/klipper_config_<timestamp>.tar.gz`. The last 5 configuration backups are kept automatically.

---

## Restore from Helper Script

From the helper script menu, select **option 22 — Restore Klipper configuration**.

The script shows a numbered list of available backups with dates and sizes. Select the one you want to restore, confirm, and Klipper restarts automatically.

---

## Backup via SSH

```bash
# Create a timestamped backup
tar -czf /mnt/UDISK/printer_data/config_backup_$(date +%Y%m%d_%H%M%S).tar.gz \
    -C /mnt/UDISK/printer_data config/

# List backups
ls -lh /mnt/UDISK/printer_data/config_backup_*.tar.gz
```

---

## Download Backups

You can download config backups via:

- **SCP** from your computer:
    ```bash
    scp root@<printer-ip>:/mnt/UDISK/helper-script/backups/klipper_config_20260601_120000.tar.gz ./
    ```
- **MobaXterm** — use the left panel file browser to download any file
- **Fluidd File Manager** — only for archives you created inside the config folder (see *Backup via SSH*)

For an off-printer copy that updates itself, see [Git Backup](git-backup.md).

---

## What to Back Up

At minimum, back up these files before any firmware update or factory reset:

| File | Contains |
|---|---|
| `printer.cfg` | Main Klipper config including the SAVE_CONFIG block (Z offset, input shaper, bed mesh, CFS cut position) |
| `box.cfg` | CFS positions and calibration (K2 Combo) |
| `gcode_macro.cfg` | Stock Creality macros |
| `printer_params.cfg` | Print parameters |
| `useful_macros.cfg` | Helper script macros |
| `moonraker.conf` | Moonraker extensions config |

The `SAVE_CONFIG` block at the bottom of `printer.cfg` is especially important — it contains your calibrated values for bed mesh, input shaper, and CFS cut position.

!!! warning "After factory reset"
    All data under `/mnt/UDISK/` — including `/mnt/UDISK/helper-script/backups/` — is wiped by a factory reset. Always download a backup to your computer before performing a reset.
