# Access to Web Interface — K2

The K2 ships with **Fluidd pre-installed** at port `4408`. No installation is required to access it.

---

## Fluidd Web Interface

Fluidd is the primary web interface on the K2.

Open your browser and navigate to:

```
http://<printer-ip>:4408
```

Replace `<printer-ip>` with your printer's IP address (found in **Settings → Network** on the touchscreen).

<img src="../../assets/img/Access-to-Web-Interface/Fluidd_Web_Interface.png">

---

## Mainsail Web Interface

Mainsail is not pre-installed. Install it via the Helper Script (option 9) to access it on port `4409`:

```
http://<printer-ip>:4409
```

<img src="../../assets/img/Access-to-Web-Interface/Mainsail_Web_Interface.png">

---

## Creality Web Interface

The original Creality web interface is accessible on port 80:

```
http://<printer-ip>
```

This is used by **Creality Print** for WiFi printing.

!!! note "No port-80 replacement in this helper script"
    The K1 helper script had a Customize Menu to replace the Creality web interface on port 80 with Fluidd or Mainsail. The K2 helper script has no such option — the Creality web interface stays on port 80 so that Creality Print keeps working. Use ports `4408` (Fluidd) and `4409` (Mainsail).
