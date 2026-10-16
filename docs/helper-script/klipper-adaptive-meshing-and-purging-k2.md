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

## Slicer Setup (OrcaSlicer)

1. **Print Settings → Others → Output options:** enable **Label objects** (KAMP needs the object outlines).
2. **Printer Settings → Machine G-code → Machine start G-code** — one command per line:

    ```gcode
    START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
    T[initial_no_support_extruder]
    LINE_PURGE
    M204 S2000
    M83
    ```

- `LINE_PURGE` **must come after** `T[initial_no_support_extruder]` — on the K2 Combo the CFS loads the filament on `T<n>`.
- Add `MESH=0` to the `START_PRINT` line to skip the mesh for a print.
- **Creality Print** still triggers its own full mesh — OrcaSlicer is recommended.

See [OrcaSlicer](../slicers/orcaslicer.md) for the rest of the printer profile, including the bed-mesh limits for the K2.

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

Remove menu (option 19) → **6) KAMP-K2**. This removes `[restore_bed_mesh]` and the KAMP include from `printer.cfg`, **restores `gcode_macro.cfg` from `gcode_macro.cfg.pre_kamp`**, deletes `restore_bed_mesh.py` from Klipper's extras and the `KAMP/` folder, and restarts Klipper. Remember to remove `LINE_PURGE` from your slicer start G-code afterwards.

!!! note "Duplicate G92"
    `Line_Purge.cfg` defines a `[gcode_macro G92]`. If another config file also defines one, the installer warns you — remove the other one if Klipper reports a duplicate `rename_existing`.

---

## Credits

- [grant0013/KAMP-K2](https://github.com/grant0013/KAMP-K2) — K2 port, `restore_bed_mesh.py` and the config patching (GPL-3.0)
- [kyleisah/Klipper-Adaptive-Meshing-Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) — original KAMP
