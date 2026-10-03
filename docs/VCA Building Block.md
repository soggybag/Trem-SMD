# Discrete VCA building block (from Trem SMD)

The Trem's VCA is a discrete version of the classic **LM3086 transistor-array VCA**. Bench-proven on Trem SMD (2026-10-02): balanced, low control bleed (a few mV after trimming), and it sounds good with a guitar. Kept here as a reusable block.

## Circuit (Trem SMD refs)

```
            VA                      VA
            │                       │
           R24 10k                 R25 10k ──┬── R26 18k ── RV4 5k (wiper→VB)   balance / bleed trim
            │                       │        │
  FB ◄──────┤ Q2A C1        Q2B C2  ├────────┴──► Q3B base
  (Q3A base)│                       │
  VB─R23 1k─┤B1  Q2 (MBT3904DW1) B2 ├─R22 1k─VB
            │                       ├─R21 47k── audio in (÷48 → a few mV at the base)
            └──── E1 ──── E2 ───────┘
                      │ TAIL
                     R14 8k2
                      │
                 control voltage (U1A out: LFO, inverted, offset by R13 270k)
                 I_tail = (VB − Vbe − Vctrl) / 8k2 ≈ 0–0.3 mA

  Diff → single-ended output amp:
  Q3 pair (MBT3904DW1, tail R27 22k to GND) compares FB vs Q2B collector →
  Q4 MMBT3906 (PNP, C14 10p Miller) → output node (R28 3k3 to GND) → R29 20k back to FB.
  Output idles ~4.3–4.75 V on VA ≈ 8.6 V; C15 1µ couples it out.
```

## How it works

- **Gain ∝ tail current.** A differential pair's transconductance is gm = I_tail / (2 × 26 mV). The audio on one base becomes a collector current proportional to I_tail, so the control voltage multiplies the signal.
- **The signal must stay tiny** (a few mV at the base) because the pair is only linear for differential inputs well under ~±20 mV (the tanh curve). Hence R21/R22 ÷48 in front, and gain afterwards.
- **Control bleed:** changing I_tail shifts both collectors' DC level. Matched loads make it common-mode, and the diff-to-SE stage rejects it. RV4 trims the residual mismatch.
- **Full cut:** when the control voltage rises above VB − Vbe (≈ 3.75 V), the tail current is zero and the output is silent.

## Measured (Trem SMD, 2026-10-02)

| | |
|---|---|
| Tail current at idle | 0.13 mA (1.06 V across R14) |
| Q2 collectors | 6.88 / 6.78 V |
| Q3 tail | 0.28 mA (6.26 V / 22k) |
| Q4 | 1.33 mA; output node 4.75 V |
| Control bleed at OUT | a few mV p-p, nulled with RV4; similar in SQ mode |

## Strengths and limits

| Strengths | Limits |
|---|---|
| All SOT-23/SOT-363 parts, JLC-assemblable, no specialty IC (no obsolescence risk) | **Control range** ~30–40 dB (fine for tremolo), vs 80 dB+ for an OTA/VCA chip |
| Smooth, "tube-like" soft compression from the tanh curve | Needs ÷48 attenuation and makeup gain, so **noise** is higher than an IC VCA |
| Simple, tunable (tail resistor sets the range; RV4 sets balance) | **Bleed needs a trim** (RV4); temperature drift of Vbe moves the operating point |
| Proven in this pedal | MBT3904DW1 is two transistors in one package: better matched than discretes, but not a monolithic matched pair (LM394/THAT) |

## Could it replace other VCAs?

### Penfold Fuzz: LM13700 (envelope-controlled VCA)

**Possible, but I'd keep the LM13700 there.**
- The Penfold roadmap uses **both** OTAs (VCA + envelope filter or clean/fuzz blend) **and** the Darlington buffers (envelope hold-cap buffer). One LM13700 does all of that; the discrete version would need 2–3 transistor packages per function.
- **Envelope control is fast** (attack in ms). Control bleed turns into a *thump on every pick attack*, which is more audible than at a slow LFO. The LM13700 also bleeds, but it's easier to null and has a wider control range for a dynamic-fuzz "swell to silence".
- Worth it only if the LM13700 becomes hard to source.

### Tremolo-Matic X: NE570 (two-channel crossfade)

**Viable and interesting, especially as the NE570 ages out.**
- **Straight swap:** two copies of this VCA (2 × MBT3904DW1 + output stage each), with antiphase control from the existing U5A/U5B. That's more parts than one NE570, but all basic SMD.
- **Neater:** a **current-steering crossfade.** Feed the audio in as the *tail* current (one transistor transconductor), apply the LFO *differentially* across the pair's bases, and the two collectors give the signal crossfaded between two outputs (one per band). That's one pair for both channels, and a natural fit for the harmonic tremolo (and for the dry↔loop crossfade idea).
- Trade-offs: the NE570 gain cells are cleaner (lower distortion and noise, wider range). For a tremolo, the discrete pair's soft saturation is arguably a feature.
- Recommended path: breadboard the current-steering crossfader before committing a PCB.

### Other uses

Frobnicator (audio-rate trem: the pair is fast enough), the Ugly Face 4 "Dynamic" VCA, any LFO tremolo or envelope swell where a chip VCA is overkill.
