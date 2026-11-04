# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Hardware project: a 2-layer PCB in EasyEDA Pro. There is no code, build, lint or test. `README.md` describes the board (pinout, I²C addresses, firmware notes, BOM). `production/` holds exported fabrication files only.

**The source of truth is the EasyEDA cloud project, not this repo.** Schematic and PCB can only be read or changed through a running EasyEDA Pro client, using the `easyeda-api` skill and its bridge (ports 49620-49629).

| Document | UUID |
| -------- | ---- |
| Project `LMP91000 AFE` | `880980bf696c4bf58844fc25b96d1f9f` |
| PCB | `35d2b91d3449eaa3` |
| Schematic page | `83b967512e647cc7` |

## Working on the board

- After any PCB change: rebuild both GND pours, save, reopen the document, run DRC, then re-export all three files in `production/` (Gerber zip, BOM, CPL). Keep the `README.md` tables in sync.
- Treat DRC as clean only when `eda.pcb_Drc.check(true, false, true)` returns an empty array on a freshly reopened document.
- Panel documents cannot be read or exported through the API. If one is added, export it by hand in the EasyEDA UI.

## EasyEDA API behaviour (Pro 3.2.209)

- Tracks and vias created through the API raise false "No Connection" DRC errors until the PCB is saved, closed and reopened (`dmt_EditorControl.closeDocument` then `openDocument`).
- `rebuildCopperRegion()` can silently return `undefined` and leave the old fill in place, which would then be exported. If so, delete the fill with `pcb_PrimitivePoured.delete`, save, reopen and rebuild.
- `sch_Netlist.getNetlist()` hangs past the 30 s bridge timeout. Use `pcb_Net.getNetlist()`.
- Enums are not defined in bridge code. Use the values stored on the board: layers top `1`, bottom `2`, multi `12`; hole `"ROUND"`; pad shape `"ELLIPSE"`.
- `getState_Hole()` on component pads returns the diameter divided by 10. Read `hole` from `JSON.parse(JSON.stringify(pad))` instead.
- PCB coordinates are in mil and Y points up.
- `getGerberFile` and the other export calls return a browser `File`. Transfer it through the bridge as base64.

## Design constraints to preserve

- WE, RE and CE run on the top layer without vias. Their bottom-layer stubs to JP1 join through the through-hole pads.
- M2 holes sit 2.0 mm from the edges, not 2.5 mm: at 2.5 mm an M2 screw head (Ø3.8 mm) hits the J1 header body.
- The sensor footprint name says `PCD14`, but 13.5 mm is the correct pin circle (the 9.55 mm chord on the ME3 drawing). The Ø14 on that drawing is a ring on top of the sensor. Do not "fix" it.
- The ME3 pin drawing is a view from the pin side, so WE/RE/CE appear mirrored compared to the top-side footprint. The current footprint is correct.
- The sensor symbol description says `2=RE, 3=CE`, but the actual pins are `2=C, 3=R`. The nets are correct; only the description is stale.
- ADS1115 `ADDR` is tied to VDD (`0x49`) to avoid the fixed LMP91000 address `0x48`.
