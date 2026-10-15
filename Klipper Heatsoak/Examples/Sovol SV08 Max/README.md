# Heatsoak on the Sovol SV08 Max

An example of the [universal heatsoak](../../Heatsoak.md) set up for the Sovol SV08 Max (enclosed, single-zone bed), running Sovol's stock firmware (2.1.9) and Sovol's Orca profile. It's one file, `heatsoak_sv08max.cfg`, plus three small edits to Sovol's `Macro.cfg`. The slicer start gcode stays exactly as Sovol ships it.

> **Status: staged, not yet print-tested.** Built from a real SV08 Max's config and passes the fleet validator (Klipper config parse + a render test of every macro). The first real soak print hasn't happened yet. Watch the first print or two (see [First print checklist](#first-print-checklist)).

*Written with Claude (AI); tested and maintained by TheJoe1990.*

## What it does

Sovol's Orca start gcode is:

```
G28
M140 S<bed>
M190 S<bed>
START_PRINT
M104 S<nozzle>
M109 S<nozzle>
```

So by the time `START_PRINT` runs, the printer is homed and the bed is already at temperature. Then:

1. `START_PRINT` (the heatsoak entry) works out the soak length from the print time in the filename and pauses the print. You get the heatsoak popup in Mainsail (Skip Heatsoak / +/- minutes / Cancel Print).
2. The head parks over the middle of the bed (Z 10), the nozzle holds at 150°C, and both part cooling fans run at 100% to move the hot air around the enclosure.
3. When the soak is done (or you hit Skip), the fans turn off and Sovol's own start sequence runs as `START_PRINT_BASE`. It re-homes first (`G28` on the warmed frame), then does the nozzle clean, the adaptive bed mesh and the rest, same as stock.
4. The slicer then heats the nozzle and prints.

Soak length: 5 seconds of soak per minute of print, minimum 5 minutes, maximum 45. Prints with a bed under 55°C skip the soak; a filename with no print time gets a 5-minute soak. Sovol's default Orca filename already includes the print time (`name_PLA_0.2_12h36m.gcode`).

## Install

Back up `printer.cfg` and `Macro.cfg` first.

1. Copy `heatsoak_sv08max.cfg` into `~/printer_data/config/`.
2. In `printer.cfg`, add the include on the line **right after** `[include Macro.cfg]`:
   ```
   [include Macro.cfg]
   [include heatsoak_sv08max.cfg]
   ```
   It has to come after `Macro.cfg`. Klipper merges sections that share a name, and the last one loaded wins.
3. Edit `Macro.cfg` (three changes):

   **a) Rename Sovol's `START_PRINT` to `START_PRINT_BASE`** and add two lines at the top of it:
   ```
   [gcode_macro START_PRINT_BASE]
   gcode:
       SET_IDLE_TIMEOUT TIMEOUT={printer.configfile.settings.idle_timeout.timeout|int}
       {% if params.BED is defined %}
       G28  ; after a heatsoak: re-home on the warmed frame
       {% endif %}
       CLEAR_PAUSE
       ... (rest of Sovol's macro unchanged)
   ```
   **b)** First line of `[gcode_macro CANCEL_PRINT]`'s gcode: `_HEATSOAK_CANCEL_TEARDOWN`

   **c)** First line of `[gcode_macro RESUME]`'s gcode: `_HEATSOAK_RESUME_CHECK`

4. `FIRMWARE_RESTART` while the printer is idle. The console should show no `Heatsoak:` errors.

To uninstall, remove the include line and undo the three `Macro.cfg` edits (or restore your backup), then restart.

## Settings

Everything is in `[gcode_macro _HEATSOAK_CFG]` near the top of the file. The ones set for the SV08 Max:

| Setting | Value | Why |
|---|---|---|
| `soak_part_fan_speed` | `1.0` | Enclosed printer: run the part fans during the soak. Sovol's `M106` macro drives both `fan0` and `fan1`. |
| `soak_nozzle_temp` | `150` | Template default, kept to limit ooze. |
| `soak_fans` | `[]` | Exhaust (`fan3`) and auxiliary (`fan2`) left alone. |
| `chamber_sensor` | none | `chamber_hot.cfg` isn't included on a stock machine. |

The soak time settings (`soak_sec_per_print_min`, `soak_min_seconds`, `soak_max_seconds`, `min_bed_temp_to_trigger`) work the same as on every other printer.

## Things to know

- **Why `START_PRINT` and not `PRINT_START`:** Sovol's `plr.cfg` (power-loss recovery) already defines `[gcode_macro PRINT_START]`. A second section with that name would silently merge over it. Sovol's Orca profile calls `START_PRINT`, so that's the entry.
- **The popup says "Nozzle for print: 200C".** Sovol's `START_PRINT` line passes no temperatures, so that's the default. It's cosmetic: the slicer sets the real nozzle temperature after `START_PRINT` returns. The bed temp comes from `heater_bed.target`, which the slicer's `M190` already set.
- **Firmware updates.** A Sovol OTA update can overwrite `Macro.cfg` and `printer.cfg`. Afterwards, check the include line, the rename and the two hook lines. If a hook is missing, Klipper prints a `Heatsoak:` startup error.
- **Never restart Klipper while it's printing.** The heatsoak only loads at the next restart, so install it between prints.
- **Single-zone bed.** The 500 mm bed has one heater, so there's nothing zone-specific to set.

## First print checklist

On the first print with a hot bed and a print over ~10 minutes:

- [ ] After the slicer's `G28` and bed heat, the popup shows up in Mainsail and the head parks over the middle of the bed.
- [ ] Both part fans come on during the soak and turn off when it ends.
- [ ] After the soak: re-home, nozzle clean, adaptive mesh, then the slicer heats the nozzle and prints.
- [ ] Try Skip Heatsoak once: goes straight to Sovol's start sequence.
- [ ] Try a Cancel during a soak once: heaters off, fans off, popup gone, no more `Heatsoak` messages in the console.
