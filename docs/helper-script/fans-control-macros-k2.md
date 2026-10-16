# Fans Control Macros — K2

The K2 family has a different fan topology from the K1 Series, and the models differ from each other (the K2 Pro and K2 Plus have a chamber heater with its own PTC fan, the K2 has not). The helper script therefore **generates** the fan macros from the fans your `printer.cfg` actually defines.

!!! note "K2 fan map differs from K1"
    The K1 Series has a part cooling fan and a nozzle cleaning fan. Do not apply K1 fan macro configs to the K2.

---

## Fan Hardware (K2 base)

The table below is taken from Creality's published K2 Klipper config ([CrealityOfficial/K2_Series_Klipper](https://github.com/CrealityOfficial/K2_Series_Klipper), `config/F021_CR0CN200400C10/printer.cfg`). Your firmware version may differ — check your own `printer.cfg`.

| Fan | Klipper name | Type | Notes |
|---|---|---|---|
| Hotend (heatsink) | `hotend_fan` | `heater_fan` | Auto — on when extruder > 50°C |
| Part cooling | `fan0` | `output_pin` PWM | Controlled by slicer `M106/M107`; gated by `static_digital_output fan0_en` |
| Aux fan | `fan2` | `output_pin` PWM | |
| Chamber fan | `chamber_fan` | `temperature_fan` | Watermark control, target 35°C, reads the chamber thermistor |
| Chamber temperature | `chamber_temp` | `temperature_sensor` | Same thermistor as `chamber_fan` |
| Filter fan, board fan | `filter_fan`, `board_dissipation_fan` | `output_pin` | |

There is **no chamber heater** (`heater_generic chamber_heater` is commented out in the K2 config) and no PTC heater fan.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 2 — Fans Control Macros**.

The macros are written to `/mnt/UDISK/printer_data/config/fans_control.cfg` (with the detected hardware in its header) and included from `printer.cfg`.

---

## Generated Macros

| Macro | Installed when | Purpose |
|---|---|---|
| `SET_FAN0 S=0-255` | always | Part cooling fan speed (also switches `fan0_en` if it is an `output_pin`; on the K2 it is a `static_digital_output` and always on). The value is converted using the pin's `scale` (255 on the K2), so `S=255` is full speed |
| `SET_FAN2 S=0-255` | `[output_pin fan2]` exists | Aux fan speed |
| `FANS_OFF` | always | Turns off the output fans (does not affect `hotend_fan` autocontrol) |
| `SAVE_FANS` / `RESTORE_FANS` | always | Save and restore the fan speeds |
| `CHAMBER_COOL` | a chamber heater or `temperature_fan` chamber fan exists | Sets the chamber fan target to 20°C (and turns the chamber heater off, if there is one) |
| `CHAMBER_STATUS` | a chamber sensor exists | Prints the current chamber temperature (and target, if heated) |
| `CHAMBER_HEAT` | a chamber heater exists — **not on the K2 base** | Sets the chamber heater target |

```gcode
SET_FAN0 S=255        ; full speed (255 = 100%)
SET_FAN0 S=128        ; 50%
SET_FAN0 S=0          ; off
SET_FAN2 S=255
CHAMBER_STATUS
```

The slicer's standard `M106 S[value]` also controls `fan0` through the stock macros.

!!! note "Chamber macros in two places"
    `CHAMBER_COOL` / `CHAMBER_STATUS` are written both by Fans Control Macros and by [Useful Macros](useful-macros-k2.md); they are identical, so installing both is fine.

### Chamber fan target

The chamber fan runs in `watermark` mode with a target of `35°C`. To change the cooling setpoint for the current session:

```gcode
SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET=40
```

For ABS/ASA you may want a higher target so the fan does not vent the chamber heat — see [Chamber Temperature](../improvements/chamber-temperature-k2.md).

---

## Notes on `duplicate_pin_override`

The stock K2 `printer.cfg` contains a `[duplicate_pin_override]` section because several pins are shared between Klipper objects (for example, the chamber thermistor is read by both `chamber_temp` and `chamber_fan`). Do not remove this section — Klipper will refuse to start if shared pins are not declared there.
