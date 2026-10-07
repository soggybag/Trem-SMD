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

