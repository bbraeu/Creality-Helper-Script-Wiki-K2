# Restore a Previous Firmware — K2

If a firmware update causes issues, you can restore a previous version using the same USB installation method used for regular updates.

!!! warning "Back up first"
    Restoring firmware can trigger a factory reset, which wipes `/mnt/UDISK/`. Back up your Klipper config **to your computer** before proceeding, e.g. create a backup with helper option 20 and download it:
    ```bash
    scp root@<printer-ip>:/mnt/UDISK/helper-script/backups/klipper_config_*.tar.gz ./
    ```

---

## Finding Previous Firmware

!!! danger "Never flash K2 Plus (or K2 Pro) firmware on a K2"
    Only use firmware that Creality publishes for the **K2 / K2 Combo**. Firmware for the K2 Plus or K2 Pro targets different hardware and must never be installed on a K2.

Previous K2 firmware versions are listed under **Older Versions** on Creality's official K2 download page:

[Creality — K2 Combo / K2 Downloads :material-open-in-new:](https://www.creality.com/download/k2-series-3d-printer){ .md-button }

If that link moves, open the [Creality Download Center](https://www.creality.com/download) and select **K2**. Download the firmware file for the version you want to restore.

---

## Restore Procedure

1. Format a USB drive as **FAT32** with 4096 allocation size.

2. Copy the firmware file to the **root** of the USB drive (not inside any folder).

3. Turn on the printer and wait for the home screen.

4. Plug the USB drive into the **front** USB port.

5. A firmware update prompt will appear. Accept it and wait for the process to complete — the printer will restart automatically.

6. After restart, re-enable root access:
    **Settings → Root account information**

7. Reinstall the helper script and all features. See [Install Helper Script](../helper-script/helper-script-installation.md).

---

## Firmware Recovery (Brick Recovery)

If the printer will not boot at all, see the stock Creality recovery procedure using a USB drive with a specific file layout. Contact Creality support or check [Creality Cloud](https://www.crealitycloud.com/) for the latest recovery image for the K2.

---

## After Restoring

After any firmware restore or factory reset:

- Root access password resets to **`creality_2024`**
- All data under `/mnt/UDISK/printer_data/` is wiped
- The helper script and all installed features must be reinstalled from scratch
- Run **Settings → Self-check** before printing
