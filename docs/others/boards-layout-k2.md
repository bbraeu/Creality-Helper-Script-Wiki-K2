# Boards Layout — K2

!!! warning "Derived from Creality's published config — not checked on the boards"
    The pin assignments below are taken from Creality's open-source K2 Klipper repository: [CrealityOfficial/K2_Series_Klipper](https://github.com/CrealityOfficial/K2_Series_Klipper), file `config/F021_CR0CN200400C10/printer.cfg` (bed `position_max` X 262, `mesh_max` 255,255 — the K2 base model; last updated in the repository on 2025-09-27). They have **not** been verified against the physical K2 boards, and your firmware version may differ. Board IDs and MCU chip types of the K2 boards are not documented here — check your own `printer.cfg` before relying on any pin.

The K2 uses two MCU boards — the main control board and a nozzle MCU board on the toolhead — plus the Linux host MCU.

---

## Main Control Board

**Serial:** `/dev/ttyS2`, baud 230400

| Function | Pin | Notes |
|---|---|---|
| Stepper X step / dir / enable | `PB8` / `!PB7` / `!PA9` | |
| Stepper Y step / dir / enable | `PB10` / `PB9` / `!PA8` | |
| Stepper Z step / dir / enable | `PB6` / `!PB5` / `!PA10` | single Z stepper; TMC2208 UART `PC1` |
| Endstop X | `PB11` | |
| Endstop Y | `PB12` | |
| Z endstop | `probe:z_virtual_endstop` | prtouch_v3 |
| Z alignment sensor | `PA15` | `[z_align]`, also `[gcode_button z1]` |
| Heated bed heater | `PC8` | |
| Heated bed thermistor | `PC4` | |
| Chamber thermistor | `PC5` | `temperature_sensor chamber_temp` and `temperature_fan chamber_fan` |
| Chamber fan | `PB3` | `temperature_fan chamber_fan` (also `output_pin fan1`) |
| Aux fan (fan2) | `PB4` | via `multi_pin heater_fan2` |
| Filter fan | `PB1` | `output_pin filter_fan` |
| Board cooling fan | `PA0` | `output_pin board_dissipation_fan` |
| LED strip | `PB0` + `PA12` | multi_pin, PWM |
| Power output | `PC9` | |
| PTC power | `PB2` | `output_pin ptc_power`, value 1 (no PTC heater is configured on the K2) |
| prtouch_v3 step swap | `!PC7` | |
| Filament rack sensor | `!PA5` | `[filament_rack]` in `box.cfg` |
| CFS RS-485 serial | `/dev/ttyS5` | baud 230400 (`box.cfg`) |

There is no chamber heater: `[heater_generic chamber_heater]` (pin `PC12` on the K2 Plus) and its PTC fan are commented out in the K2 config.

---

## Nozzle MCU Board

**Serial:** `/dev/ttyS3`, baud 230400

| Function | Pin | Notes |
|---|---|---|
| Extruder step / dir / enable | `nozzle_mcu:PB5` / `!nozzle_mcu:PB4` / `!nozzle_mcu:PB2` | |
| Hotend heater | `nozzle_mcu:PB8` | |
| Hotend thermistor | `nozzle_mcu:PA0` | custom thermistor |
| Hotend fan (heatsink) | `nozzle_mcu:PB7` | `heater_fan hotend_fan` |
| Part cooling fan enable | `nozzle_mcu:PB6` | `static_digital_output fan0_en` |
| Part cooling fan (fan0) | `!nozzle_mcu:PB15` | PWM |
| Extruder fan output | `nozzle_mcu:PB1` | |
| Filament sensor | `^!nozzle_mcu:PA11` | `filament_switch_sensor` |
| CFS cut switch | `!nozzle_mcu:PB9` | `[box] switch_pin` |
| LIS2DW CS / SCLK / MOSI / MISO | `nozzle_mcu:PA4` / `PA5` / `PA7` / `PA6` | SPI accelerometer |
| prtouch_v3 pressure CS | `nozzle_mcu:PB13`, `nozzle_mcu:PB14` | strain gauge |
| prtouch_v3 pres swap | `nozzle_mcu:PA15` | |

---

## Host MCU

```ini
[mcu rpi]
serial: /tmp/klipper_host_mcu
```

---

## Serial Port Map

| Device | Purpose |
|---|---|
| `/dev/ttyS2` | Main MCU |
| `/dev/ttyS3` | Nozzle MCU |
| `/dev/ttyS5` | CFS RS-485 bus |

---

!!! note "Duplicate pin override"
    The stock K2 `printer.cfg` requires a `[duplicate_pin_override]` section because several pins are shared between multiple Klipper objects (e.g. `PC5` is read by both `chamber_temp` and `chamber_fan`, `PB3` is used by both `chamber_fan` and `fan1`). Do not remove this section or Klipper will refuse to start.

```ini
[duplicate_pin_override]
pins: PC5,PA0,PC7,PB7,PB8,PB9,PB10,PB5,PB6,PA1,PB15,PB11,PB12,PB13,PA10,PA9,PB2,PB14,PB3
```
