# Klipper Gcode Shell Command — K2

Klipper Gcode Shell Command adds the `[gcode_shell_command]` config section and the `RUN_SHELL_COMMAND` G-code, so macros can run shell commands on the printer. It is the `gcode_shell_command.py` Klipper extra by [Arksine](https://github.com/Arksine) (GPLv3).

It is used by the [Git Backup](git-backup.md) macros and is handy for your own macros.

---

## Installation

From the helper script menu, select **option 18 — Klipper Gcode Shell Command**.

The script copies `gcode_shell_command.py` to `/usr/share/klipper/klippy/extras/` and restarts Klipper. If the firmware already ships its own `gcode_shell_command.py`, the stock file is kept.

!!! note "Firmware updates"
    A firmware update replaces `/usr/share/klipper`. The helper script re-applies the file at every boot (and whenever the helper script starts), so configs using `[gcode_shell_command]` keep working.

---

## Example

Add to a `.cfg` file that is included from `printer.cfg` (or directly to `printer.cfg`):

```ini
[gcode_shell_command hello]
command: echo Hello from the K2
timeout: 5.

[gcode_macro HELLO]
gcode: RUN_SHELL_COMMAND CMD=hello
```

After a Klipper restart, run `HELLO` in the console.

| Option | Description |
|---|---|
| `command` | The command to run (arguments are split like a shell would) |
| `timeout` | Seconds before the command is killed (default 2) |
| `verbose` | Print the command's output to the console (default `True`) |

!!! danger "Runs as root"
    Shell commands run with the same rights as Klipper (root). Only add commands you understand.

---

## Removing

Remove menu (option 19) → **18) Klipper Gcode Shell Command**. The helper refuses while any config file in the config folder still contains a `[gcode_shell_command ...]` section (Klipper would fail to start) and lists those files — remove those features (e.g. Git Backup) first. Only the helper's own copy of `gcode_shell_command.py` is deleted, never a file shipped with the firmware.
