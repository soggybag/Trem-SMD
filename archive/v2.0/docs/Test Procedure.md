# Trem SMD: test procedure

Board: `Trem_23` (TL064 LFO + discrete LM3086-style VCA, 1590B). Refs and values are from the KiCad schematic (`Trem_23_schematic.pdf`).
Expected values are hand calculations at **+9V = 8.8 V**. Where Ugly Face testing taught us something, it's noted as **UF:**.

## How it works (block summary)

| Block | Parts | What it does |
|---|---|---|
| Power | D1 SS14, C1 10µ; **R1 100 Ω + C3 470µ → VA** | +9V for the TL064; VA = a filtered supply for the audio section |
| Bias | R2/R3 100k/100k + C5 → **U1B follower → VB ≈ 4.4 V** | Buffered half-supply for the op-amps and the VCA bases |
| LFO | **U1D integrator** (C6 470n, R5 6k8) + **U1C Schmitt** (R6 10k / R7 22k, R8 68k + C7 33n) | Triangle (TRI, pin 14) and square (SQR, pin 8) |
| RATE / SHAPE | RV1 A10k + R9 2k2 sets the drive level; D3/D4 + RV2 B100k steer rise vs fall | Speed, and skew (ramp up / ramp down) |
| SQ/TRI | SW1 picks TRI directly, or SQR through R10 100k | Waveform |
| DEPTH → control | RV3 100k → R11 100k → **U1A inverting** (R12 100k, C8 10n smoothing, R13 270k offset) → R14 8k2 → VCA tail | Turns the LFO into the VCA tail current |
| Preamp | **Q1 MMBT5089** (R17 4M7 collector-base bias, R18 4k7, R19 4k7, R20 200 + C13 100n) | Input buffer/gain |
| VCA | **Q2 pair** (signal ÷48 via R21 47k / R22 1k; R24/R25 10k loads; RV4 balance trim via R26 18k) | The tail current sets the gain: the tremolo |
| Output amp | **Q3 pair + Q4 PNP**, R29 20k feedback, R28 3k3 load | Diff-to-single-ended; drives C15 → R30/R31/C16/R32 → OUT |

## Before you start

- **SW1 is fitted** (sub-mini SPDT, 2026-10-02). Pots: RV1 RATE A10k, RV2 SHAPE B100k, **RV3 DEPTH B100k** (taper now in the schematic/BOM).
- **UF lessons:**
  - DC coupling on the scope.
  - Time/DIV clicked into a detent.
  - Probe ×1/×10 matched.
  - With the 100 Ω method, move the decimal two places for mA.
  - A multimeter shows the **average** of a waveform, so use the scope for anything that moves.
  - A 10 MΩ meter loads high-resistance nodes; it reads low on anything fed through ~1M.
- Pin maps:
  - **Q2/Q3 (SOT-363):** 1 = E1, 2 = B1, 3 = C2, 4 = E2, 5 = B2, 6 = C1.
  - **Q1/Q4 (SOT-23):** 1 = B, 2 = E, 3 = C. **Physical:** looking at the top with the single pin pointing up, pin 1 is bottom-left, pin 2 bottom-right, pin 3 the lone top pin. (Not the TO-92 E-B-C habit!)
  - **SOT-363 (Q2/Q3):** pin 1 is marked by a dot or bevel. Count 1-2-3 down one side, then 4-5-6 back up the other (counter-clockwise from the top, like an IC).
  - **U1 TL064:** 4 = V+, 11 = GND.

---

## 0. Before power

| # | Check | Expected | Measured | OK |
|---|---|---|---|---|
| 0.1 | Visual: U1 pin 1, Q1–Q4 orientation, D1 band, D3/D4 bands, C3 polarity, D2 | as silkscreen | - | x |
| 0.2 | Ohms J1 (9V) → GND, wait to settle (C3 470µ is slow) | climbs; **fail: a few Ω** | 52.5K measured from J1 to J2 | x |
| 0.3 | Diode test, red on J1, black on D1 cathode | 0.15–0.3 V | fluctuates between 300 and 500mV (meter is charging C1/C3 through D1; see the powered check in 2.1) | x |

## 1. First power (100 Ω in series; knobs at noon; RV4 mid; no input)

| # | Check | Expected | Measured (V across 100 Ω) | mA | OK |
|---|---|---|---|---|---|
| 1.1 | LED off (J7 open) | **~3–4 mA** (TL064 ~0.8 + audio section ~2.5) | 0.32V | 3.2 mA | x |
| 1.2 | LED on (J7 to GND) | **+~6.8 mA** (R4 1k, red LED) | 0.79V | 7.9 mA, LED +4.7 mA (I used a random blue violet LED, Vf ≈ 3 V) | x |

Unplug above 2 V (20 mA). **Remove the 100 Ω before section 2.**

## 2. DC voltages (no input; DEPTH at 0 unless noted; black probe on GND)

**Ohm's-law exercise:** each "current" row comes from a voltage across a known resistor (I = V / R).

| # | Node | Where | Expected | Measured |
|---|---|---|---|---|
| 2.1 | +9V | D1 cathode / U1 pin 4 | supply − 0.2–0.3 V (also measure J1: J1 − +9V = D1's real forward drop) | 0.29V from J1 to Pin 4 |
| 2.2 | **VA** | C3 + | **~8.5 V** (+9V minus ~2.5 mA × 100 Ω) | 8.59V |
| 2.3 | **VB** | U1 pin 7 (= pins 5, 6, 3, 12) | **4.4 V** | 4.42V |
| 2.4 | TRI / SQR average | U1 pin 14 / pin 8 | ~4.4 V (moving: use the scope, section 3) | 4.55V |
| 2.5 | **U1 pin 1 (U1A out), DEPTH 0** | | **≈ 2.8 V**: VB − (100k/270k) × (8.8 − 4.4) = 4.4 − 1.6 | 2.18V |
| 2.6 | **TAIL** (Q2 pins 1/4) | | **≈ 3.75 V** (VB − one Vbe) | 3.8V |
| 2.7 | Tail current | V across R14 (TAIL − U1 pin 1) ÷ 8.2k | ≈ 1 V → **~0.12 mA** | 1.06V |
| 2.8 | Q1 base / emitter / collector | Q1 pins 1 / 2 / 3 | **~2.4–2.8 / 1.8–2.2 / 6.3–6.8 V** (depends on hFE) | 2.13V e / 2.45V b / 6.47V c |
| 2.9 | Q1 current | (VA − Vc) / 4.7k | ~0.4–0.45 mA | (8.59 − 6.47) / 4.7k = **0.45 mA**; Ve 2.13 / 4.7k = 0.45 mA ✓ |
| 2.10 | **Q2 C1 (FB) and Q2 C2** | Q2 pin 6 / pin 3 | **≈ 6.5–7.5 V, and equal to each other within tens of mV** (the output amp's feedback balances Q3's bases) | 6.88V / 6.78V |
| 2.11 | Q3 emitters | Q3 pins 1/4 | row 2.10's value − 0.65 V (≈ 6–7 V) | 6.26V / 6.26V |
| 2.12 | Q4 base (DRV) / emitter | Q4 pin 1 / pin 2 | VA − 0.65 / VA | 7.94V / 8.6V |
| 2.13 | **Output node** | Q4 pin 3 (C15 + side) | **≈ 4.3 V** (3.5–5.5 V depending on RV4) | 4.75V |
| 2.14 | Q4 current | V(Q4 collector) / 3.3k + ~0.1 mA | ~1.3–1.5 mA | 4.75 / 3.3k − (6.88 − 4.75) / 20k = **1.33 mA** ✓ |
| 2.15 | OUT pad (J5) | - | 0 V | x |

## 3. LFO (scope; SW1 on TRI)

**Predictions:**
- **Triangle size is set by the Schmitt resistors:** TRI peak-to-peak = (R6 / R7) × SQR peak-to-peak = **(10k / 22k) × SQR ≈ 0.45 × SQR**. For example, a 6 Vpp square gives a ~2.7 Vpp triangle, both centred on 4.4 V.
- **Rate:** the integrator ramps at I / C6, where I = (RATE wiper voltage − one diode drop) / (6.8k + SHAPE section).
  - At RATE max with SHAPE at noon: about **15–20 Hz**.
  - At RATE min the drive falls below the diode drop, so it slows to **under ~0.5 Hz** (or stalls).
  - The diodes make the low end very non-linear.

| # | Check | Expected | Measured |
|---|---|---|---|
| 3.1 | SQR (U1 pin 8): high / low levels, Vpp | ~1.5 ↔ 7.3 V (TL064 swing) | Looks like 3.5V (maybe 0.5V to 4.0V)|
| 3.2 | TRI (U1 pin 14): Vpp; **ratio TRI / SQR** | **≈ 0.45** | 2V (1V to 3V)|
| 3.3 | Rate, RATE max, SHAPE noon (period → Hz) | ~15–20 Hz | Fastest about 60mS (I can’t read the slow rate on the meter)|
| 3.4 | Rate, RATE min, SHAPE noon | < 0.5 Hz or stalls | Doesn’t stall. Feels like 1/2 Sec. |
| 3.5 | SHAPE at each end (TRI shape) | ramp up (fast fall) ↔ ramp down (fast rise); period changes too | Shape CCW Ramp up, CW Ramp down, (soft) triangle in the center |
| 3.6 | U1 pin 1, DEPTH max | inverted LFO, ~±1.2–1.4 V around 2.8 V (≈ 1.4 ↔ 4.2 V) | 2V triangle (1V to 3V) |
| 3.7 | SW1 on SQ, U1 pin 1, DEPTH max | square, about half the TRI swing (R10 100k into the 100k DEPTH pot), edges rounded by C8 | ~4V soft square|

**What the tail is doing:** the tail current = (3.75 V − U1 pin 1) / 8.2k. When U1 pin 1 rises above ~3.75 V the pair shuts off, so **full cut at the LFO peak** with DEPTH high.

## 4. Balance trim (RV4): null the LFO thump

**What this measures:** with no guitar signal, an ideal tremolo outputs *nothing*. But the LFO swings the VCA's tail current, and that shifts the DC level at Q2's collectors. If the two collector loads aren't perfectly matched, the shift reaches the output as a slow wave at the LFO rate. You hear it as a "thump" or "tick", especially in SQ mode. RV4 (via R26) trims one collector load so the shift cancels.

1. No input. DEPTH max (fully **CCW** on this board), RATE slow (~2 Hz). Scope (or audio probe) on OUT, AC coupled. Start at 50 mV/div and go down to 10 mV/div.
2. You'll see the LFO itself bleeding through as a slow wave: the thump.
3. Adjust RV4 for the **smallest** bleed. Note the residual Vpp.\
\
NOTE: Adjusting RV4 doesn’t have any effect on the output. The output reads flat. I read a small ripple with the scope set to 10mV and was able to dial that out. 

| # | | Measured |
|---|---|---|
| 4.1 | Bleed before / after nulling (mVpp) | flat at coarse scales; a small ripple at 10 mV/div, dialled out with RV4 ✓ |
| 4.2 | RV4 position at the null | |

## 5. Audio path (signal generator ~100 mVpp sine into IN; DEPTH 0 first)

| # | Point | Expected | Seen |
|---|---|---|---|
| 5.1 | Q1 collector, **200 Hz** | gain ≈ **1.3** (C13 100n / R20 200 hardly bypasses R19 at low frequency) | |
| 5.2 | Q1 collector, **2 kHz** | gain ≈ **4–5**: **the preamp boosts treble by design** (the same 100n/200 Ω as the original Trem 23). Confirm | |
| 5.3 | Q2 pin 5 (signal base) | IN × Q1 gain ÷ 48 (R21 / R22): a few mV | |
| 5.4 | OUT, DEPTH 0 | steady sine; note the overall gain IN → OUT | |
| 5.5 | OUT, DEPTH max, TRI | smooth tremolo, dipping to ~silence at the LFO peaks | |
| 5.6 | OUT, DEPTH max, SQ | choppy on/off; clicks? (C8 smooths the edges) | |
| 5.7 | Large input (1 Vpp) at DEPTH 0 | where does OUT start to clip? (the Q2 pair is small-signal) | |

## 6. Audio probe trace

IN → Q1 collector → Q2 pin 5 (quiet) → Q2 pins 6/3 (VCA out) → Q4 collector → OUT.

## Notes / surprises

- **Section 1 (2026-10-02):** 3.2 mA idle (predicted 3–4); the LED adds 4.7 mA (blue/violet LED, Vf ≈ 3 V).
- **Section 2 (2026-10-02):** D1 drop 0.29 V; VA 8.59; VB 4.42; tail current 1.06 V / 8.2k = 0.13 mA ✓; Q1 0.45 mA ✓ (hFE ≈ 560 from the R17 current); Q2 collectors 6.88 / 6.78 V (balanced within 0.1 V) ✓; output node 4.75 V ✓; Q4 1.33 mA ✓.
- **2.5 U1 pin 1 = 2.18 V vs 2.8 V predicted, but 2.6 − 2.7 implies 3.8 − 1.06 = 2.74 V.** The readings disagree, so pin 1 was probably moving: the LFO was getting through (DEPTH not fully at the VB end?).
  - **The pots are on the back, so the knob direction may be reversed** relative to the lug numbers.
  - Check on the scope in 3.6: at true DEPTH 0, pin 1 is a flat ~2.8 V.
- **2.8 Q1 base reads 2.45 V, only 0.32 V above the emitter (impossible for a conducting transistor): meter loading.**
  - The base is fed through R17 4.7M and looks like ~2M into the transistor. A 10 MΩ meter pulls it down while you measure.
  - True Vb ≈ Ve + 0.6 ≈ 2.7 V.
- **Pin-reading mix-ups (SMD vs TO-92 habits), resolved:**
  - **Q3 emitters re-measured: 6.26 V** ✓ (predicted ≈ 6.2). Q3 tail = 6.26 / 22k = **0.28 mA** ✓.
  - **Q4:** read with the correct SOT-23 pinout, the emitter is VA (8.6 V) and the base 7.94 V (0.65 V lower) ✓.
  - **Q1:** re-check the emitter on pin 2 (expect ≈ 0.45 mA × 4.7k ≈ 2.1 V).
  - **3.0 V across R17 (4.7M):** base current ≈ 0.6–0.9 µA (the meter's 10 MΩ in parallel skews it), so **hFE ≈ 500–700**, normal for an MMBT5089.
- **Section 3 (2026-10-02): the LFO works.**
  - **Rate ≈ 2 Hz (~0.5 s) to ~17 Hz (60 ms).** It doesn't stall at the slow end; 1N4148s still conduct at the tiny currents needed, so my "< 0.5 Hz / stall" guess was too pessimistic.
  - SHAPE: CCW ramp up, CW ramp down, triangle at noon ✓.
  - U1 pin 1 at DEPTH max: 2 Vpp triangle (= TRI swing × R12/R11 = 1) ✓.
  - **RATE and DEPTH turn backwards** (RATE fast at CCW; DEPTH max at CCW). So section 2's "DEPTH 0" was really DEPTH max, which explains 2.5 = 2.18 V (the average of a moving signal). Re-measure 2.5 with DEPTH fully **CW**: expect a flat ~2.8 V.
  - **Amplitude readings don't hang together:**
    - SQR 0.5–4.0 V and TRI 1–3 V should both be centred on VB 4.4 V (the DMM average was 4.55 V).
    - TRI/SQR = 0.57 (expected 0.45).
    - The SQ depth signal (~4 V) is bigger than the TRI (2 V), but the R10 path should make it *smaller*.
    - Probably a scope zero/scale reading issue. Check the scope:
      1. Set the input to GND coupling and put the trace on a gridline; that's 0 V.
      2. Probe the scope's calibrator ("CAL"/probe-comp) output to confirm V/div and the ×1/×10 setting.
      3. Re-read SQR and TRI.
- **TRI re-measured (DC coupling, zero line set): +2 V to +6 V = 4 Vpp centred on ~4 V** ✓. To do: re-measure SQR the same way. TRI/SQR = 0.45 predicts SQR ≈ 8.9 Vpp, which is more than the supply, so expect the ratio to come out higher (~0.55–0.6). C7's dynamic hysteresis is a possible reason.
- **Section 4:** the LFO bleed at OUT is only a few mV (invisible except at 10 mV/div). RV4 nulled it. Small bleed is a good result. To do: check SQ mode (steps at the square edges = clicks) the same way.
- **SQR re-measured: 1 V to 8 V (7 Vpp).** TRI/SQR = 4 / 7 = **0.57** (simple R6/R7 prediction 0.45). The extra is most likely C7 (33n on the Schmitt's − input) adding dynamic hysteresis. The TL064 swings to within ~1 V of each rail.
- **SQ-mode bleed:** about the same as TRI after trimming RV4 (very small) ✓.
- **Sections 5–6 skipped; tried with a guitar: sounds really good** (2026-10-02).

