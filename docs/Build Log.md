# Trem SMD: build log

One entry per physical pedal. Write the unit number inside the enclosure.

## Unit #1: board v1.0 (git tag `v1.0`)

- **2026-10-02:** bench-tested (see `docs/Test Procedure.md` at tag v1.0). DC, LFO and VCA all OK; sounds good with a guitar.
- **After the bench test:** the pots were pulled and replaced (same positions), then the board was installed in the enclosure.
- **Fault:** engaged audio passed (a little quieter and brighter than bypass), but there was no tremolo. The controls did nothing. On the very first test, there was no engaged audio at all; that cleared after prodding the wiring.
- **Diagnosis (2026-10-06):**
  - **The LFO had stopped.** U1 pin 8 (SQR) read 4.59 V and pin 14 (TRI) 4.99 V, both static. VB was 4.58 V, up from 4.42 V on 10-02.
  - **A stalled LFO should sit at a rail** (~1 V or ~8 V). SQR parked on VB meant **SQR was shorted to VB**.
  - **The RATE reading agreed:** 1.7 kΩ across RV1 in-circuit is 10k in parallel with R9's 2k2.
  - **Ohms from U1 pin 8 to pin 7 read ≈ 0 Ω.**
- **Cause:** **RV1 RATE lug 3 (SQR) and RV3 DEPTH lug 1 (VB) were touching.** On v1.0 these two lugs are side by side, 3.2 mm apart, between the pots. The pot swap and boxing pushed them together.
- **Fix:** separated the lugs. Tremolo works again.
- **Also worth doing:** re-flow the small J3 (IN) and J5 (OUT) wire pads. They're the likely cause of the first "no audio" episode.
- **Lesson:** after any rework or boxing, measure **U1 pin 8 → pin 7** with the power off; it should read kilohms, not ~0 Ω. The v2 layout (9 mm pots) has no lugs from neighbouring pots within 5 mm of each other.
