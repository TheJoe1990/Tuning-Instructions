# Heatsoak on the Snapmaker U1

An example of the [universal heatsoak](../../Heatsoak.md) set up for the Snapmaker U1 (4-head toolchanger), running Snapmaker's own firmware and Klipper fork. It's one file, `heatsoak_u1.cfg`, plus a one-line change to the slicer start gcode. None of Snapmaker's files get edited, so taking the include line out puts the printer back to stock.

> **Status: beta.** Built from a real U1's config and Snapmaker's actual Klipper source, and checked with the U1 fork's own config parser and a render test of every macro, with and without the bespok3d add-on installed. Real U1 prints have confirmed the soak, flow calibration during the soak (and skipped again afterwards), Resume = Skip Heatsoak, the dock-safe park move (fixed after a T3/T2 collision, see Things to know), and the handoff to Snapmaker's normal start. Watch the first print or two (see [First print checklist](#first-print-checklist)).

## "Paused" means heatsoaking

**For the whole soak, the U1's touchscreen and Fluidd will say the print is Paused. That's normal: Paused = heatsoaking.** The soak holds the print with a real pause, so nothing from the print file runs until the printer is ready.

**The printer is not idle while it says Paused.** It keeps working the whole time: heating the bed, feeding filament, running flow calibration (nozzles heat up, the head moves to the purge chute and back), then parking a head over the bed with the fans on. Things moving while the screen says Paused is expected.

**Resume = Skip Heatsoak.** Pressing **Resume** on the touchscreen (or in Fluidd) during the soak does exactly what the popup's **Skip Heatsoak** button does: it ends the soak early and carries on with Snapmaker's normal start sequence (nozzle clean, Z home, bed mesh, purge line). It doesn't jump straight into printing, and nothing breaks. If you press it while a flow calibration is running, the calibration finishes first, then the soak is skipped.

To end the print instead, use **Cancel Print** in the popup or the normal cancel button.

## What it does

When a print starts:

1. Snapmaker's normal `PRINT_START` stuff runs first, same as always.
2. The print pauses right there, before any of Snapmaker's start sequence (the screen now says **Paused**; that's the heatsoak, see above). You get the heatsoak popup in Fluidd (Skip Heatsoak / +/- minutes / Cancel Print), and the bed heats up.
3. Once the bed is at temperature, the soak countdown starts. While it runs:
   - **Flow calibration** runs now instead of later: the same per-print calibration the U1 already does, just moved into the soak. Each head gets loaded and calibrated in the same order Snapmaker does it.
   - The print's first tool gets picked up and the head **parks over the middle of the bed**. The bed stays where it is (Z isn't homed during the soak, see Things to know). Its part cooling fan and the chamber (cavity) fan turn on to move the hot air around the chamber. If the nozzle is still hot, it cools down over the purge chute first, so nothing drips onto the plate. The head always leaves the dock area straight out first, never diagonally (see Things to know).
4. When the soak is done (or you hit Skip), the tool goes back in the dock, the soak fans turn off, and Snapmaker's own start gcode carries on exactly as it would have: nozzle clean, Z home, **bed mesh**, purge line, print. Homing and the mesh happen hot, after the soak, which is the whole point. The bed mesh is never done during the soak.

Flow calibration is safe to move because the U1 firmware remembers which heads it already calibrated for the current print. When the stock start gcode asks for flow calibration again after the soak, those heads are skipped. The touchscreen's "flow calibration" on/off setting still works too.

Soak length scales with the print time in the filename. The U1's default Orca filename already includes it (`name_PLA_1h30m.gcode`). The default is 5 seconds of soak per minute of print, minimum 5 minutes, maximum 45. Prints with a bed under 55°C or shorter than 5 minutes skip the soak entirely.

## What you need

- A U1 where you can edit `printer.cfg` (SSH/root access, or a mod that exposes the config folder).
- Fluidd or Mainsail to see the popup. The touchscreen only shows the print as Paused (which means heatsoaking, see above).

Works with or without the bespok3d add-on.

## Install

1. Copy `heatsoak_u1.cfg` into the config folder, next to `printer.cfg` (`/home/lava/printer_data/config/`).
2. Open `printer.cfg` and add this line **after the last `[include ...]` line and above the `#*# <--- SAVE_CONFIG --->` block** at the bottom:
   ```
   [include heatsoak_u1.cfg]
   ```
   It has to be the last include. Klipper merges sections that share a name, and the last one loaded wins.
3. In Snapmaker Orca, go to **Printer settings → Machine G-code → Machine start G-code** and change the very first line from just `PRINT_START` to:
   ```
   PRINT_START BED_TEMP={bed_temperature_initial_layer_single} TOOL_TEMP={nozzle_temperature_initial_layer[initial_extruder]} TOOL={initial_extruder}
   ```
   Don't touch anything else in Snapmaker's start gcode. Without `BED_TEMP` the soak never triggers, because the stock line doesn't pass any temperatures.
4. Do a `FIRMWARE_RESTART`. The console should show no `Heatsoak:` errors.

To uninstall, delete the include line (and the slicer change if you want) and restart.

## Settings

Everything is in `[gcode_macro _HEATSOAK_CFG]` near the top of the file. The U1-specific ones:

| Setting | Default | What it does |
|---|---|---|
| `u1_flow_cal_during_soak` | `True` | Run flow calibration during the soak. `False` leaves it where Snapmaker had it. |
| `u1_park_x` / `u1_park_y` | `135` / `135` | Park spot, the middle of the bed. |
| `u1_park_part_fan_speed` | `0.6` | Parked head's part cooling fan, 0.0–1.0. |
| `u1_park_cavity_fan_speed` | `1.0` | Chamber side fan (`cavity_fan`), 0.0–1.0. |
| `u1_park_max_nozzle_temp` | `120` | A nozzle hotter than this cools over the purge chute before parking over the bed. |
| `chamber_sensor` | `"cavity"` | Shows chamber temp in the popup. |
| `soak_nozzle_temp` | `0` | Nozzles stay off during the soak (the head is over the bed). |

The soak time settings (`soak_sec_per_print_min`, `soak_min_seconds`, `soak_max_seconds`, `min_bed_temp_to_trigger`) work the same as on every other printer.

## Things to know

- **Use Fluidd for the popup.** The touchscreen just shows the print as Paused (= heatsoaking). **Resume = Skip Heatsoak**: it skips the rest of the soak and starts the print sequence normally. The console shows `Heatsoak: Resume pressed during the soak -- skipping the rest of the soak`.
- **The popup freezes while flow calibration runs** (a few minutes per head). The printer is busy calibrating and Klipper can't redraw the popup until that finishes; it catches up right after. Nothing is stuck. Pressing Resume during that time just skips the rest of the soak once calibration finishes. Cancel from anywhere cancels cleanly.
- **Bed max is 100°C on the U1** (Snapmaker's own limit in `printer.cfg`). A filament profile asking for more (ASA/ABS profiles often want 105–110°C) fails right at print start with `heater_bed: Requested temperature (105.0) out of range (0.0:100.0)` and the print cancels. That happens with or without the heatsoak; set the bed to 100°C or lower in that filament profile.
- **Z isn't homed during the soak, so the bed doesn't move.** An earlier version did Snapmaker's rough Z home during the soak to raise the bed under the parked head. On real prints that left the U1's default bed mesh loaded (its `G28 Z` loads it), and Snapmaker's start sequence then stopped at the nozzle clean with `Must home Z axis first` (System Anomaly 522). Raising the bed wasn't worth it, so it was removed; `park_z` from the shared settings isn't used on the U1. If your copy has `u1_rough_z_home_during_soak`, update it (or set that to `False`).
- **Tool docks: no diagonal moves.** The docks are at the back (Y ~332) and each head's "idle line" is at Y 250. Behind that line the head is among the docks, and moving sideways there can hit the next tool. An earlier version of this file did one diagonal move from the purge chute to bed center, and T3 hit T2. Now every soak move leaves the dock area with Snapmaker's own idle move (straight out in Y first), then goes Y, then X, never both at once. If your copy is older than that fix, update it before printing.
- **Why it doesn't use the normal PAUSE:** on the U1, `PAUSE` parks over the purge chute, drops every nozzle to 40°C and turns the fans off. The soak uses the bare `PAUSE_BASE` instead, which only stops the file.
- **Firmware updates.** Two Snapmaker macros are copied into this file: the `PRINT_START` body (now `_SNAPMAKER_STOCK_PRINT_START`) and `CANCEL_PRINT` (plus one teardown line). If a firmware update changes those in Snapmaker's `fluidd.cfg`, copy the new versions over. An update that resets `printer.cfg` will also drop the include line, so check it's still there after updating.
- **Other macros it overrides** (same name, last one wins): `PRINT_START`, `CANCEL_PRINT`, and `RESUME` (passes straight through to Snapmaker's resume when no soak is running). `PAUSE` is left alone.
- If a flow calibration fails during the soak, the soak carries on. Snapmaker's start gcode tries that head again after the soak and reports the error the normal way if it fails again.

## First print checklist

On the first print with a hot bed and a print over ~10 minutes:

- [ ] Screen says Paused (that's the soak), popup shows up in Fluidd and the bed heats. Nothing else moves until the bed is at temp (apart from the X/Y home).
- [ ] Flow calibration runs for each head the print uses, and is **skipped** again after the soak (console: `flow calibration ... has been finished`, or nothing at all).
- [ ] Head leaves the chute straight forward (Y) first, then sideways (X), and parks over the middle of the bed (the bed doesn't move). Part fan and chamber fan come on. **It must never cut diagonally across the docks.**
- [ ] After the soak: head goes back in the dock, fans stop, and Snapmaker's normal sequence runs (clean, Z home, bed mesh, purge line).
- [ ] Try a Cancel during a soak once: heaters off, popup gone, no more `Heatsoak` messages in the console.
- [ ] Try Resume on the touchscreen during a soak once: it should act as Skip Heatsoak and go straight to Snapmaker's start sequence, with no error popup.
