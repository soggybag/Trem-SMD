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

## Trem v2 with RANGE vs a separate Frobnicator (2026-10-02)

- Bench test proved the Trem's discrete VCA works well (bleed a few mV, sounds good). With the planned **RANGE switch** (audio rate), an **LFO-rate LED**, and the layout fixes, Trem v2 covers everything the Frobnicator does: RATE, DEPTH, RANGE, rate LED. It adds SHAPE and SQ/TRI.
- **So the separate Frobnicator SMD may be moot.** What it would still offer:
  - the 3080/LM13700 OTA character (if that's what makes the original special);
  - a simpler 3-knob pedal;
  - a socket for the user's vintage 3080.
- Suggestion: make **Trem v2 the audio-rate platform**. Keep the Frobnicator SMD draft on hold unless an A/B against the original shows the OTA sound is worth a separate pedal.
- Borrow from the Frobnicator: its rate LED sits *in the control-current path* (no extra supply draw that varies with the LFO). In the Trem the tail current (≤ 0.3 mA) is too small to light an LED that way, so drive it from +9V with a follower instead (see Build Notes).

## Trem v2 plan (2026-10-02): 1590B, audio rate, fewer extended parts

**Decided:** stay in the **1590B**. v2 = RANGE (audio rate) + bench fixes (pot lug swap, RATE/DEPTH spacing with 9 mm pots, SQ/TRI toggle to the other side, bigger pads or 1×4 header, JLC rotation offsets) + extended-part reduction. Optional: LFO-rate LED. Work in place in the Trem repo (from tag `v1.0`).

**RANGE:** small "audio" caps always fitted; CD4053 (C6742) switches the big "LFO" caps in parallel for C6 (integrator), C7 (Schmitt edge) and C8 (DEPTH smoothing). Driven by one SPDT/SPST toggle. RV4 bleed trim matters more at audio rate.

**Extended parts in v1 (9 lines, $3 each on Economic):** C3 470µ (C970706), C14 10p (C157608), Q1 MMBT5089 (C116160), Q2/Q3 MBT3904DW1 (C110150), Q4 MMBT3906 (C84105), R13 270k (C17589), R17 4M7 (C17674), RV4 3314J (C58391), U1 TL064 (C402159). Checked basic: C6 470n (C13967), C10 220p (C107145).

| Part | Change | Status |
|---|---|---|
| C14 10p | → 22p C0G **C1804** | basic ✓ |
| Q4 MMBT3906 | → same part, CJ **C2910191** | Preferred Extended ✓ (no loading fee on Economic) |
| R17 4M7 | → 2 × 2M2 in series (**C26113**) = 4.4M | **status unverified** (JLC page ambiguous). If 2M2 turns out Extended at BOM upload, this saves nothing: revert to one 4M7 |
| R13 270k | → 220k (**C17556**) + 47k (**C17713**) in series = 267k | 47k basic ✓; 220k listed, confirm basic |
| RV4 trim | hand-solder (THT 3362P/3296W or the 3314J) | no fee |
| C3 470µ | hand-solder THT radial 470µ 16V (or find a basic SMD electrolytic) | no fee |
| U1 TL064 | keep ($3), or 2 × TL072 (C6961 basic, +~5 mA and board space) | decide; LM324 rejected (skews the LFO) |
| Q1 MMBT5089 | keep, or BC847C/BC850C (hFE 420–800) if a Basic/Preferred one exists | check at order |
| Q2/Q3 MBT3904DW1 | **keep**: matched dual for VCA balance (one line covers both) | |
| new: CD4053 | C6742 | confirm Basic/Preferred at order; fallback 3PDT toggle |

Result: 9 extended lines → about 2–3 (Q2/Q3, maybe U1, maybe Q1).

### v2 schematic draft (2026-10-02): done, awaiting review

- **Decisions:** keep the TL064; LED included.
- **RANGE block:** U2 CD4053 (C6742). C6/C7/C8 are now the always-fitted AUDIO values (10n / 1n / 220p). The LFO caps C17 470n, C18 33n, C19 10n switch in through U2 when SW2 (Taiway 200 sub-mini, hand-soldered) is at +9V. C20 100n decouples U2.
  - Rates: LFO ≈ 2–17 Hz (as v1); AUDIO ≈ 100–800 Hz.
- **LED:** Q5 MMBT3904 (C20526) emitter follower from TRI (via R35 10k) drives D2 → R4 → LED_K. It pulses at the LFO rate, is off in bypass, and runs from +9V (not VA). Use a red / high-efficiency LED (a blue one barely lights from a ~1.4–5.4 V emitter swing).
- **Parts:** C14 22p (C1804); Q4 C2910191 (Preferred Extended); R13 220k + R34 47k; R17 2M2 + R33 2M2; RV4 and C3 (now a THT radial) hand-soldered (excluded from BOM/CPL).
- **Pot direction:** fixed in the layout (9 mm RD901F on B.Cu with the verified orientation, as on Hyper Fuzz v1.1), not in the schematic.
- ERC 0. Next: user review → PCB (1590B, 9 mm pots, toggles, bigger pads/header, rotation offsets).

## v2.1 (from v2.0 unit testing, 2026-10-12)

### Problem

On the v2.0 unit (Unit #2), RV4's best point was at its end stop: 20 mV of bleed at LFO rate, and 50 mV, clearly audible, at audio rate.
- The v2.0 trim (R26 18k + RV4 5k = Rt 18–23k, Q2B collector to VB) only covers about ±4% of imbalance, and only adds load on one side.
- This unit needed ~7% (≈1.75 mV of Vbe mismatch in the MBT3904DW1, which isn't a matched pair, plus resistor tolerance).
- **Bench patch (unit-specific):** 100k from FB to VB moved the null back to the centre; bleed ~2.5 mV.
  - It is **not** a fix for every board: a unit mismatched the other way would need the opposite patch (~100k piggybacked on R26).
  - Details are in `docs/Test Procedure v2.0.md` §5.

### Plan: symmetric collector-load trim (classic LM3086 arrangement)

```
          VA
          │
     RV4 2k trimmer (wiper → VA)
      end A        end B
        │            │
     R24 9k1      R25 9k1
        │            │
   Q2A C (FB)    Q2B C ── R26 20k ── VB   (fixed: mirrors R29 20k on the FB side)
```

- **Changes:**
  - R24/R25 10k → **9k1** each, to the two ends of **RV4 2k** (3314J or 3362P), wiper to VA. At centre each side sees ~10.1k, the same as now.
  - **R26 18k → 20k fixed** to VB (it mirrors R29). The old one-sided RV4 path is removed.
- **Balance condition:** (Q2A current change) × (R24eff ∥ R29) = (Q2B current change) × (R25eff ∥ R26). A perfectly matched unit nulls with RV4 centred.
- **Range:** turning RV4 moves load from one side to the other. That's about **±13%** around the centre, so it covers this unit's 7% with margin, in **either direction**.
- **Trim procedure (unchanged):** no input, DEPTH max, RATE slow, OUT at 10 mV/div AC, turn RV4 for minimum. Then check the AUDIO range at the same setting.
- No FB–VB patch footprint is needed.
- **Trimmer: a multi-turn Bourns 3296W 2k (top adjust, THT), hand-soldered** and excluded from the BOM/CPL, like C3. (Decided 2026-10-13.)
  - Why: the wider range makes a single-turn too touchy. The bleed changes ~12 mV per 1% of balance. Getting to ~2.5 mV needs ~0.2%:
    - single-turn (~260° for ±13%): about 2° of rotation;
    - 25-turn 3296W: about 70°.
  - Multi-turn cermet also holds its setting better through boxing and knocks.
  - Footprint: `Potentiometer_THT:Potentiometer_Bourns_3296W_Vertical`. Put the adjust screw where it can be reached with the board in the box if possible. Check Tayda for 2k (code 202).

### Option: matched pair for Q2 (checked 2026-10-13)

- **DMMT3904W-7-F (Diodes Inc), LCSC C155305**, SOT-363.
  - Matched pair (adjacent die from the same wafer). **Vbe1 − Vbe2: 1 mV typ, 2 mV max**; hFE match within 2%; both at 2 mA / 5 V.
  - ~6,000 in stock, ~$0.13 each at 50.
  - Extended part, but MBT3904DW1 (C110150) is extended too. Using it for both Q2 and Q3 keeps it to one line.
- **What it buys:**
  - Typical mismatch ~1 mV ≈ 4% current split, so most units would null near the centre.
  - The 2 mV worst case is ~8%, as bad as this unit. **So it reduces the spread but doesn't remove the need for the trim.**
  - Only Q2 matters for bleed (Q3's offset is inside the output loop and only shifts DC).
- **Before switching, check:**
  - The pin-out against the footprint. The board uses the MBT3904DW1 order: **1 E1, 2 B1, 3 C2, 4 E2, 5 B2, 6 C1**. The DMMT3904W datasheet pin-out is a drawing I couldn't read; it's very likely the same standard SOT-363 dual order, but confirm it from Diodes' DS30311 "Device Schematic and Pin-Out".
  - The CPL rotation in JLC's viewer (Q2/Q3 needed 180° in v1).
- **Recommendation:** do the symmetric trim (required). Add the DMMT3904W as a cheap improvement if the pin-out checks out.
