# Chamber Temperature — K2

The K2 (base model) has an enclosed chamber, but **no chamber heater**. The chamber only gets warm from the heated bed, the hotend and the closed lid. The K2 Pro (up to 60 °C) and the K2 Plus have an active chamber heater — chamber heating macros and `CHAMBER_TEMP` slicer settings written for those models do not apply here.

---

## What the K2 Has

From Creality's published K2 config ([CrealityOfficial/K2_Series_Klipper](https://github.com/CrealityOfficial/K2_Series_Klipper), `config/F021_CR0CN200400C10/printer.cfg`) — check your own `printer.cfg`:

| Component | Klipper name | Type | Notes |
|---|---|---|---|
| Chamber temperature sensor | `chamber_temp` | `temperature_sensor` | EPCOS 100K thermistor |
| Chamber fan | `chamber_fan` | `temperature_fan` | Same thermistor; watermark control, target 35 °C |
| Chamber heater | — | — | `[heater_generic chamber_heater]` is commented out |

!!! note "Sensor accuracy"
    A forum report says the K2 chamber thermistor reads a few °C lower than the actual chamber air. Treat the value as a trend indicator rather than an exact measurement.

### Helper script macros

With [Useful Macros](../helper-script/useful-macros-k2.md) or [Fans Control Macros](../helper-script/fans-control-macros-k2.md) installed:

| Macro | On the K2 |
|---|---|
| `CHAMBER_STATUS` | Installed — prints the current chamber temperature from the existing sensor |
| `CHAMBER_COOL` | Installed when a chamber fan exists — sets the chamber fan target to 20 °C so it vents |
| `CHAMBER_HEAT`, `PID_CHAMBER` | **Not installed** — there is no heater |

In Fluidd / Mainsail the chamber temperature appears in the temperature panel as `chamber_temp` / `chamber_fan`.

---

## Passive Heat-Soak for ABS / ASA

Without a heater, you can still get a warm, stable chamber for materials that warp or delaminate:

1. **Close the lid** (and keep the chamber closed for the whole print).
2. **Heat the bed** to the print temperature (up to 100 °C) and let it soak — often 10–20 minutes. Run `CHAMBER_STATUS` (or watch the temperature graph) until the chamber temperature stops rising.
3. **Keep the chamber fan from venting the heat**: the `chamber_fan` runs in watermark mode with a 35 °C target, so it starts cooling once the chamber passes 35 °C. For ABS/ASA you can raise the target for the session:

    ```gcode
    SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET=50
    ```

    Set it back to 35 (or run `CHAMBER_COOL` after the print) for PLA/PETG, which need a cool chamber.
4. **Reduce part cooling** for ABS/ASA in the filament profile, and avoid drafts around the printer.
5. **Let the part cool slowly** in the closed printer after the print.

!!! warning "PLA and PETG"
    PLA (and to a lesser degree PETG) does not like a warm chamber — it can soften, clog or show heat creep. For these materials, open the lid or keep the chamber fan target low.

!!! tip "Pre-heat before starting"
    `START_PRINT` does not wait for a chamber temperature on the K2. A simple approach is to pre-heat the bed manually (e.g. `M140 S100` in the Fluidd console, lid closed) and start the print once the chamber temperature has settled.

---

## Chamber Heater Retrofits

Third-party chamber heater retrofits for the K2 exist. They are **out of scope** for this wiki and the helper script: the helper only creates `CHAMBER_HEAT` / `PID_CHAMBER` if your `printer.cfg` defines a `[heater_generic ...chamber...]` section, and it makes no assumptions about such hardware. Wiring and safety are your responsibility.
