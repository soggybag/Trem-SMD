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
| 0.1 | Visual: U1 pin 1, **U2 pin 1 (CD4053)**, Q1–Q5, D1, D3/D4 bands, **C3 polarity (THT)**, both pot mounting tabs soldered | as silkscreen | |
| 0.2 | Ohms J1 → GND (slow: C3 470µ) | tens of kΩ | |
| 0.3 | **Ohms U1 pin 8 → pin 7** (the v1 failure) | kilohms, **not ~0 Ω** | |

## 1. First power (100 Ω; knobs at noon; RANGE on LFO; no input)

| # | Check | Expected (v1.0 measured) | Measured |
|---|---|---|---|
| 1.1 | LED off (J9 LED open) | ~3.2 mA (the CD4053 adds µA) | |
| 1.2 | LED on (J9 LED to G) | **pulsing**: +0 to ~3.5 mA with the LFO (red LED) | |

## 2. DC voltages (no input; DEPTH at 0)

| # | Node | Expected (v1.0 measured) | Measured |
|---|---|---|---|
| 2.1 | VA (C3 +) | ~8.6 V | |
| 2.2 | VB (U1 pin 7) | 4.4 V | |
| 2.3 | **U1 pin 1, DEPTH 0** | **≈ 2.75 V, flat** (4.4 − 100k/267k × 4.4) | |
| 2.4 | TAIL (Q2 pins 1/4) | ~3.8 V | |
| 2.5 | Q2 collectors (pins 6/3) | ~6.8–6.9 V, equal within ~0.1 V | |
| 2.6 | Output node (Q4 pin 3) | ~4.3–4.8 V | |
| 2.7 | U2 pins 9/10/11 (control), RANGE each way | +9V in one position, 0 V in the other | |

## 3. LFO and RANGE (scope on U1 pin 14 TRI and pin 8 SQR)

| # | Check | Expected | Measured |
|---|---|---|---|
| 3.1 | **Which SW2 position is LFO?** The one where U2 pins 9–11 are **high** connects the 470n | label it | |
| 3.2 | LFO: RATE min → max | ~2 Hz → ~17 Hz (as v1). **RATE now faster clockwise** (v1 was backwards) | |
| 3.3 | **AUDIO: RATE min → max** | **~100 Hz → ~800 Hz** (48× the LFO) | |
| 3.4 | TRI / SQR levels (both ranges) | TRI ~2–6 V, SQR ~1–8 V | |
| 3.5 | SHAPE ends (both ranges) | ramp up ↔ ramp down; triangle at noon | |
| 3.6 | U1 pin 1, DEPTH max, LFO range | inverted LFO ~±1.3 V around 2.75 V. **DEPTH max now clockwise** | |
| 3.7 | U1 pin 1, DEPTH max, AUDIO range | same shape, edges barely smoothed (C8 220p only) | |

## 4. LFO LED

| # | Check | Expected | Seen |
|---|---|---|---|
| 4.1 | LED_K grounded (footswitch on), LFO range | LED **pulses** with the triangle: brightest at the TRI peak | |
| 4.2 | AUDIO range | looks steady (too fast to see), roughly half brightness | |
| 4.3 | LED_K open (bypassed) | off | |

## 5. Balance trim (RV4) and audio

1. **Null in the LFO range** as on v1: no input, DEPTH max, RATE slow, scope on OUT at 10 mV/div AC. Turn RV4 for the smallest LFO bleed.
2. **Then check the AUDIO range at the same RV4 setting.** Any bleed is now an audible tone (100–800 Hz), not a thump. Re-trim for the best compromise; note both.

| # | Check | Measured |
|---|---|---|
| 5.1 | Bleed LFO range after trimming (mVpp) | |
| 5.2 | Bleed AUDIO range at the same setting | |
| 5.3 | Guitar, LFO range: smooth trem, chop with SQ | |
| 5.4 | Guitar, AUDIO range: ring-mod / AM tones; note the sound at low and high RATE | |
| 5.5 | Any clicks on the RANGE switch | |

## 6. After boxing

- U1 pin 8 → pin 7 ohms again (power off): kilohms.
- Supply current (1.1).

## Record

| Item | Result |
|---|---|
| Unit number | |
| Date | |
| Notes | |
