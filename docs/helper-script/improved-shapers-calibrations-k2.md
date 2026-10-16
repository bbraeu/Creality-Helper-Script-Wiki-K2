# Improved Shapers Calibrations — K2

The K2 uses a **LIS2DW** accelerometer connected via SPI to the nozzle MCU. This is different from the K1 Series which uses an ADXL345. The helper script adds a set of input-shaper and belt macros on top of Klipper's standard resonance testing.

---

## Hardware Details

From Creality's published K2 config (`F021_CR0CN200400C10/printer.cfg` in [CrealityOfficial/K2_Series_Klipper](https://github.com/CrealityOfficial/K2_Series_Klipper)) — check your own `printer.cfg`:

| Item | Value |
|---|---|
| Accelerometer chip | LIS2DW |
| Interface | SPI (software) |
| CS pin | `nozzle_mcu:PA4` |
| SCLK | `nozzle_mcu:PA5` |
| MOSI | `nozzle_mcu:PA7` |
| MISO | `nozzle_mcu:PA6` |
| Axes map | `x, z, y` |
| SPI speed | 5 MHz |

```ini
[lis2dw]
cs_pin: nozzle_mcu:PA4
spi_speed: 5000000
axes_map: x,z,y
spi_software_sclk_pin: nozzle_mcu:PA5
spi_software_mosi_pin: nozzle_mcu:PA7
spi_software_miso_pin: nozzle_mcu:PA6

[resonance_tester]
accel_chip: lis2dw
probe_points:
   130,130,130
min_freq: 20
max_freq: 120
accel_per_hz: 100
```

Your calibrated input shaper values are stored in the `SAVE_CONFIG` block of `printer.cfg` (`shaper_type_x`, `shaper_freq_x`, `shaper_type_y`, `shaper_freq_y`).

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 7 — Improved Shapers Calibrations**. The macros are written to `/mnt/UDISK/printer_data/config/shapers_calibration.cfg`.

---

## Macros

| Macro | Description |
|---|---|
| `INPUT_SHAPER_CALIBRATION` | Homes if needed and runs `SHAPER_CALIBRATE` for X and Y (recommended) |
| `SHAPER_CALIBRATE_X` / `SHAPER_CALIBRATE_Y` | One axis only |
| `BELTS_SHAPER_CALIBRATION` | Low-frequency sweep on X and Y to compare belt frequency profiles (raw CSV) |
| `EXCITATE_AXIS_AT_FREQ AXIS=X FREQ=25 TIME=10` | Holds a frequency to locate rattles and loose parts |
| `AUTOTUNE_SHAPERS` | Saves the results of the last calibration (`SAVE_CONFIG`) |
| `TEST_RESONANCES_GRAPHS` | Raw CSV data for both axes, for analysis with Klipper's `calibrate_shaper.py` |
| `RESTORE_SHAPERS` | Re-applies the shaper settings saved in `printer.cfg` |
| `SHAPER_STATUS` | Shows the saved shaper settings |

!!! warning "Home the printer first"
    The resonance tester moves to `probe_points` from `[resonance_tester]` in your `printer.cfg`. The macros home automatically if needed; make sure the bed is clear.

---

## Applying Results

After `INPUT_SHAPER_CALIBRATION` completes, run:

```gcode
AUTOTUNE_SHAPERS
```

(or `SAVE_CONFIG`). This writes the recommended shaper type and frequency to the `SAVE_CONFIG` block in `printer.cfg`. Klipper restarts automatically to apply the changes.

---

## Re-running After Hardware Changes

Re-run the calibration after:

- Changing the nozzle or hotend assembly
- Tightening or replacing belts
- Any significant modification to the toolhead mass

The K2's CoreXY kinematics means X and Y shaper values are independent.

---

## Belt Tension Check

The stock config includes `[belt_mdl]` modules for belt tension measurement. Check belt tension via the touchscreen (**Settings → Self-check → Belt Tension**). The target tension for your model is defined in the stock firmware — the helper macros no longer assume the K2 Plus value. Re-run shaper calibration after any belt adjustment.
