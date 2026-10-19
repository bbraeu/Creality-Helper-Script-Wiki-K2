# Fluidd — K2

Fluidd is the primary web interface on the K2, pre-installed at port `4408` from the factory. This page covers updating, repairing, and restoring Fluidd using the Helper Script.

!!! note "Pre-installed"
    You do not need to install Fluidd — it ships with the K2. This page is for updating to the latest version or repairing a broken installation.

---

## Access Fluidd

Open your browser and navigate to:

```
http://<printer-ip>:4408
```

Replace `<printer-ip>` with your printer's IP address (found in **Settings → Network** on the touchscreen).

---

## Update / Repair Fluidd

From the helper script menu, select **option 8 — Fluidd (install/update/repair)**. 

When Fluidd is already installed you will be offered:

- **Update** — download and install Fluidd (version: see below)
- **Repair** — re-download and reinstall it (fixes corrupted files)
- **Restore nginx block only** — fixes port 4408 access without re-downloading Fluidd

!!! warning "Fluidd 1.37+ and Creality's Moonraker"
    From Fluidd 1.37 on, editing config files fails against the K2's stock (Creality) Moonraker. With the stock Moonraker the helper therefore installs **Fluidd v1.36.4**, the newest version whose config editor works there. With the [updatable Moonraker](moonraker-k2.md) (Install menu 1 → 2) it installs the **latest** Fluidd, and later updates come from **Settings → Software Updates**.

---

## Manual Update via SSH

The stock firmware has no `wget`; the commands below need [Entware](entware-k2.md) (`wget` is installed with it). The helper script (option 8) downloads with Python instead.

```bash
# Download Fluidd v1.36.4 (Creality's Moonraker) - with the updatable Moonraker
# use https://github.com/fluidd-core/fluidd/releases/latest/download/fluidd.zip
wget -O /tmp/fluidd.zip https://github.com/fluidd-core/fluidd/releases/download/v1.36.4/fluidd.zip

# Install
mkdir -p /usr/share/fluidd
unzip -q -o /tmp/fluidd.zip -d /usr/share/fluidd
rm /tmp/fluidd.zip

# Restart nginx
/etc/init.d/S80nginx restart
```

---

## Fluidd Configuration

Key Fluidd settings are stored in Moonraker. Once Moonraker Extensions are installed, configure Fluidd from **Settings** in the web interface.

### Camera Setup

Install **option 11 — Camera Support** to get the K2 camera in Fluidd. It registers the webcam in Moonraker and enables it automatically; see [Camera Support](../configurations/configure-camera.md). If the camera does not show up, check **Settings → Cameras** and make sure the **K2 Camera** entry is enabled.

---

## Nginx Configuration

Fluidd is served by nginx from `/usr/share/fluidd/` on port `4408`. The nginx config is at `/etc/nginx/nginx.conf`.

The K2 nginx config proxies all Moonraker API requests (`/printer/`, `/api/`, `/machine/`, etc.) to `127.0.0.1:7125`, webcam streams to `127.0.0.1:8080–8083`, and — with Camera Support installed — `/go2rtc/` to go2rtc on port 1984.

!!! warning "Do not edit nginx.conf directly"
    `/etc/nginx/nginx.conf` is on the writable partition but the Helper Script backs it up before modifying it. If you edit it manually and break nginx, restore the backup:
    ```bash
    cp /mnt/UDISK/helper-script/.nginx.conf.bak /etc/nginx/nginx.conf
    /etc/init.d/S80nginx restart
    ```
