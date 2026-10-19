# Creality Print — K2

Creality Print is Creality's own slicer (based on OrcaSlicer / Bambu Studio). It ships the official **Creality K2** profile, connects to the printer in the **Device** tab and sends prints directly. It works fine with the helper script; with [KAMP-K2](../helper-script/klipper-adaptive-meshing-and-purging-k2.md) only the start G-code has to change.

Download: :material-github: [GitHub Releases](https://github.com/CrealityOfficial/CrealityPrint/releases/latest) (the G-code on this page is from v7.3.0)

---

## Printer Profile

Add the printer **Creality K2** with your nozzle size (0.2 / 0.4 / 0.6 / 0.8 mm). Not *K2 Plus* (350 mm), *K2 Pro* (300 mm) or *K2 SE* — those have a different bed size.

The profile already has everything the K2 needs: Klipper G-code flavor, relative extrusion, build volume 260 × 260 × 260 mm, no chamber temperature control.

!!! tip "Edit a copy, not the system preset"
    When you change the G-code below, Creality Print asks to save the printer preset — save it as a **new user preset** (e.g. *Creality K2 0.4 nozzle - KAMP*). The system preset keeps Creality's original and is updated with Creality Print; switching back is just selecting it again.

The machine G-codes are in the printer settings (edit icon next to the printer) → **Machine G-code**.

---

## Start G-code — Creality Original

The start G-code of the **Creality K2 0.4 nozzle** system preset has two branches. Without KAMP-K2, keep it as it is:

- `{if multicolor_method}` — **with the CFS** (K2 Combo): cut, unload the old filament, load the first slot at the filament's maximum temperature, flush and wipe three times at the purge position behind the bed, then the purge line.
- `{else}` — **without the CFS**: load the filament, then the purge line.

Both branches end with the same fixed purge line along the **front edge** of the bed (Y −1, X 105 → 155) and a short wipe.

??? example "Creality's original start G-code (v7.3.0, 181 lines)"

    ```gcode
    M140 S0
    M104 S0
    START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
    {if multicolor_method}

    M83
    M8200 P S[initial_no_support_extruder]
    M220 S100
    G0 Y150 F12000
    G0 X10
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=25000
    G0 F30000

    M8200 C S0
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 Y261 F18000
    G0 X131
    G0 Y291.5
    G0 X115
    M8200 R
    M104 S[nozzle_temperature_range_high[initial_no_support_extruder]]
    M8200 L I[initial_no_support_extruder]
    M106 S0
    M106 P2 S0
    T[initial_no_support_extruder]

    ; FLUSH_START
    M106 S30
    G1 F60
    M400
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60

    M400
    M106 S255
    G4 P5000
    M106 S30
    G1 E-[retract_length_toolchange[initial_no_support_extruder]] F1800
    ; FLUSH_END

    ; WIPE
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 X137 F12000
    G0 X127

    G0 X145 Y291.5 F12000
    G2 I4 J0 P1 F10000
    G0 X155 Y291.5 F12000
    G3 I-4 J0 P1 F10000

    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000

    ; FLUSH_START
    G1 F60
    M400
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60

    M400
    M106 S255
    G4 P5000
    M106 S30
    G1 E-[retract_length_toolchange[initial_no_support_extruder]] F1800
    ; FLUSH_END

    ; WIPE
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 X137 F12000
    G0 X127

    G0 X145 Y291.5 F12000
    G2 I4 J0 P1 F10000
    G0 X155 Y291.5 F12000
    G3 I-4 J0 P1 F10000

    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000

    ; FLUSH_START
    G1 F60
    M400
    M104 S[nozzle_temperature_initial_layer]
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60

    M400
    M106 S255
    G4 P5000
    M106 S0
    G1 E-[retract_length_toolchange[initial_no_support_extruder]] F1800
    ; FLUSH_END

    ; WIPE
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 X137 F12000
    G0 X127

    G0 X145 Y291.5 F12000
    G2 I4 J0 P1 F10000
    G0 X155 Y291.5 F12000
    G3 I-4 J0 P1 F10000

    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000

    M8200 O
    M204 S2000
    G1 Z3 F600
    G1 X137 F12000
    G1 X105 Y-1 F30000
    G1 E[retraction_length]  F300
    G1 Z0.3 F1200
    G1 X130 E9  F{outer_wall_volumetric_speed/(24/20)  * 60}
    G1 X135 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)/4*60}
    G1 X140 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)*60}
    G1 X145 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)/4*60}
    G1 X150 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)*60}
    G1 X155 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)/4*60}
    G91
    G1 X1 Z-0.3
    G1 X4
    G92 E0

    {else}
    T[initial_no_support_extruder]
    M104 S[nozzle_temperature_initial_layer]
    M204 S2000
    G1 Z3 F600
    M83
    G1 X105 Y-1 F30000
    G1 E[retraction_length]  F300
    G1 Z0.3 F1200
    G1 X130 E9  F{outer_wall_volumetric_speed/(24/20)  * 60}
    G1 X135 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)/4*60}
    G1 X140 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)*60}
    G1 X145 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)/4*60}
    G1 X150 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)*60}
    G1 X155 E.3742  F{outer_wall_volumetric_speed/(0.3*0.5)/4*60}
    G91
    G1 X1 Z-0.3
    G1 X4
    G92 E0
    G1 Z1 F600
    {endif}
    ```

Line by line: [KAMP-K2 → Creality's original K2 start G-code](../helper-script/klipper-adaptive-meshing-and-purging-k2.md#crealitys-original-k2-start-g-code).

---

## Start G-code with KAMP-K2

If you installed [KAMP-K2](../helper-script/klipper-adaptive-meshing-and-purging-k2.md) (option 6), replace the start G-code with Creality's original, with the fixed purge line in **both branches** swapped for KAMP's `LINE_PURGE`. The CFS load, flush and wipe stay exactly as Creality ships them.

Only the end of each branch changes. CFS branch, from `M8200 O` on:

```gcode
M8200 O
M204 S2000
G1 Z3 F600
G1 X137 F12000
M109 S[nozzle_temperature_initial_layer]
LINE_PURGE
```

Branch without CFS, from `{else}` on:

```gcode
{else}
T[initial_no_support_extruder]
M109 S[nozzle_temperature_initial_layer]
M204 S2000
G1 Z3 F600
M83
LINE_PURGE
{endif}
```

??? example "Complete start G-code with KAMP-K2 — paste this"

    ```gcode
    M140 S0
    M104 S0
    START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
    {if multicolor_method}

    M83
    M8200 P S[initial_no_support_extruder]
    M220 S100
    G0 Y150 F12000
    G0 X10
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=25000
    G0 F30000

    M8200 C S0
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 Y261 F18000
    G0 X131
    G0 Y291.5
    G0 X115
    M8200 R
    M104 S[nozzle_temperature_range_high[initial_no_support_extruder]]
    M8200 L I[initial_no_support_extruder]
    M106 S0
    M106 P2 S0
    T[initial_no_support_extruder]

    ; FLUSH_START
    M106 S30
    G1 F60
    M400
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60

    M400
    M106 S255
    G4 P5000
    M106 S30
    G1 E-[retract_length_toolchange[initial_no_support_extruder]] F1800
    ; FLUSH_END

    ; WIPE
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 X137 F12000
    G0 X127

    G0 X145 Y291.5 F12000
    G2 I4 J0 P1 F10000
    G0 X155 Y291.5 F12000
    G3 I-4 J0 P1 F10000

    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000

    ; FLUSH_START
    G1 F60
    M400
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60

    M400
    M106 S255
    G4 P5000
    M106 S30
    G1 E-[retract_length_toolchange[initial_no_support_extruder]] F1800
    ; FLUSH_END

    ; WIPE
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 X137 F12000
    G0 X127

    G0 X145 Y291.5 F12000
    G2 I4 J0 P1 F10000
    G0 X155 Y291.5 F12000
    G3 I-4 J0 P1 F10000

    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000

    ; FLUSH_START
    G1 F60
    M400
    M104 S[nozzle_temperature_initial_layer]
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60
    G1 E{90 * 0.18} F{filament_max_volumetric_speed[initial_no_support_extruder]/2.4053*60}
    G1 E{90 * 0.02} F60

    M400
    M106 S255
    G4 P5000
    M106 S0
    G1 E-[retract_length_toolchange[initial_no_support_extruder]] F1800
    ; FLUSH_END

    ; WIPE
    SET_VELOCITY_LIMIT ACCEL=5000 ACCEL_TO_DECEL=5000
    G0 X137 F12000
    G0 X127

    G0 X145 Y291.5 F12000
    G2 I4 J0 P1 F10000
    G0 X155 Y291.5 F12000
    G3 I-4 J0 P1 F10000

    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000
    G0 X137 Y291.5 F12000
    G0 X115 Y291.5 F12000

    M8200 O
    M204 S2000
    G1 Z3 F600
    G1 X137 F12000
    M109 S[nozzle_temperature_initial_layer]
    LINE_PURGE

    {else}
    T[initial_no_support_extruder]
    M109 S[nozzle_temperature_initial_layer]
    M204 S2000
    G1 Z3 F600
    M83
    LINE_PURGE
    {endif}
    ```

- **CFS branch:** `M109` added before `LINE_PURGE`. Creality's third flush only *sets* the print temperature (`M104`) — coming down from the filament's maximum temperature, the nozzle may still be hotter than the first layer temperature. `M109` waits for it.
- **Without CFS:** `M104` became `M109` (wait for print temperature before purging).
- **Both branches:** the fixed purge line (`G1 X105 Y-1 …` to `G1 X4`) became `LINE_PURGE`, and the trailing `G92 E0` (and `G1 Z1 F600`) are removed so KAMP's post-purge Z protection stays in place — [why](../helper-script/klipper-adaptive-meshing-and-purging-k2.md#start-g-code-with-kamp-k2). The purge line's `G91` goes with it, so nothing switches to relative positioning any more.
- The adaptive mesh itself runs inside `START_PRINT` — the installer put it there. Nothing else to add.
- Add `MESH=0` to the `START_PRINT` line to skip the mesh for one print.
- Keep `START_PRINT EXTRUDER_TEMP=… BED_TEMP=…` on **one line**, and keep `{if multicolor_method}` / `{else}` / `{endif}` — without them Creality Print would run the CFS sequence on every print.

!!! note "Placeholders"
    KAMP-K2's own README says Creality Print's placeholder syntax is incompatible with its OrcaSlicer block. That does not apply here: this block only uses the placeholders from Creality Print's own K2 profile (`multicolor_method`, `[nozzle_temperature_initial_layer]`, `[initial_no_support_extruder]`, `filament_max_volumetric_speed`, …).

### Exclude objects

KAMP needs the object outlines (`EXCLUDE_OBJECT_DEFINE`) in the G-code. In the process settings, switch to **Advanced** and check **Others → G-code output → Exclude objects**. Creality's K2 process presets have it on; check it if you use your own preset.

Creality Print sends files to the K2 through Creality's own service, not through Moonraker, so Moonraker cannot add the object outlines on upload — the slicer has to write them.

### Sending the print

Slice, then **Print** / send to the K2 as usual. In the console of Fluidd / Mainsail you should see `BED_MESH_CALIBRATE_START_PRINT deferred (KAMP adaptive in START_PRINT)`, the adapted mesh bounds and `KAMP purge starting at …` — [full list](../helper-script/klipper-adaptive-meshing-and-purging-k2.md#what-happens-at-print-start).

!!! warning "Not yet verified with Creality Print on a K2 base"
    KAMP-K2's author rates Creality Print as *partial*: with Creality's default start G-code, a full bed mesh ran in addition to the adaptive one. KAMP-K2 turns the `G29` / `BED_MESH_CALIBRATE_START_PRINT` that Creality's firmware runs before a print into no-ops, so with the block above only the adaptive mesh should run. If you still see a full-bed mesh before the adaptive one, untick **Print Calibration** in the send dialog. Reports are welcome in the [Discussions](https://github.com/bbraeu/Creality-Helper-Script-Wiki-K2-Base/discussions).

---

## End G-code

Creality's original — keep it:

```gcode
{if print_sequence == "by object"}
G91
G1 X2 Y2 Z1 F24000
G90
G1 Z{max_layer_z+2} F600
{endif}
END_PRINT
```

With [Auto Power Off](../helper-script/auto-power-off-k2.md) installed but not armed for every print, add `ACTIVATE_POWER_OFF` after `END_PRINT` in a preset you use for long or overnight prints.

---

## Removing KAMP-K2

After removing KAMP-K2 (Remove menu → 6), select the **system** Creality K2 preset again (or paste the original start G-code back). Without KAMP-K2 there is no `LINE_PURGE` macro — prints with the KAMP block would start without a purge line and Klipper reports `Unknown command: "LINE_PURGE"`.
