# Backup & Restore Menu

The `[Backup & Restore] Menu` creates and restores compressed archives on the printer. All archives are stored in `/mnt/UDISK/helper-script/backups/`, and the **last 5 of each kind** are kept — older ones are deleted automatically.

| Option | Action | Archive | Page |
|---|---|---|---|
| 20 | Backup Klipper configuration | `klipper_config_<timestamp>.tar.gz` | [Klipper configuration](../backup-and-restore-klipper-configuration-files.md) |
| 21 | Restore Klipper configuration | — | [Klipper configuration](../backup-and-restore-klipper-configuration-files.md) |
| 22 | Backup Moonraker database | `moonraker_database_<timestamp>.tar.gz` | [Moonraker database](../backup-and-restore-moonraker-database.md) |
| 23 | Restore Moonraker database | — | [Moonraker database](../backup-and-restore-moonraker-database.md) |

!!! warning "Backups stay on the printer"
    `/mnt/UDISK/` is wiped by a factory reset. Copy important archives to your computer (SCP or MobaXterm), or use [Git Backup](../git-backup.md) to keep the config folder on GitHub.
