This is a universal, interruptible drop-in heatsoak macro for Klipper. It pauses the print for a bit before it actually starts printing, so the frame/bed/motors can come up to temperature evenly before the nozzle starts laying down plastic. Been running this on several different printers (Voron, RatOS, Sovol, toolchanger, Neptune) so it should adapt to most setups.

To use it, drop `Heatsoak_Universal.cfg` into your Klipper config folder and `[include]` it, then follow the setup steps in the comment block at the top of the file. Full feature list and changelog are in there too.

Quick summary of what it does:
- Figures out how long to soak based on the print time in the filename (falls back to a default if it can't find one).
- Allows actions to be taken while the heatsoak timer is running, such as calibrating nozzle offsets on a Stealthchanger setup.
- Can wait on a chamber sensor if you have one, and skip the soak if the chamber's already warm enough.
- Optionally runs your fans during the soak to ensure the entire chamber heats.
- Shows a popup with Skip/Adjust/Cancel buttons in Mainsail/Fluidd/KlipperScreen.
- Actually pauses the print queue for real during the soak (not just a delay), so cancel/resume behave properly.
- Forces a re-home/re-level/mesh after the soak, to ensure repeatable sensor-less homing values and Z sensor behavior, especially useful with inductive probes that are temp sensitive.

The only real gotcha: you have to double check your printer's actual renamed pause/resume macro names (`rename_existing` on your `[gcode_macro PAUSE]`/`[gcode_macro RESUME]`) before deploying, or the soak will run with no real pause protection and fail silently. Details are in the file.

Originally based on a macro from Contomo (SC.0164) on the Stealthchanger Discord's #user-projects channel — thank you! We've made a number of significant changes/additions since, but credit where it's due.

