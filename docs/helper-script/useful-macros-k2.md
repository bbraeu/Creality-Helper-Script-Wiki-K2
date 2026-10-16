# Useful Macros — K2

The helper script installs a set of commonly used macros generated for the **detected** hardware: PID tuning, bed leveling, a movement warm-up and chamber macros. On the K2 base model there is no chamber heater, so no chamber heating macros are created.

!!! note "Stock print macros are not replaced"
    `START_PRINT`, `END_PRINT`, `PAUSE`, `RESUME` and `CANCEL_PRINT` are **not** part of Useful Macros. They are already defined in the stock `gcode_macro.cfg` with full CFS integration; overwriting them broke CFS filament loading and unloading in earlier versions.

!!! note "K1 macros are not compatible"
    Do not copy K1 Series macros to the K2. Fan pin names, chamber references, and bed dimensions all differ.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 3 — Useful Macros (PID, leveling, warmup)**.

Macros are written to `/mnt/UDISK/printer_data/config/useful_macros.cfg` and included from `printer.cfg`. The file header shows the detected printer, e.g. `# Useful Macros - K2 (no chamber heater, bed mesh 20,20 to 255,255)`. After installing, the script prints the list of macros it generated.

---

## Macros

| Macro | Installed when | Description |
|---|---|---|
| `PID_HOTEND TEMP=220` | always | `PID_CALIBRATE` for the hotend, then `SAVE_CONFIG` |
| `PID_BED TEMP=60` | always | `PID_CALIBRATE` for the bed, then `SAVE_CONFIG` |
| `PID_CHAMBER TEMP=45` | chamber heater exists — **not on the K2 base** | Heats the chamber and waits |
| `BED_LEVELING BED_TEMP=60 EXTRUDER_TEMP=150` | always | Home, heat, (`Z_TILT_ADJUST` if `[z_tilt]` exists), full `BED_MESH_CALIBRATE PROFILE=default`, `SAVE_CONFIG` |
| `Z_TILT_CALIBRATE` | `[z_tilt]` exists | Home if needed, run `Z_TILT_ADJUST` |
| `WARMUP LOOPS=10 ACCEL=5000` | always | Movement warm-up inside the bed mesh area |
| `CHAMBER_STATUS` | a chamber sensor exists | Prints the chamber temperature (and target, if heated) |
| `CHAMBER_COOL` | a chamber fan or heater exists | Sets the chamber fan target to 20°C (heater off, if any) |
| `CHAMBER_HEAT TARGET=45 WAIT=1` | chamber heater exists — **not on the K2 base** | Sets the chamber heater target |

!!! warning "PID macros save the config"
    `PID_HOTEND`, `PID_BED` and `BED_LEVELING` end with `SAVE_CONFIG`, which restarts Klipper. Do not run them during a print.

---

## WARMUP

Moves the toolhead in a rectangle-and-diagonals pattern to warm up belts and bearings before a print or calibration:

```gcode
WARMUP LOOPS=10 ACCEL=5000
```

| Parameter | Default | Description |
|---|---|---|
| `LOOPS` | 10 | Number of pattern repetitions |
| `ACCEL` | 5000 | Acceleration during the warm-up (restored afterwards) |

The corners are the **`mesh_min` / `mesh_max` of `[bed_mesh]`** (20,20 to 255,255 on a stock K2 after the helper's `mesh_min` fix). The axis travel limits are not used: on the K2 they reach past the bed (X −12…262, Y up to 296.5). Earlier versions drove to X/Y 345, which is out of range on the 260 mm K2. The installer prints the X/Y range it used; the exact numbers depend on your `printer.cfg`.

---

## Z Tilt

Creality's published K2 config (`F021`) has a single `[stepper_z]` with `[z_align]` and the `[z_tilt]` section commented out. In that case `BED_LEVELING` skips `Z_TILT_ADJUST` and `Z_TILT_CALIBRATE` is not installed. Printers whose `printer.cfg` does define `[z_tilt]` (e.g. the K2 Plus) get both.

---

## Chamber Macros on the K2

The K2 has a chamber temperature sensor and a `temperature_fan chamber_fan`, but no chamber heater. Expect:

- `CHAMBER_STATUS` — prints `Chamber: 31.5 C` (reads `temperature_sensor chamber_temp`)
- `CHAMBER_COOL` — lowers the chamber fan target so it vents the chamber
- no `CHAMBER_HEAT`, no `PID_CHAMBER`

See [Chamber Temperature](../improvements/chamber-temperature-k2.md) for how to get a warm chamber without a heater.
