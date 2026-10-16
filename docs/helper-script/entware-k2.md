# Entware Package Manager — K2

Entware is a software package manager for embedded systems. It provides hundreds of Linux packages via `opkg` on the K2.

!!! success "Tested on the K2 Plus"
    Entware was successfully installed on the K2 Plus using a Python-based wget shim. The K2 uses the same firmware family and installer.

!!! note "Needed by options 14–17"
    OctoEverywhere, Mobileraker Companion, OctoApp Companion and Git Backup all install their dependencies (`git`, `python3`, `inotifywait`, `ffmpeg`, ...) with Entware. Install Entware (option 13) first.

---

## Background

The K2 uses **armhf** (hard-float ARM ABI). Standard Entware installation requires `wget` which is not present on the K2. The solution uses a Python script to simulate `wget` for the bootstrap installer, after which the real `wget` is installed via `opkg`.

Full credit to [vsevolod-volkov](https://github.com/vsevolod-volkov/K2Plus-entware) for this approach.

---

## Installation

Install from the helper script:

```sh
sh /mnt/UDISK/helper-script/helper.sh
```

Select **13) Entware Package Manager**.

The installer will:

1. Create a Python-based dummy `wget` for bootstrapping
2. Run the Entware installer
3. Install the real `wget` via `opkg`
4. Add `/opt/bin` and `/opt/sbin` to PATH permanently
5. Configure Entware services to start on boot

---

## Installing Packages

The installer offers to install useful packages right away. Later, select option 13 again — it then shows a small menu (*Install useful packages*, *Update package list*, *Open opkg shell*):

| Package | Description |
|---|---|
| `nano` | Better text editor than vi |
| `htop` | Interactive process monitor |
| `git` | Version control |
| `openssh-sftp-server` | SFTP file transfer access |
| `curl` | HTTP client |

Or install any package manually via SSH:

```bash
export PATH=/opt/bin:/opt/sbin:$PATH
opkg update
opkg install <package-name>
```

---

## After Reboot

Entware PATH is added to `rc.local` automatically. After reboot, `opkg` and all installed packages are available immediately.

---

## Updating Packages

Use **Tools menu → 30) Update Entware packages** (`opkg update && opkg upgrade`).

---

## Useful Commands

```bash
opkg update              # Update package list
opkg install <package>   # Install a package
opkg remove <package>    # Remove a package
opkg list                # List all available packages
opkg list-installed      # List installed packages
```

---

## Credits

- [vsevolod-volkov/K2Plus-entware](https://github.com/vsevolod-volkov/K2Plus-entware) — wget shim solution
- [Entware project](https://github.com/Entware/Entware) — package manager
