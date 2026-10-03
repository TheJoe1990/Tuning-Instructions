This is a universal, interruptible drop-in heatsoak macro for Klipper. It pauses the print for a bit before it actually starts printing, so the frame/bed/motors can come up to temperature evenly before the nozzle starts laying down plastic. Been running this on several different printers (Voron, RatOS, Sovol, toolchanger, Neptune) so it should adapt to most setups.

To use it, drop `Heatsoak_Universal.cfg` into your Klipper config folder and `[include]` it, then follow the setup steps in the comment block at the top of the file. Full feature list and changelog are in there too.

Quick summary of what it does:
- Figures out how long to soak based on the print time in the filename (falls back to a default if it can't find one).
- Allows actions to be taken while the heatsoak timer is running, such as calibrating nozzle offsets on a Stealthchanger setup.
- Can wait on a chamber sensor if you have one, and skip the soak if the chamber's already warm enough.
- Optionally runs your fans during the soak to ensure the entire chamber heats.
- Shows a popup with Skip Heatsoak / +/- minutes / Cancel Print buttons in Mainsail/Fluidd/KlipperScreen.
- Actually pauses the print queue for real during the soak (not just a delay), so cancel/resume behave properly. See ["Paused" means heatsoaking](#paused-means-heatsoaking) below.
- Pressing Resume during the soak skips the rest of the soak (same as the Skip Heatsoak button) instead of starting the print on an unsoaked bed.
- Forces a re-home/re-level/mesh after the soak, to ensure repeatable sensor-less homing values and Z sensor behavior, especially useful with inductive probes that are temp sensitive.

![Heatsoak popup](photos/Heatsoak%20Popup.png)

### "Paused" means heatsoaking
The soak works by really pausing the print right at the start (that's what stops the rest of the file from running until the printer is ready). So for the whole soak, **your printer's screen and Mainsail/Fluidd will say the print is Paused. That's normal: Paused = heatsoaking.**

The printer is **not idle** while it says Paused. During the soak it keeps working: heating the bed, holding the nozzle at its standby temp, running fans, and running whatever soak tasks your printer has (parking, tool calibration, flow calibration, etc.). Don't be surprised if things move while the status says Paused.

What the buttons do during the soak:
- **Resume** (on the printer's screen or in Mainsail/Fluidd) = **Skip Heatsoak**. It ends the soak early and starts the print sequence properly (re-home, level, mesh). It does not jump straight into printing. On most printers the console also shows one red `Resume pressed during the soak` line when you do this. That's expected.
- **Skip Heatsoak** (popup) = same as Resume.
- **+/- minutes** (popup) = make the soak longer or shorter.
- **Cancel Print** (popup), or your normal cancel button = cancel the whole print. The soak shuts itself down cleanly.

While a long soak task is running (e.g. a calibration), the popup can't refresh until that task finishes. It catches up right after; nothing is stuck.

The real setup gotcha: this file never overrides your printer's own `CANCEL_PRINT` or `RESUME` (Klipper would silently merge over them). Instead you add one line to each: `_HEATSOAK_CANCEL_TEARDOWN` at the top of your `CANCEL_PRINT`, and `_HEATSOAK_RESUME_CHECK` at the top of your `RESUME`. Mainsail/Fluidd client.cfg and Happy Hare have variable hooks for both. After a restart the console shows a red `Heatsoak:` error if either hookup is missing. The pause/resume macro names are found automatically. Details are in the file.

This started life as [Contomo's heatsoak macro](https://github.com/Contomo/klipper-questionable-macros/blob/main/macro-examples/interruptable_heatsoak_print_start.cfg) — big thanks to him for the original work. We've since built on it with a number of significant changes and additions.

### Example configs
Real setups for specific printers, as worked examples of hooking this into a printer's existing macros without breaking them:
- [Snapmaker U1](Examples/Snapmaker%20U1/README.md): stock Snapmaker firmware. Runs the U1's per-print flow calibration during the soak, then parks a head over the middle of the bed with the fans circulating the chamber air.

### Done (from the old to-do list)
- Countdown now waits until the bed actually reaches target temp; the soak clock doesn't start while it's still heating.
- Build-plate layout: an SVG of every part's outline is sent to the Mainsail/Fluidd console when the soak starts (the popup itself can't show pictures).
- Per-tool work during the soak: `PRINTER_HEATSOAK_PARK` is now a general "tasks to run during the heatsoak" hook. The SV08 toolchanger uses it for tool calibration and the Snapmaker U1 for flow calibration.

### To do / future ideas
- Keep the popup updating while a long soak task runs. Right now a calibration holds Klipper's command queue, so the popup freezes until it finishes. That would mean breaking tasks into smaller steps.
- Split the file into a shared engine file plus a small per-printer settings file, so updating is just replacing one file instead of re-merging your edits.
- More example configs (SV08 toolchanger, RatOS, Voron 0 with Happy Hare, Sovol, Neptune 4 Max).
