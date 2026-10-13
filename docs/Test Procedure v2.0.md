# Trem SMD v2.0: test procedure

Board: `Trem_23` **v2.0** (git tag `v2.0`; `archive/v2.0/` has the schematic PDF and the interactive BOM). TL064 LFO + discrete VCA + CD4053, 1590B.
General method: `../../_tools/BENCH_TESTING.md`. v1.0's full procedure and results (`Test Procedure.md`) still apply to the unchanged audio path. This sheet covers what's new, plus a short re-run of the key v1 checks.

## What's new in v2.0

| Change | Parts | Check |
|---|---|---|
| **RANGE switch: LFO / AUDIO** | SW2 drives U2 CD4053 control pins 9/10/11. One position switches in C17 470n (integrator), C18 33n (Schmitt) and C19 10n (depth smoothing) for LFO rates. Without them only C6 10n / C7 1n / C8 220p remain: **~48× faster** | Section 3 |
| **LFO LED** | Q5 MMBT3904 emitter follower from TRI (R35 10k) drives D2 → R4 1k → LED_K | Section 4 |
| 9 mm pots on the back, turning the right way | RV1 RATE A10k, RV2 SHAPE B100k, RV3 DEPTH B100k | Section 3 |
| 1×4 header J9 (IN / LED / G / OUT) | | Section 0 |
| RV4 balance trim → Bourns 3314J 5k (SMD); C3 470µ THT hand-soldered; Q4 → MMBT5401 | | Sections 1–2, 5 |
| DEPTH offset network | R13 220k + R34 47k to +9V (was 270k) | Section 2 |

## 0. Before power

| # | Check | Expected | Measured |
|---|---|---|---|
| 0.1 | Visual: U1 pin 1, **U2 pin 1 (CD4053)**, Q1–Q5, D1, D3/D4 bands, **C3 polarity (THT)**, both pot mounting tabs soldered | as silkscreen | ✅ |
| 0.2 | Ohms J1 → GND (slow: C3 470µ) | tens of kΩ | 54.5K |
| 0.3 | **Ohms U1 pin 8 → pin 7** (the v1 failure) | kilohms, **not ~0 Ω** | 10K |

## 1. First power (100 Ω; knobs at noon; RANGE on LFO; no input)

| # | Check | Expected (v1.0 measured) | Measured |
|---|---|---|---|
| 1.1 | LED off (J9 LED open) | ~3.2 mA (the CD4053 adds µA) | 0.13V across 100ohm resistor = 3.3mA|
| 1.2 | LED on (J9 LED to G) | **pulsing**: +0 to ~3.5 mA with the LFO (red LED) | 0.45V = 4.5mA with LED on, I’m using a 3mm red. |

## 2. DC voltages (no input; DEPTH at 0)

| # | Node | Expected (v1.0 measured) | Measured |
|---|---|---|---|
| 2.1 | VA (C3 +) | ~8.6 V | 8.53V at R1 |
| 2.2 | VB (U1 pin 7) | 4.4 V | 4.26V |
| 2.3 | **U1 pin 1, DEPTH 0** | **≈ 2.75 V, flat** (4.4 − 100k/267k × 4.4) | Min 2.61V |
| 2.4 | TAIL (Q2 pins 1/4) | ~3.8 V | 3.66V |
| 2.5 | Q2 collectors (pins 6/3) | ~6.8–6.9 V, equal within ~0.1 V | 6.46V / 6.67V  |
| 2.6 | Output node (Q4 pin 3) | ~4.3–4.8 V | 4.66V |
| 2.7 | U2 pins 9/10/11 (control), RANGE each way | +9V in one position, 0 V in the other | 8.54V / 0V |

## 3. LFO and RANGE (scope on U1 pin 14 TRI and pin 8 SQR)

| # | Check | Expected | Measured |
|---|---|---|---|
| 3.1 | **Which SW2 position is LFO?** The one where U2 pins 9–11 are **high** connects the 470n | label it | SW2 slow rate. Not sure what you are asking |
| 3.2 | LFO: RATE min → max | ~2 Hz → ~17 Hz (as v1). **RATE now faster clockwise** (v1 was backwards) | Hard to measure about half a second to 60mS |
| 3.3 | **AUDIO: RATE min → max** | **~100 Hz → ~800 Hz** (48× the LFO) | 8mS to 1.5mS, I’m using an analog scope the numbers are hard to read. |
| 3.4 | TRI / SQR levels (both ranges) | TRI ~2–6 V, SQR ~1–8 V | Square 1.5V, Triangle 1.5V |
| 3.5 | SHAPE ends (both ranges) | ramp up ↔ ramp down; triangle at noon | Working |
| 3.6 | U1 pin 1, DEPTH max, LFO range | inverted LFO ~±1.3 V around 2.75 V. **DEPTH max now clockwise** | 0V to 1.0V |
| 3.7 | U1 pin 1, DEPTH max, AUDIO range | same shape, edges barely smoothed (C8 220p only) | 0V to 1.5V |

## 4. LFO LED

| # | Check | Expected | Seen |
|---|---|---|---|
| 4.1 | LED_K grounded (footswitch on), LFO range | LED **pulses** with the triangle: brightest at the TRI peak |  Yes|
| 4.2 | AUDIO range | looks steady (too fast to see), roughly half brightness | Yes |
| 4.3 | LED_K open (bypassed) | off | Yes |

**Sections 0–4 notes (2026-10-12):**
- **0–2 ✓.**
  - 0.3 = 10 kΩ: no SQR–VB short (the v1 failure).
  - Supply 3.3 mA; the LED adds ~1.2 mA on average (it's pulsing, so the meter averages it).
  - VB 4.26 V. DEPTH-0 offset 2.61 V (the 2.75 V prediction scaled to VB 4.26 V gives ~2.66 V).
  - 2.5: the collectors differ by 0.21 V (6.46 / 6.67 V). That's fine before RV4 is trimmed: section 5 balances them.
- **3.1:** the question is just which way the **lever** points for LFO (e.g. "toward the footswitch = LFO"), so the box can be labelled. The position with U2 pins 9–11 at 8.5 V should be the slow one.
- **3.2 ✓** 0.5 s → 60 ms period = **2 → 17 Hz** (Hz = 1000 / ms), as designed.
- **3.3 ✓** 8 → 1.5 ms = **125 → 670 Hz**. That's 40–60× the LFO, in line with the predicted 48×.
- **3.4, 3.6, 3.7: the levels read small; re-measure with DC coupling.** This is the same thing that happened on v1:
  - v1's first reading was SQR 3.5 V and TRI 2 V. With DC coupling and the zero line set, it was **SQR 1 → 8 V, TRI 2 → 6 V**.
  - Analog scope steps: input coupling **GND**, put the trace on a graticule line (that's 0 V), switch to **DC**, check the probe ×1/×10 matches the V/div you're reading, then read the top and bottom.
  - Expected: SQR ~1 → 8 V; TRI ~2 → 6 V; U1 pin 1 at DEPTH max (now **clockwise**) ~1.4 → 4.1 V, centred near 2.7 V. "0 V to 1.0 V" can't be a DC reading here, because pin 1 sits at 2.6 V even with DEPTH at 0.
  - The frequencies being right and SHAPE/LED working say the LFO itself is fine.
- **4 ✓** LED pulses on LFO, steady on AUDIO, off when bypassed.

## 5. Balance trim (RV4) and audio

1. **Null in the LFO range** as on v1: no input, DEPTH max, RATE slow, scope on OUT at 10 mV/div AC. Turn RV4 for the smallest LFO bleed.
2. **Then check the AUDIO range at the same RV4 setting.** Any bleed is now an audible tone (100–800 Hz), not a thump. Re-trim for the best compromise; note both.

| # | Check | Measured |
|---|---|---|
| 5.1 | Bleed LFO range after trimming (mVpp) | Seems like there is significant bleed through with this version. At audio rate I hear a steady 50mV wave form at the output. |
| 5.2 | Bleed AUDIO range at the same setting | I was unable to trim this out. At LFO range it isn’t audible. |
| 5.3 | Guitar, LFO range: smooth trem, chop with SQ | yes |
| 5.4 | Guitar, AUDIO range: ring-mod / AM tones; note the sound at low and high RATE | The AM tones are not bad. The bleed through is sort of a deal breaker. Otherwise the sound is good. |
| 5.5 | Any clicks on the RANGE switch | No |

**Section 5 notes (2026-10-12):**
- **The LFO range is good:** smooth trem, SQ chop, no audible bleed, and no clicks on RANGE.
- **The AUDIO range has a ~50 mV steady tone at OUT that RV4 can't trim out.** That's the problem to solve.
- **Why it's suspicious:** RV4 nulled v1's bleed to a few mV, and it should null the VCA's own control bleed (a load mismatch) at any frequency. If it nulls at LFO rates but not at audio rates, the tone is probably reaching OUT **some other way**, not through the VCA's balance. 

- I see 50mV tri or sqr at audio rate. Turning the trimmer clockwise brings this as low as it will get (50mV)
- At LFO rate I measure a 20mV wave, again the lowest I can get this is by turning the trimmer full clockwise. 

- **Candidate paths:**
  - **The LFO itself coupling in**: the SQR square (7 Vpp, sharp edges) via shared VB (the RATE network R9/RV1 and R8 hang on VB, and so do the VCA bases), via the supply, or capacitively on the board.
  - **The LED driver**: Q5 pulsing ~3.5 mA in the ground return through J9 and the footswitch wiring.
  - **The VCA's balance**, in a frequency-dependent way (collector and Miller capacitances).
- **Tests (scope on OUT, AC, AUDIO range, RATE ~mid, no input):**
  1. **Waveform shape:** a triangle-ish tone = control through the VCA; spikes at each edge or a square = SQR coupling. 

I see a triangle flat on the bottom, 20mV, about 3mS 

  2. **DEPTH to 0** (the VCA control goes flat, but the LFO keeps running). **If the tone stays, it isn't coming through the VCA control** → coupling (VB, supply, layout). If it goes, it's the VCA's balance.

With DEPTH 0 the tone is gone


  3. **RV4 end to end:** does the 50 mV change at all, and is there a dip anywhere?

With the control full CW is the best it gets. Before it nulled in the center. 

  4. **LED_K open vs grounded** (on the bench, LED_K is J9 pin 2).

Not sure what you are asking?

  5. **SQ vs TRI** (SW1), and **RATE low vs high**: does the level change with frequency?

The shape of the wave the sound change. The output follows the control Tri (flat on the bottom) and Sq (rounded)

**Diagnosis (2026-10-12): it IS the VCA balance; RV4's range is too narrow for this unit's transistors.**
- DEPTH 0 kills the tone, and the tone follows the control (TRI with a flat bottom = the VCA cutting fully off at the LFO peak; SQ rounded). So it comes through the VCA's control path, not from coupling or the LED. (My coupling guess was wrong.)
- **The best point is at RV4's end stop** (full CW), in both ranges: 20 mV at LFO rate, 50 mV at audio rate. v1 nulled in the middle. **The null is past the end of the trim range.**
- **How the trim works:**
  - RV4 (wiper to VB, pin 1 open) + R26 18k = **Rt, 18–23 kΩ**, from Q2B's collector to VB, in parallel with R25 10k.
  - The output-stage feedback (R29 20k, R24 10k) makes the control bleed at OUT ∝ **R29 − (1 + R29/R24) × (R25 ∥ Rt) = 20k − 3 × (R25 ∥ Rt)**.
  - So the bleed is zero at R25 ∥ Rt = 6.67k, i.e. **Rt = 20k** for perfectly matched parts, inside the 18–23k range.
- **Why this unit can't reach it:**
  - A few mV of Vbe mismatch in the Q2 dual (MBT3904DW1 isn't a monolithic matched pair) splits the tail current unevenly by a few %. Moving the null needs ~2.5 k per % of mismatch in Rt (it's diluted by R25 ∥).
  - 1% mismatch in R24/R25 does the same.
  - The range only covers about ±1%. The collectors' DC gap (0.21 V here vs 0.10 V on v1) points the same way.
- **Bench fix on this board:**
  1. **Find which way to go.** Power off, RV4 at full CW; ohms from RV4 **pin 2 (wiper) to pin 3**:
     - **~0 Ω** → full CW = Rt minimum (18k) → **the null needs Rt < 18k**.
     - **~5k** → full CW = Rt maximum (23k) → **the null needs Rt > 23k**.
  2. **Needs less Rt:** piggyback a resistor on top of R26: **100k** → R26 15.3k (range 15.3–20.3k). If it's still at the stop, use 47k (→ 13k).
  3. **Needs more Rt:** instead, add a resistor from **FB (R24 pin 2 / R29 pin 1) to VB**, starting at **100k**. This loads the other side the same way.
  - **Measured: wiper→pin 3 = 4.7k at full CW, 49 Ω at full CCW → full CW = Rt max (22.7k). The null needs Rt > 23k → step 3 (100k from FB to VB), not step 2.**
    - Why loading the *other* side works: the null condition is (Q2A current change) × (R24 ∥ R29 ∥ Rx) = (Q2B current change) × (R25 ∥ Rt). The required R25 ∥ Rt drops by the same % as the left side does: 100k takes it from 6.67k to 6.25k (−6%), against a trim span of about ±4%.
    - Solder points: FB = R24 pin 2 (or R29 pin 1); VB = RV4 pin 2 (the wiper) or R22/R23 pin 2.
    - If the best point then moves to the **CCW** stop, it's overcorrected: use 220k. If it's still at the CW stop: 47k.
  4. Re-trim. A good result is a dip **in the middle of the travel**, a few mV in the LFO range. Then check the AUDIO range at the same setting.
- **Result (2026-10-12): 100k fitted from FB to VB; re-trimmed. The null is now near the centre of RV4 (a little to one side), and the bleed is down to ~2.5 mV, from 50 mV.** The tone is much quieter. Unit #1 keeps this mod (log it in the Build Log).
- **v2.1 change:** a symmetric collector-load trim (R24/R25 9k1 to the ends of a 2k trimmer, wiper to VA; R26 20k fixed), about ±13% range in either direction. Optionally a matched DMMT3904W pair for Q2. See `Trem Roadmap.md` § v2.1.

## 6. After boxing

- U1 pin 8 → pin 7 ohms again (power off): kilohms.
- Supply current (1.1).

## Record

| Item | Result |
|---|---|
| Unit number | |
| Date | |
| Notes | |
