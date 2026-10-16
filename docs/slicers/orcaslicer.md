# OrcaSlicer — K2

OrcaSlicer is the recommended slicer for the K2 — in particular with [KAMP-K2](../helper-script/klipper-adaptive-meshing-and-purging-k2.md), because Creality Print still triggers its own full bed mesh.

Download: :material-github: [GitHub Releases](https://github.com/SoftFever/OrcaSlicer/releases/latest)

---

## Printer Profile

Select the Creality K2 profile that matches your printer (**Printer → Creality**). If your OrcaSlicer version does not list the K2 base model, create a custom printer from another K2-family profile and set the values below.

!!! warning "Unverified profile name"
    The exact name of the K2 base profile in OrcaSlicer has not been verified for this wiki. Make sure the build volume is **260 × 260 × 260 mm** — a K2 Plus profile (350 mm) will send moves outside the K2's travel range.

| Setting | K2 value |
|---|---|
| Build volume | 260 × 260 × 260 mm |
| Kinematics | CoreXY |
| Max speed | 600 mm/s |
| Max acceleration | 20,000 mm/s² |
| Max nozzle / bed temperature | 300 °C / 100 °C |
| Chamber heater | **None** — leave chamber temperature at 0 / off |

### Adaptive bed mesh limits

**Printer Settings → Basic information → Adaptive bed mesh**: set **Bed mesh min / max** to the `mesh_min` / `mesh_max` values of `[bed_mesh]` in your `printer.cfg`. On the K2 the maximum is typically **255, 255**. (The helper script changes a stock `mesh_min: 5,5` to `20,20`.)

---

## Machine G-codes

In OrcaSlicer, edit your printer preset and go to the **Machine G-code** tab.

### Machine start G-code — with KAMP-K2 (recommended)

If you installed [KAMP-K2](../helper-script/klipper-adaptive-meshing-and-purging-k2.md) (option 6), use:

```gcode
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
T[initial_no_support_extruder]
LINE_PURGE
M204 S2000
M83
```

- Also enable **Print Settings → Others → Output options → Label objects** — KAMP needs the object outlines.
- `LINE_PURGE` **must come after** `T[initial_no_support_extruder]`: on the K2 Combo, the CFS loads the filament on `T<n>`; a purge before that runs with an empty nozzle.
- Add `MESH=0` to the `START_PRINT` line to skip the bed mesh for one print.

### Machine start G-code — without KAMP-K2

```gcode
SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count]
M140 S0
M104 S0
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
```

!!! warning "One command per line — never split a command"
    Both blocks above have **several lines, one complete command per line**. What must not happen is a single command being split over several lines: if your template has line breaks between `START_PRINT` and its parameters (for example, pasted from an older guide or edited by hand), the sliced file contains `BED_TEMP=60` and `EXTRUDER_TEMP=220` as separate lines. Klipper then sees them as unrecognized standalone commands and logs `Unknown command` (key61) errors for each one. Keep `START_PRINT EXTRUDER_TEMP=... BED_TEMP=...` together on one line and re-slice.

!!! note "No CHAMBER_TEMP on the K2"
    K2 Plus profiles pass `CHAMBER_TEMP=[chamber_temperature]` to `START_PRINT`. The K2 has no chamber heater, so it is left out here. See [Chamber Temperature](../improvements/chamber-temperature-k2.md).

**Machine end G-code:**
```gcode
END_PRINT
```

**Before layer change G-code:**
```gcode
;BEFORE_LAYER_CHANGE
;[layer_z]
G92 E0
```

**Layer change G-code:**
```gcode
SET_PRINT_STATS_INFO CURRENT_LAYER={layer_num + 1}
;AFTER_LAYER_CHANGE
;[layer_z]
```

**Time lapse G-code** (if Moonraker Timelapse is installed):
```gcode
TIMELAPSE_TAKE_FRAME
```

**Change filament G-code** (if M600 Support is installed):
```gcode
M600
```

---

## Multi-Material Setup (K2 Combo with CFS)

For the K2 Combo, configure OrcaSlicer as a multi-material printer:

- Set the number of extruders to match your loaded CFS slots (up to 16 with four CFS units)
- The slicer's `T0`–`T15` tool change commands are translated to CFS slot addresses by the stock `box.cfg` macros
- Set flush/purge volume to match your `box.cfg` settings
- Purging during tool changes happens at the CFS purge chute (`extrude_pos_x/y` in `box.cfg`) — not as a line on the bed

See [CFS — Color Filament System](../helper-script/cfs-k2.md) for the full tool-change sequence.

---

## Upload G-code Files to Printer

- Click the **Connection** icon in OrcaSlicer
- Enter your printer's IP address:
    - For **Fluidd**: `http://<printer-ip>:4408/`
    - For **Mainsail**: `http://<printer-ip>:4409/`
- Click **Connect** — you can now upload and start prints directly from the slicer
