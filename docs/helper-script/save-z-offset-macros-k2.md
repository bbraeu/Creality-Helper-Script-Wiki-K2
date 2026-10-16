# Save Z-Offset Macros — K2

The K2 uses a `prtouch_v3` strain-gauge probe for bed leveling. Z-offset is stored in the `SAVE_CONFIG` block of `printer.cfg` at `/mnt/UDISK/printer_data/config/printer.cfg`.

!!! note "Path difference from K1"
    On K1, config lives under `/usr/data/printer_data/config/`. On K2 it is `/mnt/UDISK/printer_data/config/`.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 4 — Save Z-Offset Macros**. The macros are written to `/mnt/UDISK/printer_data/config/z_offset_macros.cfg`.

---

## Usage

Adjust Z-offset live during a print using babystepping, then save it permanently:

```gcode
SAVE_Z_OFFSET
```

This writes the current live Z-offset to the `[prtouch_v3]` section in `printer.cfg` via `SAVE_CONFIG`.

`SAVE_Z_OFFSET` runs `Z_OFFSET_APPLY_PROBE` followed by `SAVE_CONFIG` (which restarts Klipper — do not run it mid-print if you want the print to continue).

Other macros:

| Macro | Description |
|---|---|
| `SET_Z_OFFSET Z=0.05` | Applies a Z offset (`SET_GCODE_OFFSET ... MOVE=1`); run `SAVE_Z_OFFSET` to persist it |
| `RESET_Z_OFFSET` | Resets the live Z offset to 0 |

---

## Notes on prtouch_v3

The K2 probe uses temperature compensation (`enable_not_linear_comp: True`) and performs automatic re-probing when the bed or nozzle temperature changes significantly. This means Z-offset can shift slightly between cold and hot starts — the temperature compensation accounts for this. If you notice first-layer variation between cold and warm starts, let the printer reach full print temperature before running `BED_MESH_CALIBRATE`.
