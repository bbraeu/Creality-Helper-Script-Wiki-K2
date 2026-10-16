# Restore a Previous Firmware — K2

!!! note "Not a helper script tool"
    The K2 helper script has **no firmware restore option** in its Tools menu. Firmware is installed and downgraded with a USB drive from the touchscreen. Follow [Restore Previous Firmware](../firmwares/restore-previous-firmware-k2.md).

!!! danger "Never flash K2 Plus (or K2 Pro) firmware on a K2"
    Only use firmware published for the **K2 / K2 Combo**. Firmware for the K2 Plus or K2 Pro targets different hardware and must never be installed on a K2.

Get K2 firmware (current and older versions) from Creality's official download page:

[Creality — K2 Combo / K2 Downloads :material-open-in-new:](https://www.creality.com/download/k2-series-3d-printer){ .md-button }

After restoring firmware, re-enable root access and start the helper script again. If `/mnt/UDISK` survived, the script re-applies the KAMP-K2 and Gcode Shell Command Klipper extras when it starts; features that live in `/usr/share` or `/etc` (Camera Support, Timelapse, Mainsail) may need to be reinstalled. If a factory reset wiped `/mnt/UDISK`, reinstall the helper script and all features from scratch.
