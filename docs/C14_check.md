# Identifying Creality Ender-3 V3 SE C13 vs C14 motherboard

This short guide shows how to determine whether your Ender 3 V3 SE has the C13 or C14 motherboard revision.

WARNING: Always power off and unplug your printer before opening the case.

Steps:

1. Power off the printer and unplug it.
2. Remove the lower case panel to expose the mainboard (see your printer manual).
3. Look for the printed board identifier on the PCB near the connectors or silkscreen. Typical identifiers:
   - `CR4NS200320C14` &rarr; C14 (this firmware target)
   - `CR4NS200320C13` &rarr; C13 (do NOT use this firmware)
4. If you find a different label or are unsure, compare the board to photos and docs from Creality or ask in the project's issues describing the exact board markings.

Notes:
- The file [Marlin/src/pins/stm32f4/pins_CREALITY_F401.h](Marlin/src/pins/stm32f4/pins_CREALITY_F401.h) contains the `BOARD_INFO_NAME` used in this repo for the C14 board.
- If your board says `CR4NS200320C13` or similar with `C13`, use firmware explicitly built for that revision (this repo is not compatible).

If you want, I can add example photos showing where the label usually is — say the word and I'll include a couple of photos in this docs page.