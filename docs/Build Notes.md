# Trem SMD: build notes

## SW1 (SQ/TRI toggle): sub-miniature switch

- The footprint `Trem:SW_Taiway_200_SPDT` is for a **sub-miniature** SPDT toggle, the same switch as the original Trem 23 layout.
  - Example part: **Taiway 200MSP1T1B1M1QE** (Series 200, ON-ON, M1 solder lugs).
  - [Datasheet](https://www.taiway.com/userfiles/files/products/03.Toggle_Switches/Series%20200_20241231.pdf)
- **Pin pitch 2.54 mm**, body about 5 × 8 mm. Plated slots 2.0 × 0.8 mm take M1 solder lugs or M2 PC pins.
- **Standard mini toggles do not fit** (4.7 mm pitch, ~7 × 13 mm body: Taiway 100, Dailywell 1MS, C&K 7101).
- What to buy: SPDT **ON-ON**, sub-mini, 2.54 mm pitch. Equivalent: **Dailywell 2MS1** series (Tayda etc.); check the listing says 2.54 mm pitch.
- **Panel hole** is smaller than for a mini toggle (~5 mm vs 6.35 mm). Check the drilling template against the switch you buy.
- **Fallback:** mount a mini toggle on the enclosure and wire it to the SW1 pads with three short wires (pin 2 = common).
- 2026-10-01: boards arrived; the user's on-hand toggles are minis, so a sub-mini needs ordering (or use the wired fallback).

## Wiring pads: make them bigger / use a header (2026-10-02)

- The J1–J8 wire pads (TestPoint THT, 1.5 mm pad / 0.7 mm hole) are **too small** to solder off-board wires comfortably.
- **Next revision:**
  - Larger pads, e.g. the `SolderWire-0.5sqmm` style used on the Ugly Face boards (≈ 2.1 mm pad / 0.9–1.0 mm hole).
  - **Or switch the audio/LED wiring to a 1×4 0.1" header** (IN, LED, GND, OUT), as on Ugly Face 1 SMD. Per the pedal standard, centre it on the footswitch so the ribbon runs straight.
  - Keep +9V/GND as larger pads or a 2-pin header.

## Pot direction: RATE and DEPTH turn backwards (2026-10-02)

- RATE is fastest fully CCW, and DEPTH is maximum fully CCW (SHAPE also runs CCW = ramp up). Both feel backwards.
- **Cause:** the pots are mounted on the **back** side. Flipping the footprints to B.Cu mirrored the lug positions, so lug 1 and lug 3 swapped *physically* while the nets didn't. This also reverses the taper: an A (log) pot wired this way behaves like a C (reverse log) one.
- **Fix next revision:** swap the nets on pins 1 and 3 of RV1, RV2 and RV3 (or use a back-side footprint with mirrored pin numbering). That restores both the direction and the intended taper.
- **Current boards:** live with it (the pots are board-mounted).
- Checked all boards (2026-10-02): **only the Trem SMD has panel pots on the back.** DUSG and Wave Folder have back-side trimmers only (direction doesn't matter). Watch for this on any future back-mounted layout.

## Future idea: LED pulses with the LFO (2026-10-02)

- Make the LED (or a second LED) pulse at the LFO rate, as a rate indicator.
- **Options:**
  - LED + resistor from SQR (blinks). Simple, but the TL064 output is weak.
  - An NPN emitter follower on TRI (or on U1A's output) driving the LED, for a smooth pulse that follows SHAPE/DEPTH.
- **Caution (the UF3 lesson):** LED current pulsing at the LFO rate makes supply ripple, and supply ripple bleeds into audio as a tick/thump. Feed the LED from +9V (not VA, the filtered audio supply), keep the current low (high-efficiency LED, 1–2 mA), and/or use a constant-current driver so the supply draw doesn't change with the LFO.
- Possibly make the power LED do both: steady when bypassed, pulsing when on.

## Layout changes for the next revision (2026-10-02)

- **Top-row pots (RATE and DEPTH) are too far apart.** Bring them closer together. Consider **9 mm pots** to free space and allow a tighter, more balanced knob layout.
- **Move the SQ/TRI toggle to the opposite side** of the board/panel.
- Bundle with the other layout items: bigger wiring pads or a 1×4 header centred on the footswitch; pot lugs 1↔3 swapped for the back-side mounting.

## Bench verdict (2026-10-02)

- DC operating point, LFO (≈ 2–17 Hz, SHAPE OK) and VCA balance (bleed only a few mV, nulled with RV4) all check out. **Sounds really good with a guitar.** Signal-generator tests (sections 5–6) still to do.
- The discrete VCA is worth reusing: see `VCA Building Block.md`.


## v2 PCB (2026-10-02): built from the v2 schematic; awaiting review

- **Same 45 × 50 board, 1590B.** v1 is tagged `v1.0` in git.
- **9 mm Alpha RD901F pots** on the back, oriented like Hyper Fuzz v1.1, so they **turn the right way** and the A taper on RATE is restored. **Solder both mounting tabs.**
  - Board coordinates: RATE (33.5, 31) and DEPTH (53.5, 31), now 20 mm apart (were 28 mm); SHAPE (43.5, 50).
- **LED D2** between RATE and DEPTH (43.5, 31). It pulses with the LFO via Q5.
- **Toggles:** SQ/TRI (SW1) moved to the other side (27, 51.5); new **RANGE** (SW2) at (60, 51.5). From outside: RANGE left, SQ/TRI right. Both are Taiway 200 sub-mini.
- **Wiring:** J3/J5/J7/J8 replaced by the **J9 1×4 header** (IN / LED / G / OUT) centred on the footswitch at the bottom edge. +9V, GND, IG and OG are now the larger UF-style solder pads.
- **C3** is a THT radial 470 µ (8 mm), hand-soldered, bottom-left. It is excluded from the JLC BOM/CPL. **RV4** (Bourns 3314J SMD trimmer; v2.0 5k C58391, v2.1 2k C94513) is placed by JLC (extended part).
- **Fully re-placed and re-routed**, with GND pours on both layers. DRC: 0 errors, 0 unconnected (silkscreen warnings only). ERC: 0. Board connectivity identical to the schematic.
- **Silkscreen:** "TREM v2" and "SUPER-FREQ" on the front, RATE/DEPTH/SHAPE assembly labels, and the header labels on the back. Many reference designators overlap (dense board), so tidy them in KiCad as usual.
- **Drill:** `enclosure/Trem_1590B_drill.csv` + SVG (Tayda coordinates, enclosure centre at board (43.5, 69.5)). Jack holes are **not** included, because the jack positions on the 1590B are still to be decided.
  - Check the sub-mini toggle hole size (5.2 mm used) against the actual switch bushing.
- **JLC files generated 2026-10-02** after the user's silkscreen review: `production/Trem_23.zip`, `bom.csv` (64 SMD placements), `positions.csv` (all top), `Trem_23_hand_solder.csv`.
  - Q2/Q3 now carry `FT Rotation Offset = 180`, cancelling the plugin's built-in SOT-363 offset that showed them rotated in v1's JLC preview.
  - **Fixed before release:** the board-sync step hadn't copied LCSC numbers from the schematic (new parts were blank; C6/C7/C8/C14/R13/R17/Q4 still had v1 numbers). All corrected from the schematic, and the BOM was re-checked value by value.
  - `production/JLCPCB/` holds the old v1 outputs; don't upload those.
  - Confirm the CD4053 (C6742) Basic/Preferred status in JLC's BOM step.

## BOM check (2026-10-02)

- Board BOM vs schematic: **every value and LCSC number matches**, and nothing is missing.
- Every LCSC number checked against its value/package: either proven on a received board (UF1, UF3, Trem v1) or looked up on JLC (C1790 100p C0G, C7950 LM358DR2G, C17593 27k, C17380 1M2, C26113 2M2, C17797 680k, C26110 1M5, C17501 180k, C4310 1k5, C1779 4u7 25V X5R, C1729 22n X7R, C28636 560, C27834 5k1). **All correct.**
- Basic vs Extended status **could not be read reliably** from JLC's pages for the high-value resistors (1M2, 1M5, 2M2, 680k, 180k, 1k5, 5k1, 560). Check the fee summary at JLC's BOM step.


## v2 layout tweak (2026-10-02): RATE / DEPTH on the drill guides

- RATE moved to shaft **x = 29.5** (was 33.5) and DEPTH to **x = 57.75** (was 53.5), both at y = 31: on the vertical guide lines in User.Drawings. Their RATE/DEPTH front-silk labels moved with them. C5 and R2 were nudged to clear the new pins.
- Re-routed. One track narrowed 0.2 → 0.19 mm to clear SW2's pad, and C2 pad 2 set to a solid zone connection (single-spoke thermal). DRC: 0 errors, 0 unconnected (silkscreen warnings only). Connectivity identical to the schematic.
- Drill CSV/SVG updated: RATE X = 14.0, DEPTH X = −14.25 (Y 38.5).
- **JLC files regenerated 2026-10-02 23:34** after the user's silkscreen pass: schematic vs board fields 0 mismatches; 64 placements; Q2/Q3 rotation offsets intact.

- **Q4 → MMBT5401 (C8326), 2026-10-03:** JLC was short of MMBT3906 C2910191 at order time. The MMBT5401 is a drop-in SOT-23 PNP (same B-E-C pinout; 150 V, hFE ~200–300, fT 100 MHz). In this output stage (~1.3 mA, C14 22p Miller cap sets the speed) it behaves the same.
