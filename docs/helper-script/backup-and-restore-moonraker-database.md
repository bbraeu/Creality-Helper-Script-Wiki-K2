# Backup & Restore Moonraker Database — K2

Moonraker keeps its state in a database under `/mnt/UDISK/printer_data/database/`: print history and statistics, the job queue, Fluidd / Mainsail UI settings, webcam entries, announcements and more. The helper script can save and restore it.

---

## Backup from Helper Script

From the helper script menu, select **option 22 — Backup Moonraker database**.

The script archives `/mnt/UDISK/printer_data/database/` to:

```
/mnt/UDISK/helper-script/backups/moonraker_database_<timestamp>.tar.gz
```

The last 5 database backups are kept automatically (independently of the Klipper configuration backups).

---

## Restore from Helper Script

From the helper script menu, select **option 23 — Restore Moonraker database**.

1. The script lists the available database backups with dates and sizes.
2. Select one and confirm.
3. Moonraker is **stopped first**, the current `database/` folder is replaced by the backup, and Moonraker is started again.

!!! warning "Restoring overwrites the current database"
    Print history and UI settings recorded after the backup was made are lost. Do not restore while a print is running.

---

## Download Backups

```bash
scp root@<printer-ip>:/mnt/UDISK/helper-script/backups/moonraker_database_20260601_120000.tar.gz ./
```

!!! note "Factory reset"
    A factory reset wipes `/mnt/UDISK/`, including the backups folder. Copy the archives to your computer first.
