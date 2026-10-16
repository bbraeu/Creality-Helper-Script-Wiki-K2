# HelixScreen — K2

HelixScreen is a modern, lightweight touchscreen interface for Klipper 3D printers. It replaces the stock Creality touchscreen UI with a feature-rich Klipper-native interface.

!!! note "Tested on the K2 Plus"
    HelixScreen v0.99.72 was tested and confirmed working on the Creality K2 Plus. HelixScreen supports the K2 series; it has not yet been verified with this wiki on the K2 base model.

---

## Features

- Modern touch UI for Klipper
- Temperature control and monitoring
- Print management and history
- Bed mesh visualization
- Input shaper graphs
- Multi-material (CFS) management
- Auto-updates via Fluidd/Mainsail update manager

---

## Installation

Install from the helper script: select **12) HelixScreen (touchscreen UI)**.

!!! warning "First boot"
    The printer reboots after the install. The first start can take longer than normal and may stay on the Creality logo for several minutes — do **not** power off. If it stays stuck, use the `wipe_all` USB recovery (see [Reset Factory Settings](../firmwares/reset-factory-settings.md#method-3-usb-drive)).

Or install manually via SSH:

```bash
python3 -c "import urllib.request as u; open('/tmp/install.sh','wb').write(u.urlopen(u.Request('http://dl.helixscreen.org/install.sh', headers={'User-Agent':'helixscreen-installer/1.0'}), timeout=30).read())" && sh /tmp/install.sh
```

Reboot after installation:

```bash
reboot
```

---

## Known Limitations

**WiFi management unavailable** — HelixScreen showed a "WiFi unavailable" warning on the K2 Plus (expected on the K2 as well, same firmware family). This is cosmetic only — your printer's WiFi connection is not affected. The K2 family uses a proprietary WiFi management system that HelixScreen cannot access.

---

## Updating

To update HelixScreen run the helper script and select option 12 again (it offers *Update HelixScreen*), or via SSH:

```bash
/opt/helixscreen/install.sh --update
```

HelixScreen's own `[update_manager helixscreen]` entry is removed by the helper script because it is not supported on the K2, so updates are done from the helper script or SSH.

---

## Removing

To restore the stock Creality touchscreen UI:
Select **19) Remove a feature** → **12) HelixScreen**, or via SSH:

```bash
/opt/helixscreen/install.sh --uninstall
reboot
```

---

## More Information

- [HelixScreen Website](https://helixscreen.org)
- [HelixScreen GitHub](https://github.com/prestonbrown/helixscreen)
- [K2 Series Documentation](https://helixscreen.org/dev/printers/creality-k2/)
