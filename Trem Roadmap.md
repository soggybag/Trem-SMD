# Trem SMD — roadmap, notes and to-do

Discussion 2026-09-29. Baseline: **Trem SMD** (`Trem_23`, TL064 LFO + discrete LM3086-style VCA, knobs RATE / SHAPE / DEPTH, SQ/TRI toggle).

## How the LFO works (from the schematic)

- **U1C Schmitt trigger** → SQR. The triangle comes in through R6 10k, with R7 22k hysteresis, so the triangle swings about ±1.45 V around VB (≈2.9 V p-p). The reference input is R8 68k to VB, with **C7 33n from the output to that input**: when the output flips, C7 briefly pulls the reference with it, so each edge ramps over about 68k × 33n ≈ **2.2 ms** instead of snapping. That's the edge softening that keeps the square's clicks out of the audio.
- **RATE** (A10k, R9 2k2 to VB) divides the square down. The **SHAPE** diodes D3/D4 steer it into two paths (6k8 + each side of SHAPE B100k) into the **U1D integrator (C6 470n)** → TRI.
  - The total resistance of the two paths is always about 114k, so SHAPE changes symmetry, not speed.
  - The diode drop (~0.6 V) is subtracted from the RATE voltage. That makes the bottom of RATE very slow and touchy; it's also why R9 is 2k2.
  - Estimate: **about 17 Hz at full RATE, down to well under 1 Hz**.
- **U1A** (DEPTH → R11, R12 100k, R13 270k offset, **C8 10n smoothing ≈ 160 Hz corner**) turns the LFO into tail current (R14 8k2) for the VCA pair.

## Idea 1 — Frobnicator-style RANGE switch (audio-rate "pseudo ring mod")

**Yes, it can be added.** The Trem LFO has the same Schmitt + integrator structure as the Frobnicator's 3080 design. The range change is mainly the timing capacitor, but **three parts set speed limits, not just one:**

| Part | Role | LFO range | AUDIO range |
|---|---|---|---|
| **C6** integrator | sets the rate | 470n (≈0.2–17 Hz) | **≈10n (≈10–800 Hz)**; 15n ≈ 530 Hz top, 4n7 ≈ 1.7 kHz top |
| **C7** Schmitt edge softener | 2.2 ms edges | 33n | **≈1n (~70 µs)** |
| **C8** U1A smoothing | 160 Hz low-pass on the modulation | 10n | **≈0–1n** (otherwise audio-rate modulation above ~160 Hz is filtered out) |

**Your C7 question: yes, it matters a lot at audio rate.**
- At 200 Hz the period is 5 ms, and C7's 2.2 ms edges eat most of it. The square turns into a rounded, slow-edged wave.
- The Schmitt also fights its own reference during each transition, so at higher rates it switches sluggishly or unreliably. You'd get a lower top frequency, a mushy wave, and possibly stalling.
- So in AUDIO mode, C7 must shrink along with C6, roughly in proportion (about 1n). Its job, softening clicks, doesn't matter at audio rate anyway: there the "clicks" *are* the tone.
- C8 has the same problem: its 160 Hz low-pass would smear the modulation. Either switch it too, or reduce it permanently to about 1n and let C7 handle the click-softening in LFO mode. Needs a listen test for LFO thump.

**How to switch three caps:**
1. **DPDT or 3PDT toggle:** keep the small value fitted permanently and let the switch add the big one in parallel (C6: 10n + switched 470n; C7: 1n + switched 33n; C8 either switched or permanently 1n). A DPDT covers C6 + C7 if C8 becomes a fixed 1n.
2. **Electronic:** a **CD4053** (three SPDT analog switches, JLC basic C6742) switches all three from a simple SPST toggle. Nothing about the toggle's wiring is audio-critical, and it's easy to add a footswitch later.

**What else changes at audio rate:**
- **Control bleed-through:** the VCA's LFO feedthrough (nulled by the RV4 BALANCE trim) is inaudible at 3 Hz but a buzz at 300 Hz. Trimming RV4 carefully matters more; consider making it reachable, or adding a note on the silkscreen.
- **DEPTH** becomes the ring-mod amount. Tremolo is amplitude modulation, so you hear the guitar plus sidebands (not a true ring mod, which suppresses the carrier). That's the "pseudo ring mod" the Frobnicator is known for.
- **SQ/TRI:** at audio rate the square chops, giving a buzzy, octave-like, gated sound, while the triangle gives smoother, bell-like sidebands. Both are useful. SHAPE skews the triangle to a saw, which adds more sidebands.
- **RATE feel:** the diode-steered divider makes the bottom of RATE touchy. In AUDIO mode that end is roughly 10–40 Hz, which is useful as "growl" territory.
- **Panel:** needs a second toggle (RANGE) next to SQ/TRI. The Taiway 200 sub-mini fits the existing style; check space on the 45×50 board.

**Please send the Frobnicator schematic.** I'll compare its range-switch values and how it treats its equivalent of C7 and C8 before we pick values.

## Carried-over items (from the first JLC order)

- Extended-part reduction, deferred 2026-09-25:
  - C14 10p → 22p C0G C1804 (basic).
  - Q4 MMBT3906 → MMBT5401 C8326 (basic, same SOT-23 pinout).
  - Optionally hand-solder C3 (470µ) and RV4 (3314J).
  - Keep Q1 MMBT5089 + R17 4M7, R13 270k, the TL064, and the MBT3904DW1 pairs.
- CPL rotation: C3, RV4, Q2, Q3 showed 180° in JLC's viewer → `FT Rotation Offset` = 180 (confirm it was applied before the next order).
- Pot footprint: Omeg PC16BU (12.55 mm shaft-to-pins) vs Alpha RV16AF-41 (16 mm). Confirm against the real pots.

## Road map

1. **Phase 0 — bench test on a built Trem:**
   - Temporarily put ~10n in place of C6 and ~1n in place of C7 (or tack them on with a toggle) and listen at audio rate.
   - Try C8 at 1n in both modes.
   - Check how audible the bleed-through is, and trim RV4.
2. **Phase 1 — pick the switching method:** DPDT toggle (C6 + C7, C8 fixed at 1n) vs CD4053 (all three).
3. **Phase 2 — Trem SMD v2:** RANGE toggle + switched caps + extended-part reductions + rotation offsets + pot footprint fix → PCB → JLC.

## To-do

- [ ] Get the Frobnicator schematic from the user; compare its range caps and edge-softening cap.
- [ ] Phase 0 bench test: C6 10n / C7 1n / C8 1n; note the usable AUDIO range and any stalling at the top of RATE.
- [ ] Measure bleed-through at audio rate with RV4 nulled; decide whether RV4 needs panel access or a better null.
- [ ] Decide on DPDT vs CD4053 switching.
- [ ] Check board space for a second Taiway 200 toggle (45×50 board, back-side panel parts).
- [ ] Fold in the carried-over items (extended parts, rotation offsets, pot footprint).
- [ ] Schematic → layout → JLC for Trem SMD v2.

## Open questions

- Target AUDIO top frequency: ~500 Hz (15n) or ~800 Hz (10n)?
- Should RANGE be a toggle, or a footswitchable option (CD4053 makes that easy)?
- Keep C8 at 10n in LFO mode for the smoothest thump-free sweep, or accept 1n everywhere?
