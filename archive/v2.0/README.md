# Trem SMD v2.0: archived revision

- **Git tag:** `v2.0` (commit `7549410`, 2026-10-06)
- **Tag note:** v2.0: ordered from JLCPCB 2026-10-04 (9 mm pots, RANGE, LFO LED, 1x4 header)
- **Archived:** 2026-10-06 with `_tools/archive_version.py`

## Finding a part

1. Open `Trem_23_ibom.html` in a browser. Type a reference (R14) or a value (100k) in the filter box; the part lights up on the board.
   - Hover over a pad to see its net.
   - F / B switch sides.
2. Find the same reference in `Trem_23_schematic.pdf` with Cmd-F.
3. `Trem_23_board_front.pdf` / `_back.pdf` are printable: page 1 has the outlines and references, page 2 the copper. The back is mirrored, as you see it holding the board.

## Files

- `Trem_23_board_back.pdf`
- `Trem_23_board_front.pdf`
- `Trem_23_ibom.html`
- `Trem_23_render_bottom.png`
- `Trem_23_render_top.png`
- `Trem_23_schematic.pdf`
- `docs`/
- `parts.csv`
- `production`/

`production/` holds the files sent to JLCPCB for this revision: Trem_23.zip, Trem_23_hand_solder.csv, bom.csv, designators.csv, netlist.ipc, positions.csv.
