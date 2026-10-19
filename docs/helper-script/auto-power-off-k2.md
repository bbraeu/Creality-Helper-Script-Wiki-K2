# Auto Power Off — K2

Option 19 of the helper script switches the K2 off with a **smart plug** once a print has finished and nozzle and bed have cooled down. It is a K2 version of [tinntbg/auto-power-off-klipper](https://github.com/tinntbg/auto-power-off-klipper): a few Klipper macros wait for the end of the print and the cool-down, then tell the plug to switch off.

- Starting a new print or turning a heater on **cancels** a pending power off.
- A **cancelled** or **failed** print does not power off — you probably want to look at the printer first.
- A **paused** print keeps waiting (e.g. during an M600 filament change).
- The K2 cannot switch itself back on: switch the plug on again in its app (or Home Assistant).

---

## Supported Smart Plugs

| Plug | How it is switched | Login |
|---|---|---|
| **TP-Link Kasa — classic** (HS100, HS110, KP105, KP115, …) | Moonraker power device `type: tplink_smartplug` | None |
| **TP-Link Kasa — newer models / firmware** (KP125M, EP25, …) and **Tapo** | `kasa.py` (helper script) via `RUN_SHELL_COMMAND` | TP-Link account |
| **Tasmota** | Moonraker power device `type: tasmota` | Web password, if set |
| **Shelly** | Moonraker power device `type: shelly` | User / password, if set |
| **Home Assistant** entity (any plug HA can switch) | Moonraker power device `type: homeassistant` | Long-lived access token |
| **Power device already in `moonraker.conf`** | Moonraker, by its name | — |

Newer Kasa plugs and all Tapo plugs only speak TP-Link's encrypted **KLAP** protocol, which Moonraker cannot switch. For those the helper ships `scripts/kasa.py`, a small client that only needs the Python standard library (the K2's stock Python has no crypto module, so it brings its own AES-128).

!!! tip "Give the plug a fixed IP address"
    The plug's IP address is written into the configuration. Reserve it in your router (DHCP reservation), otherwise Auto Power Off stops working after the plug gets a new address.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section. For every plug type except KLAP Kasa / Tapo, [Moonraker Extensions](moonraker-k2.md) (option 1) must be installed first — the plug becomes a Moonraker power device in the persistent `moonraker.conf`.

From the `[Install] Menu` select **option 19 — Auto Power Off** and choose the plug type:

```
1) TP-Link Kasa (with network search; also Tapo)
2) Tasmota
3) Shelly
4) Home Assistant entity
5) Power device already in moonraker.conf
0) Cancel
```

### TP-Link Kasa / Tapo

1. The installer asks whether to **search the network** (5 seconds). Found plugs are listed with IP address, model, name and what they need:
    - *no login needed* — classic Kasa plug, becomes a Moonraker `tplink_smartplug` device.
    - *needs TP-Link account* — KLAP plug, switched by `kasa.py`.
    - *… encryption — not supported* — update the plug's firmware in the Kasa / Tapo app and try again.
2. Pick a plug, or press Enter and type the IP address yourself (the installer then probes the plug).
3. For a **KLAP** plug, enter the e-mail and password of the TP-Link account the plug is registered to. The installer logs in to the plug to test it (three tries).
    - Both e-mail and password are case-sensitive.
    - For **Tapo** plugs, enable **Third-Party Compatibility** in the Tapo app first.
    - The password itself is **not stored** — only the KLAP auth hash derived from it, in `/mnt/UDISK/helper-script/.kasa.json` (readable by root only, not in the printer config and therefore not in [Git Backup](git-backup.md)).
    - [Klipper Gcode Shell Command](gcode-shell-command.md) is installed automatically if missing, because Klipper calls `kasa.py` through `RUN_SHELL_COMMAND`.

### Tasmota, Shelly, Home Assistant

Enter the address and (if set) the login. For Home Assistant you need the **entity ID** of the plug (e.g. `switch.printer`) and a **long-lived access token** (HA profile → *Security*). The installer adds a `[power printer_plug]` section to `moonraker.conf`, restarts Moonraker and reports what Moonraker sees:

- `Moonraker reports power device 'printer_plug': on` — working.
- `Moonraker cannot reach power device …` — wrong address or login; check `moonraker.log`.

The device gets `locked_while_printing: True`, so Moonraker refuses to switch it off during a print — also from the Fluidd / Mainsail power menu.

### Existing Moonraker power device

If you already have a `[power …]` section in `moonraker.conf` (or in Moonraker's own config), choose option 5 and type its name. The installer lists the devices it finds.

### Last questions

| Question | Default | Meaning |
|---|---|---|
| Power off automatically after every print? | **Yes** | *Yes*: armed for every print. *No*: only when you run `ACTIVATE_POWER_OFF`. |
| Power off when nozzle and bed are below (°C) | **50** | Cool-down threshold, 30–100 °C. Both nozzle and bed must be below it. |

The installer writes `auto_power_off.cfg`, includes it in `printer.cfg` and restarts Klipper.

To change the plug, the threshold or the every-print default, simply run **option 19 again** and answer *Reconfigure*.

---

## Usage

| Macro | What it does |
|---|---|
| `ACTIVATE_POWER_OFF` | Power off after the running print, once cool. Run without a print, it powers off as soon as nozzle and bed are cool. |
| `DEACTIVATE_POWER_OFF` | Cancel a pending power off. During a print it also stops the every-print mode from re-arming it for **this** print. |
| `AUTO_POWER_OFF ENABLE=1` / `ENABLE=0` | Switch the every-print mode on / off **until Klipper restarts** (after a restart the installer's choice applies again). |

All three show up as buttons in the Fluidd / Mainsail macro panel. The console tells you what is going on:

```
Auto power off armed: the printer turns off after the print, once nozzle and bed are below 50 C.
Auto power off: waiting for nozzle and bed to cool down.
Auto power off: switching the printer off now.
```

### Only for some prints

Answer *No* to *every print* during installation and either click `ACTIVATE_POWER_OFF` while the print is running, or add it to a slicer profile you use for long overnight prints:

**Machine end G-code:**
```gcode
END_PRINT
ACTIVATE_POWER_OFF
```

---

## How It Works

Everything is in `/mnt/UDISK/printer_data/config/auto_power_off.cfg` (managed by the helper — re-run the installer instead of editing it):

1. **`_POWER_OFF_WATCH`** (every 30 s): if every-print mode is on and a print is running, it runs `ACTIVATE_POWER_OFF`.
2. **`ACTIVATE_POWER_OFF`** arms the power off and starts **`_POWER_OFF_WAIT_PRINT`**.
3. **`_POWER_OFF_WAIT_PRINT`** (every 30 s) waits while the print is *printing* or *paused*:
    - print *cancelled* or *error* → disarm, no power off;
    - print *complete* (or no print) → start the cool-down check.
4. **`_POWER_OFF_WAIT_COOL`** (every 10 s):
    - a new print started or a heater target set → disarm;
    - nozzle **and** bed below the threshold → **`_POWER_OFF_PRINTER`**.
5. **`_POWER_OFF_PRINTER`** switches the plug:
    - Moonraker device: `{action_call_remote_method("set_device_power", device="printer_plug", state="off")}`
    - KLAP Kasa / Tapo: `RUN_SHELL_COMMAND CMD=kasa_power_off` → `python3 /mnt/UDISK/helper-script/scripts/kasa.py off` (runs `sync` first so pending writes reach the disk).

The cool-down wait is what makes it safe: cutting the power also stops the hotend fan, and doing that while the hotend is still hot can cause heat creep and a clogged nozzle.

---

## Testing

!!! warning
    These commands switch the printer **off immediately**. Finish or cancel any print first.

- Quick test of the whole chain: with nozzle and bed cold and no print running, run `ACTIVATE_POWER_OFF` — the printer turns off within about a minute.
- KLAP Kasa / Tapo plug status, over SSH:

    ```bash
    python3 /mnt/UDISK/helper-script/scripts/kasa.py status
    ```

- Moonraker devices appear as `printer_plug` in the **power** menu of Fluidd / Mainsail, with their current state.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| No plug found by the network search | The plug must be in the same network (subnet) as the K2. Type the IP address instead. |
| `uses '…' encryption, which is not supported` | Update the plug's firmware in the Kasa / Tapo app. |
| Login fails (KLAP) | E-mail and password are case-sensitive; use the TP-Link account the plug is registered to. Tapo: enable *Third-Party Compatibility*. |
| `Moonraker cannot reach power device` | Check address and login, then `/mnt/UDISK/printer_data/logs/moonraker.log`. |
| Stopped working after a router restart | The plug got a new IP address — reserve one in the router and run option 19 again. |
| Printer did not turn off after a print | Check the console: a cancelled print, a heater left on (e.g. a preheat macro) or `DEACTIVATE_POWER_OFF` disarm it. |

---

## Removing

Remove menu (option 20) → **19) Auto Power Off**. This removes the include from `printer.cfg`, deletes `auto_power_off.cfg` and `.kasa.json`, removes the `[power printer_plug]` section the helper created (a power device you configured yourself stays) and restarts Moonraker and Klipper. Remove `ACTIVATE_POWER_OFF` from your slicer end G-code if you added it.

---

## Credits

- [tinntbg/auto-power-off-klipper](https://github.com/tinntbg/auto-power-off-klipper) — the macro approach
- [python-kasa/python-kasa](https://github.com/python-kasa/python-kasa) — reference for the KLAP protocol used by `kasa.py`
- [Moonraker power devices](https://moonraker.readthedocs.io/en/latest/configuration/#power) — Tasmota, Shelly, TP-Link and Home Assistant support
