# KAMP-K2 — Klipper Adaptive Meshing & Purging — K2

KAMP (Klipper Adaptive Meshing & Purging) probes the bed only where the current print is, and draws a purge line next to it. Option 6 of the helper script installs **[KAMP-K2](https://github.com/grant0013/KAMP-K2)** by grant0013 (v1.1.0, GPL-3.0), a K2 port of [kyleisah's KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging).

---

## Why Plain KAMP Does Not Work on the K2

Upstream KAMP never produced an adaptive mesh on the K2 family:

- Creality's **`prtouch_v3`** probe module replaces Klipper's `BED_MESH_CALIBRATE` with its own handler that **ignores `MESH_MIN` / `MESH_MAX`** — it always probes the full bed.
- The **`master-server`** daemon runs `G29` / `BED_MESH_CALIBRATE_START_PRINT` **before every print** and waits for the line `[G29_TIME]Execution time:` before it continues — a full mesh every time, regardless of KAMP.

KAMP-K2 fixes both:

1. `restore_bed_mesh.py` (a Klipper extra) re-registers Klipper's upstream `BED_MESH_CALIBRATE`, so the adaptive bounds are respected.
2. `G29` and `BED_MESH_CALIBRATE_START_PRINT` are replaced with no-ops that clear the mesh and print the `[G29_TIME]` line `master-server` waits for.
3. The real (adaptive) mesh runs inside `START_PRINT`, where the object outlines of the loaded G-code are known.

---

## Installation

Make sure you have followed the [Install Helper Script](helper-script-installation.md) section, then from the `[Install] Menu` select **option 6 — Klipper Adaptive Meshing & Purging (KAMP-K2)**. Moonraker Extensions are installed first if missing.

The installer:

1. Backs up `printer.cfg`, `gcode_macro.cfg` (and an existing `KAMP/` folder) to `/mnt/UDISK/printer_data/config/backups/kamp_k2_<timestamp>/`.
2. Downloads KAMP-K2 v1.1.0 and copies `KAMP_Settings.cfg`, `Adaptive_Meshing.cfg` and `Line_Purge.cfg` to `/mnt/UDISK/printer_data/config/KAMP/` (Smart Park and Voron Purge are not used — the stock CFS macros already park).
3. Copies `restore_bed_mesh.py` to `/usr/share/klipper/klippy/extras/`. A firmware update replaces that folder, so the helper re-applies the file **on every boot** (and every time the helper script starts).
4. Patches `printer.cfg` — before the `SAVE_CONFIG` block it adds `[exclude_object]` (if missing), `[restore_bed_mesh]` and `[include KAMP/KAMP_Settings.cfg]`. (On a K2 Plus it also sets `forced_leveling: false`.)
5. Patches `gcode_macro.cfg` — saves the original as `gcode_macro.cfg.pre_kamp`, replaces `G29` and `BED_MESH_CALIBRATE_START_PRINT` with the no-ops described above, and inserts this block into `START_PRINT`, right after the `{% endif %}` that closes the stock *prepare* branch (heat, `G28`, `NOZZLE_CLEAR`, `G28 Z`). Because `BED_MESH_CALIBRATE_START_PRINT` no longer marks the print as prepared, `START_PRINT` always runs that full branch first, so the mesh runs homed and at temperature. If your `START_PRINT` does not have the stock layout, the installer prints a warning and does **not** insert the block (guessing a spot could mesh before homing) — add `BED_MESH_CLEAR` + `BED_MESH_CALIBRATE` after homing yourself:

    ```
    {% if params.MESH|default(1)|int == 1 %}
      BED_MESH_CLEAR
      BED_MESH_CALIBRATE
    {% else %}
      M118 Mesh skipped (MESH=0 from slicer)
    {% endif %}
    ```

    An old `LINE_PURGE` inside `START_PRINT` (from earlier KAMP installers) is removed — on CFS printers it fired before `T<n>` had loaded filament.
6. Enables `[file_manager] enable_object_processing: True` in the UDISK `moonraker.conf` (KAMP needs the object outlines) and restarts Moonraker and Klipper.

The patches are idempotent and are re-checked at every boot, so they come back after a firmware update rewrites the stock files.

### Verify

After Klipper has started, `klippy.log` must contain:

```
bed_mesh_override: re-registered
```

```bash
grep "bed_mesh_override: re-registered" /mnt/UDISK/printer_data/logs/klippy.log
```

The installer checks this itself and prints either *"KAMP installed - bed mesh override active."* or a hint to check the log.

---

## Start G-code

KAMP-K2 needs one change in the slicer: the fixed purge line in Creality's start G-code is replaced with KAMP's `LINE_PURGE`. Everything else stays as Creality ships it. Creality Print and OrcaSlicer ship **different** K2 start G-codes, so take the block from the page of your slicer:

- [Creality Print → Start G-code with KAMP-K2](../slicers/creality-print.md#start-g-code-with-kamp-k2) — two branches (with / without CFS), both get `LINE_PURGE`
- [OrcaSlicer → Machine start G-code with KAMP-K2](../slicers/orcaslicer.md#machine-start-g-code-with-kamp-k2) — one block, shown below

### Creality's original K2 start G-code

**Creality Print** (v7.3.0, **Creality K2 0.4 nozzle** system preset) — the complete 181 lines are on the [Creality Print](../slicers/creality-print.md#start-g-code-creality-original) page. What they do:

| Lines | What they do |
|---|---|
| `M140 S0`, `M104 S0` | Clear any heater targets — `START_PRINT` does the heating. |
| `START_PRINT EXTRUDER_TEMP=… BED_TEMP=…` | Heat, home, clean the nozzle (stock `gcode_macro.cfg`). Must stay on **one line**. |
| `{if multicolor_method}` … `{else}` … `{endif}` | Two branches — the first with the CFS (K2 Combo), the second without. |
| *CFS:* `M83`, `M8200 P S…`, `G0 Y150`, `G0 X10`, `M8200 C S0` | Relative extrusion, CFS prepare, move to the cutter and cut the filament. |
| *CFS:* `G0 Y261` … `G0 X115`, `M8200 R` | Move to the purge position behind the bed (Y 291.5), retract the old filament into the CFS. |
| *CFS:* `M104 S[nozzle_temperature_range_high[…]]`, `M8200 L I…`, `T…` | Heat to the filament's **maximum** temperature, load the first slot. |
| *CFS:* 3 × `; FLUSH_START` … `; WIPE` | Flush 90 mm each time (fast / slow in turns), part fan blast for 5 s, retract, wipe the nozzle. The third flush sets the print temperature (`M104`, no wait). |
| *CFS:* `M8200 O`, `M204 S2000`, `G1 Z3 F600`, `G1 X137 F12000` | End the CFS operation, start acceleration, lift 3 mm. |
| *Without CFS:* `T…`, `M104 S…`, `M204 S2000`, `G1 Z3 F600`, `M83` | Load the filament, print temperature, start acceleration, lift 3 mm, relative extrusion. |
| `G1 X105 Y-1 …` to `G1 X155 …` | **Fixed purge line** at Z 0.3 along the **front edge** (Y −1), from X 105 to X 155. |
| `G91`, `G1 X1 Z-0.3`, `G1 X4`, `G92 E0` (`G1 Z1 F600`) | Wipe on the bed, reset the extruder position (without CFS: lift 1 mm). |

**OrcaSlicer**'s built-in Creality K2 profile is older (taken from Creality Print 7.0.1): one block without the CFS branch, `M109` instead of `M104`, and a purge line along the left and front edge — see [OrcaSlicer](../slicers/orcaslicer.md#machine-start-g-code-creality-original-without-kamp-k2).

### Start G-code with KAMP-K2

OrcaSlicer's block with KAMP-K2:

```gcode
M140 S0
M104 S0
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
T[initial_no_support_extruder]
M109 S[nozzle_temperature_initial_layer]
M204 S2000
G1 Z3 F600
M83
LINE_PURGE
```

For Creality Print, the same change is made at the end of each branch — [complete block](../slicers/creality-print.md#start-g-code-with-kamp-k2).

Changes compared to Creality's original:

| Original | With KAMP-K2 | Why |
|---|---|---|
| `M104 S[nozzle_temperature_initial_layer]` | `M109 S[nozzle_temperature_initial_layer]` before `LINE_PURGE` | Waits until the nozzle is at print temperature before `LINE_PURGE` extrudes. In Creality Print's CFS branch the nozzle comes down from the filament's maximum temperature. OrcaSlicer's profile already has `M109`. |
| Fixed purge line | `LINE_PURGE` | Purges just in front of the print area instead of along the bed edge. Must come **after** `T…` — before it, the nozzle is still empty. |
| `G92 E0`, `G1 Z1 F600` | removed | `LINE_PURGE` ends with the nozzle raised by `purge_z_hop` (2 mm, a temporary Z offset). KAMP-K2's `G92` override clears it at the slicer's first `G92 E0` in the layer-change G-code. A `G92 E0` right after the purge would clear it too early — the K2 profile's tool-change G-code then moves to Z 0 and the nozzle can drag across the bed. |

Optional additions:

- Add `MESH=0` to the `START_PRINT` line to skip the mesh for one print (e.g. a calibration cube).
- Add `SET_PRINT_STATS_INFO TOTAL_LAYER=[total_layer_count]` as the **first** line to show the layer count in Fluidd / Mainsail / Mobileraker (Creality's profile does not have it; the layer-change line is on the [OrcaSlicer](../slicers/orcaslicer.md#machine-g-codes) page).
- Do **not** pass `MESH_MIN` / `MESH_MAX` / `PROBE_COUNT` — KAMP calculates them from the objects.

### Exclude objects

KAMP reads the outlines of the objects from the `EXCLUDE_OBJECT_DEFINE` lines at the top of the G-code. The slicer only writes them with **Process / Print settings → Others → G-code output → Exclude objects** enabled. Creality's K2 process presets (both slicers) have it on; generic or self-made presets have it off. (*Label objects* only adds comments and is not needed.)

### What happens at print start

1. `START_PRINT`: CFS prepare (`BOX_START_PRINT`), bed heats, nozzle preheats to 140 °C, `G28`, nozzle wipe (`NOZZLE_CLEAR`), wait for temperatures, `BOX_NOZZLE_CLEAN`, precise `G28 Z`.
2. **Adaptive mesh** — `BED_MESH_CLEAR` + `BED_MESH_CALIBRATE`, probing only the area of the objects on the bed.
3. `T<n>` loads the filament. With Creality Print and the CFS, the start G-code first cuts and unloads, loads the first slot, then flushes and wipes the nozzle at the purge position behind the bed.
4. `M109` waits for the print temperature, `LINE_PURGE` draws the purge line next to the objects and lifts the nozzle, then the first layer starts.

The `G29` / `BED_MESH_CALIBRATE_START_PRINT` that Creality's `master-server` fires before every print only clear the mesh now. In the console you see:

```
BED_MESH_CALIBRATE_START_PRINT deferred (KAMP adaptive in START_PRINT)
Default mesh bounds: ...
Adapted probe count: ...
KAMP purge starting at ..., ... and purging 30mm of filament, requested flow rate is 12mm3/s.
```

If the mesh still covers the whole bed for a small part in the centre, check the [verification](#verify) above and that *Exclude objects* is enabled.

---

## Bed Mesh Limits on the K2

Creality's published K2 config uses:

```ini
[bed_mesh]
mesh_min: 5,5
mesh_max: 255,255
probe_count: 7,7
```

The helper script changes a stock `mesh_min: 5,5` to `20,20` (probing too close to the edge produced false spikes on the K2 Plus). KAMP generates an adaptive sub-mesh within these bounds based on the print's footprint — a small print in the centre probes only the centre region. Always check your own `printer.cfg`.

---

## Removing

Remove menu (option 20) → **6) KAMP-K2**. This removes `[restore_bed_mesh]` and the KAMP include from `printer.cfg`, **restores `gcode_macro.cfg` from `gcode_macro.cfg.pre_kamp`**, deletes `restore_bed_mesh.py` from Klipper's extras and the `KAMP/` folder, and restarts Klipper. Afterwards put [Creality's original start G-code](#crealitys-original-k2-start-g-code) back in your slicer (or select the slicer's system K2 preset again) — without KAMP there is no `LINE_PURGE` and the print starts without a purge line.

!!! note "Duplicate G92"
    `Line_Purge.cfg` defines a `[gcode_macro G92]`. If another config file also defines one, the installer warns you — remove the other one if Klipper reports a duplicate `rename_existing`.

---

## Credits

- [grant0013/KAMP-K2](https://github.com/grant0013/KAMP-K2) — K2 port, `restore_bed_mesh.py` and the config patching (GPL-3.0)
- [kyleisah/Klipper-Adaptive-Meshing-Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) — original KAMP
