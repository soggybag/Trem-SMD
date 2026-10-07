# Trem SMD v1.0: archived revision

- **Git tag:** `v1.0` (commit `a949a5a`, 2026-10-02)
- **Tag note:** v1.0: boards as received from JLCPCB (2026-10-01), with bench-test docs
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
