# TalkBox — High-Power Talk Box PCB Project

> A from-scratch, no-compromise talk box design built to outperform the TalkStar.
> **~65 W clean Class D** into a **MIYAKO DU-100 compression driver** (16 Ω), powered by a **48 V** supply — delivering **133 dB SPL** (3× the TalkStar's power).

---

## Design Goals
| Feature | TalkStar | **TalkBox (Ours)** |
|---|---|---|
| Power Amplifier | 20 W RMS (Class AB) | **~65 W clean RMS (Class D, TPA3255) — 3.25× more power** |
| Supply | 24 V / 60 W brick | **48 V / ≥3 A supply (many options — see sourcing guide)** |
| Driver | Generic compression driver | **MIYAKO DU-100 (100 W, 16 Ω, 115 dB/W/m)** |
| Preamp | Single op-amp stage | **Low-noise NE5532 differential input** |
| High-Pass Filter | Fixed 150 Hz | **Switchable 100 Hz / 150 Hz / 200 Hz** |
| Edger (Clipping) | Soft clip, invert | **Soft clip + hard clip + clean modes, invert** |
| Tone Control | Tilt EQ | **Tilt EQ with broader sweep range** |
| LED Lighting | Signal-following | **Signal-following + manual on/off switch** |
| Bypass | Relay | **True-bypass DPDT relay + status LED** |
| Protection | Basic | **Reverse polarity, DC offset, overcurrent fuse, soft-start** |

---

## System Block Diagram

```
                                                    ┌─────────────┐     ┌──────────────┐
                                                    │  48V / 5A   │────▶│  100/240 VAC │
                                                    │  Power Pack │     │  Wall Outlet  │
                                                    └──────┬──────┘     └──────────────┘
                                                           │
                                              ┌────────────┼────────────────┐
                                              │            │                │
                                              ▼            ▼                ▼
                                        ┌──────────┐ ┌──────────┐   ┌──────────┐
                                        │  ±15 V   │ │  48 V    │   │   5 V    │
                                        │  DC-DC   │ │  Direct  │   │  Buck    │
                                        │ (Op-Amps)│ │ (TPA3255)│   │(Relay/  │
                                        └────┬─────┘ └────┬─────┘   │  LEDs)  │
                                             │            │          └────┬────┘
                                             │            │               │
 ┌───────┐    ┌─────────┐   ┌─────────┐  ┌──┴──────┐  ┌──┴──────┐  ┌────┴────┐
 │ INPUT ├───▶│ PREAMP  ├──▶│ HI-PASS ├─▶│  EDGER  ├─▶│  TILT   │  │  LED    │
 │ Jack  │    │ (Gain)  │   │ FILTER  │  │(Clipper)│  │  TONE   │  │ DRIVER  │
 └───┬───┘    └────┬────┘   │100-200Hz│  │Soft/Hard│  │ CONTROL │  └────┬────┘
     │             │        └─────────┘  └─────────┘  └────┬────┘       │
     │             ▼                                       │            ▼
     │        ┌─────────┐                             ┌────┴────┐  ┌─────────┐
     │        │  THRU   │                             │ VOLUME  │  │  LEDs   │
     │        │ OUTPUT  │                             │  POT    │  │(In Enc.)│
     │        └─────────┘                             └────┬────┘  └─────────┘
     │                                                     │
     │                                                     ▼
     │                                              ┌─────────────┐     ┌──────────────┐
     │                                              │  TPA3255    │     │   MIYAKO     │
     │                                              │  100W Class │────▶│   DU-100     │
     │                                              │  D Power Amp│     │ (via tube)   │
     │                                              └─────────────┘     └──────────────┘
     │
     │         ┌───────────┐     ┌─────────┐
     └────────▶│ FOOTSWITCH├────▶│  RELAY  │ (True Bypass)
               │   Jack    │     │  DPDT   │
               └───────────┘     └─────────┘
```

---

## Compression Driver Specs — MIYAKO DU-100

| Parameter | Value |
|---|---|
| Type | Compression Driver Horn Unit |
| Power Rating | 100 W |
| Impedance | 16 Ω |
| Frequency Response | 60 Hz – 8 kHz |
| Sensitivity | 115 dB / 1 W / 1 m |
| Diaphragm | Neodymium |
| Body | Aluminum |

---

## Section-by-Section Circuit Design

---

### 1. Power Supply

A 48 V / ≥3 A external supply feeds the board through a DC barrel jack (see **Power Supply Sourcing Guide** below for specific recommendations). On-board, we derive three rails:

| Rail | Purpose | Method |
|---|---|---|
| **48 V** | TPA3255 power amp | Direct from brick (fused, filtered) |
| **±15 V** | Op-amp analog stages | Isolated DC-DC module (e.g., TRACO TEN 5-4823 or Murata MEE3S4815SC) |
| **5 V** | Relay, footswitch logic, LEDs | Buck converter (LM2596-5.0 or MP1584) |

#### Schematic

```
         F1 (3A)    D1 (Reverse Polarity)           C1          C2
48V IN ──┤├────┬───▶│──┬───────────────────┬────────┤├────┬─────┤├────┬──── +48V RAIL
                │      │                   │      470µF   │   100nF   │
               GND     │                   │      63V     │   X7R     │
                        │                   │             GND         GND
                        │                   │
                        │    ┌──────────────┴──────────────┐
                        │    │    Isolated DC-DC Module     │
                        │    │    48V → ±15V (3-5W)        │
                        │    │                              │
                        │    │  +Vin   +Vout  -Vout  -Vin  │
                        │    └───┬───────┬──────┬──────┬───┘
                        │        │       │      │      │
                        │       48V   +15V   -15V    GND
                        │               │      │
                        │              ┌┤├┐  ┌┤├┐   C3,C4: 100µF/25V
                        │              │   │  │   │   C5,C6: 100nF ceramic
                        │             GND GND GND GND
                        │
                        │    ┌──────────────────────────┐
                        ├───▶│   LM2596-5.0 Buck Module  │
                        │    │   48V → 5V @ 1A           │
                        │    │  VIN  VOUT  GND  FB       │
                        │    └───┬────┬────┬────┘
                        │        │   5V   GND
                        │        │    │
                        │       48V  ┌┤├┐  C7: 220µF/10V
                        │           GND
                        │
        SW1 (Power)     │
GND ──────/  /──────────┴──── POWER GND RAIL
         SPST

D1: P-channel MOSFET (Si4435 or IRLML6402) for low-loss reverse polarity protection
    Gate → GND via 100kΩ, Source → input, Drain → +48V rail
    (Or use Schottky SB560 if simplicity preferred — 0.6V drop)
```

**Notes:**
- F1 is a 3 A slow-blow fuse in a PCB fuse holder
- C1 (470 µF / 63 V) provides bulk energy storage
- C2 (100 nF ceramic) decouples high-frequency noise
- The isolated DC-DC module provides galvanically isolated ±15 V — this eliminates ground loops between the analog and power stages
- The LM2596-5.0 module is a simple, cheap, and widely available voltage regulator
- SW1 (power) is on the ground side to keep the positive rail always hot only after the fuse

---

### 2. Input Stage & Preamp

The input is designed for **guitar-level signals** (~10–100 mV peak, high impedance). We use a high-impedance (1 MΩ) input buffer followed by a variable-gain stage.

#### Schematic

```
                       R1 1MΩ                    C1 100nF
INPUT JACK ──┬────────/\/\/──┬───────────────────┤├──────┬────────────┐
  (TIP)      │               │                           │            │
             │              GND                          │     R3     │
            R2 1MΩ                                       │   ┌/\/\/┐  │
             │                                           │   │ 47kΩ│  │
            GND                                          │   │     │  │
                                                         │   ├─────┘  │
                         ┌───────────────────────────────┘   │        │
                         │                                   │        │
                         │          ┌────────────────────────┘        │
                         │          │                                 │
                         │  RV1     │   100kΩ Log (GAIN)              │
                         │  ┌───────┤◁──────/\/\/──┐                  │
                         │  │       │              │                  │
                         │  │   R4  │              │                  │
                         │  │  1kΩ  │              │       ┌─────┐   │
                         │  │   │   │              │    ┌──┤+    │   │
                         │  │   │   └──────────────┴────┤  │NE   ├───┴──▶ TO HPF
                         │  │  GND                      │  │5532 │        (PREAMP OUT)
                         │  │                       ┌───┤- │ A   │
                         │  │                       │   └──┤     │
                         │  │                       │      └─────┘
                         │  │                       │         │
                         │  │           R5 1kΩ      │        V+  V-
                         │  └──────────/\/\/────────┘       +15  -15
                         │
                         │
                         │         ┌─────┐
                         │      ┌──┤+    │
                         └──────┤  │NE   ├──────────▶ THRU OUTPUT JACK
                                │  │5532 │
                            ┌───┤- │ B   │
                            │   └──┤     │
                            │      └─────┘
                            │         │
                            └─────────┘  (Unity gain follower)
```

**Component Values:**
- R1: 1 MΩ (input impedance — guitar friendly)
- R2: 1 MΩ (pulldown to prevent DC offset when unplugged)
- C1: 100 nF (AC coupling, f-3dB ≈ 1.6 Hz with 1 MΩ)
- R3: 47 kΩ (sets maximum gain ≈ 47×/33 dB)
- R4: 1 kΩ (minimum gain resistor — gain never below ~1×)
- RV1: 100 kΩ log pot (GAIN knob)
- R5: 1 kΩ (gain set resistor — Gain = 1 + R3/(R5 + RV1))
  - Gain range: ~1× (RV1 max) to ~48× (RV1 zero) → 0 to 33 dB
- NE5532A: Low-noise dual op-amp (section A = gain, section B = thru buffer)

**Gain Formula:**
$$A_v = 1 + \frac{R3}{R5 + RV1}$$
- RV1 = 0 Ω → $A_v = 1 + 47000/1000 = 48\times$ (33.6 dB)
- RV1 = 100 kΩ → $A_v = 1 + 47000/101000 ≈ 1.47\times$ (3.3 dB)

---

### 3. High-Pass Filter (Sallen-Key, 2nd Order Butterworth)

Removes low frequencies that waste amplifier power and could damage the compression driver. The TalkStar uses a fixed 150 Hz. We offer a **switchable** cutoff: 100 Hz / 150 Hz / 200 Hz.

#### Schematic

```
                    R1              R2
PREAMP OUT ──┬────/\/\/──┬────────/\/\/──┬──────┐
             │           │               │      │
             │          C1               │      │
             │           │               │   ┌──┤+   ┐
             │          GND              │   │  │NE   │
             │                          C2   │  │5532 ├──┬──▶ TO EDGER
             │                           │   │  │ A   │  │
             │                          GND  │  └┤-   ┘  │
             │                               │   │       │
             │                               │   └───────┘ (Voltage follower
             │                               │               feedback)
             │                               │
             └───────────────────────────────┘

FREQUENCY SELECT (SW2 — 1P3T rotary or 3-pos toggle):

Position 1 (100 Hz):  R1 = R2 = 15 kΩ,   C1 = C2 = 100 nF
  fc = 1/(2π × 15k × 100n) = 106 Hz

Position 2 (150 Hz):  R1 = R2 = 10 kΩ,   C1 = C2 = 100 nF
  fc = 1/(2π × 10k × 100n) = 159 Hz

Position 3 (200 Hz):  R1 = R2 = 8.2 kΩ,  C1 = C2 = 100 nF
  fc = 1/(2π × 8.2k × 100n) = 194 Hz

Implementation: SW2 switches R1/R2 pairs (use 3 pairs of resistors on PCB,
switch selects which pair is in circuit).
```

**Design Notes:**
- Sallen-Key topology gives a flat (Butterworth) response with unity gain
- 2nd order = 12 dB/octave rolloff below cutoff
- Butterworth Q = 0.707 (maximally flat passband)
- Use 1% metal film resistors for accuracy

**Transfer Function (2nd order Butterworth HPF):**
$$H(s) = \frac{s^2}{s^2 + \frac{\omega_0}{Q}s + \omega_0^2}$$

where $\omega_0 = 2\pi f_c$ and $Q = 0.707$

---

### 4. Edger (Clipping/Distortion Stage)

This is the signature talk box grit. We provide **three modes** via a switch: Clean, Soft Clip, and Hard Clip. Plus a phase **Invert** switch and **Edger Level** control.

#### Schematic

```
                         SW3 (Mode Select: 1P3T)
                         ┌─── Position 1: CLEAN (no diodes)
                         │    Position 2: SOFT CLIP (diodes in feedback)
                         │    Position 3: HARD CLIP (diodes to ground)
                         │
                    R1 4.7kΩ               R2 47kΩ
HPF OUT ──┬────────/\/\/──┬───────────────/\/\/──────────────┬───┐
           │               │                                  │   │
           │               │         SOFT CLIP PATH:          │   │
           │               │         D1a ──▶│──┐              │   │
           │               │         D1b ──◀│──┤  (anti-      │   │
           │               │                │  │  parallel)    │   │
           │               │                │  │              │   │
           │               │         When SW3=Pos2:           │   │
           │               │         Diodes in parallel       │   │
           │               │         with R2 (in feedback)    │   │
           │               │                                  │   │
           │               │                            ┌─────┘   │
           │               │                            │         │
           │               │                         ┌──┤-   ┐    │
           │               │                         │  │NE   │   │
           │               │                         │  │5532 ├───┴──┐
           │               │                    ┌────┤+ │ A   │      │
           │               │                    │    └──┤     ┘      │
           │               │                    │       │            │
           │               │                   GND     V+  V-       │
           │               │                          +15  -15      │
           │               │                                        │
           │               │    HARD CLIP PATH (When SW3=Pos3):     │
           │               │    Diodes D2a/D2b anti-parallel        │
           │               │    to GND AFTER op-amp output          │
           │               │                                        │
           │               │              D2a ──▶│──┐               │
           │               └──────────    D2b ──◀│──┤               │
           │                                     │  │               │
           │                                    GND GND             │
           │                                                        │
           │                                                        │
           │              SW4 (INVERT — DPDT)                       │
           │              Flips signal phase (×-1)                  │
           │                                                        │
           │              ┌─────┐                                   │
           │           ┌──┤-    │         R3 10kΩ                   │
           ├───────────┤  │NE   ├───┬────/\/\/──┐                   │
           │    R4 10kΩ│  │5532 │   │           │                   │
           │  ┌─/\/\/──┤+ │ B   │  OUT       ┌──┘                  │
           │  │        └──┤     ┘    │       │                      │
           │  │           │     When SW4 ON: │                      │
           │ GND         V+  V-   output = -1 × input              │
           │            +15  -15  When SW4 OFF: bypassed            │
           │                                                        │
           │                                                        │
           │                    RV2 100kΩ Log (EDGER LEVEL)         │
           └────────────────────◁──/\/\/──┤                         │
                                          │                         │
                                         OUT ──────────▶ TO TILT TONE
```

**Component Values:**
- R1: 4.7 kΩ (input resistor)
- R2: 47 kΩ (feedback resistor — gain = R2/R1 = 10×/20 dB)
- D1a, D1b: 1N4148 (silicon, ~0.6 V clip) for soft clip — **OR** use LEDs (1N34A germanium for ~0.3 V, or red LEDs for ~1.7 V clip — experiment!)
- D2a, D2b: 1N4148 for hard clip
- R3, R4: 10 kΩ (unity-gain inverter)
- RV2: 100 kΩ log pot (EDGER LEVEL knob)
- SW3: 1P3T (clean/soft/hard mode selector)
- SW4: DPDT toggle (invert on/off)

**How the clipping works:**
- **Clean**: Only R2 in feedback. Linear gain of 10×.
- **Soft Clip**: Diodes in parallel with R2. When signal exceeds ~0.6 V, diodes conduct and reduce gain → smooth, round clipping.
- **Hard Clip**: Diodes to ground after op-amp. Output is hard-limited to ±0.6 V → aggressive, square-wave-like distortion.

---

### 5. Tilt Tone Control

A single-knob EQ that tilts the frequency response around a center point (~800 Hz). Turning CW boosts treble and cuts bass; turning CCW boosts bass and cuts treble. This is the same topology as the TalkStar but with a wider range.

#### Schematic

```
                R1 20kΩ           R2 20kΩ
EDGER OUT ────/\/\/──┬──────────/\/\/──────┬─────────────────┐
                     │                     │                 │
                    C1                     │            ┌────┤-   ┐
                   4.7nF                   │            │    │NE   │
                     │                     │            │    │5532 ├──┬──▶ TO VOL
                     ├──── RV3 ────────────┤            │    │ A   │  │
                     │   50kΩ Lin           │        ┌───┤+  │     ┘  │
                     │  (TONE knob)        │        │   └──┤     ┘    │
                     │                     │        │      │          │
                    C2                    C3       GND    V+  V-      │
                   47nF                  4.7nF           +15  -15     │
                     │                     │                          │
                    GND                   GND                         │
                                                                      │
                          R3 20kΩ                                     │
                     ┌───/\/\/─────────────────────────────────────────┘
                     │
                     └──▶ (feedback to inverting input)
```

**Component Values:**
- R1, R2, R3: 20 kΩ
- C1: 4.7 nF
- C2: 47 nF
- C3: 4.7 nF
- RV3: 50 kΩ linear pot (TONE knob)

**Behavior:**
- Pot center → flat response
- Pot CW → treble boost / bass cut (~±6 dB tilt)
- Pot CCW → bass boost / treble cut (~±6 dB tilt)
- Center/pivot frequency ≈ 800 Hz

---

### 6. Volume Control

Simple but important. A log-taper pot followed by a buffer to present a low impedance to the power amp input.

#### Schematic

```
                    RV4 100kΩ Log (VOLUME)
TONE OUT ──────────/\/\/──┬────◁
                          │
                          │         ┌─────┐
                          │      ┌──┤+    │
                          └──────┤  │NE   ├──┬──▶ TO POWER AMP
                                 │  │5532 │  │
                             ┌───┤- │ B   │  │
                             │   └──┤     ┘  │
                             │      │        │
                             └──────┴────────┘  (Unity gain buffer)
                                    │
                                   V+  V-
                                  +15  -15
```

**Component Values:**
- RV4: 100 kΩ logarithmic (audio taper) pot (VOLUME knob)
- Buffer: NE5532 section (remaining half of a dual op-amp)

---

### 7. Power Amplifier — TPA3255 (Class D, ~65 W clean into 16 Ω)

The TPA3255 is TI's flagship Class D audio amplifier:
- Up to **315 W** in PBTL mode, **>90% efficiency**, **18–53.5 V** supply range
- In **BTL mode** at 48 V into 16 Ω: **~65 W RMS clean** (1% THD+N)
- Available in **HTSSOP-44** (TPA3255DDV) — **hand-solderable** with exposed leads on 0.5 mm pitch
- Also available in QFN-44 (TPA3255DKD) if you prefer reflow

**Key Components:**
- TPA3255DDV (HTSSOP-44, recommended) or TPA3255DKDR (QFN-44)
- L1, L2: 22 µH power inductors (Bourns SRP1245A-220M, ≥5 A)
- C_out: 1 µF / 100 V film (output filter)
- Zobel: 100 nF / 100 V + 10 Ω per channel
- C_bulk: 2× 470 µF / 63 V, C_dec: 4× 100 nF + 2× 10 µF MLCC
- Follow TI SLAA701 reference layout — Class D is **very layout-sensitive**

| Parameter | Value |
|---|---|
| Output Power (BTL, 16 Ω, 48 V, 1% THD) | ~65 W RMS |
| Output Power (BTL, 16 Ω, 48 V, 10% THD) | ~75 W RMS |
| Efficiency | >90% |
| SNR | >105 dB |
| SPL at driver (65 W into DU-100) | **~133 dB** |

---

### 8. LED Lighting System (Signal-Following Brightness)

Envelope follower (precision rectifier + RC filter) drives an IRLZ44N MOSFET to modulate LED brightness proportional to signal amplitude.

- R1: 10 kΩ, D1: 1N4148, R3: 100 kΩ, C1: 10 µF (τ ≈ 1s decay)
- Q1: IRLZ44N logic-level MOSFET, R4: 10 kΩ gate resistor
- 12 white LEDs: 3 parallel strings of 4 series, R_led = 150 Ω each, powered from +15 V
- SW5: SPST toggle (LED on/off)

---

### 9. True Bypass (Relay + Footswitch)

DPDT relay (Omron G5V-2-DC5) driven by 2N3904 + CD4013 D flip-flop toggle.

- Footswitch: momentary NO → CLK of CD4013 (Q̄→D for toggle)
- K1 relay: COM1=Input, COM2=Amp input, NC paths create bypass wire
- Bypass LED: green 3mm, 330 Ω from +5V, driven by Q output

---

## Power Analysis — The Honest Math

Understanding exactly how much clean power we deliver to the driver:

**BTL Class D formula:** $P = \frac{(V_{PVCC} - V_{drop})^2}{2 \times R_L}$

| Parameter | Value |
|---|---|
| PVCC (supply voltage) | 48 V |
| V_drop (MOSFET losses, ~4 FETs in BTL path) | ~2 V |
| R_L (DU-100 impedance) | 16 Ω |
| **P_clean (1% THD+N)** | **(48 − 2)² / (2 × 16) = ~66 W RMS** |
| P_clipping (10% THD) | ~75 W RMS |

**What does 65 W mean in practice?**

| Power | SPL at Driver (115 dB/W/m) | Context |
|---|---|---|
| TalkStar (20 W) | 128 dB | Already painfully loud |
| **TalkBox at 65 W** | **133 dB** | **Jet engine at 100 ft** |
| Theoretical 100 W | 135 dB | Only 2 dB more — barely audible difference |

> **Key insight:** Going from 65 W to 100 W adds only **1.9 dB** — not a perceptible loudness increase. The TalkStar runs 20 W; we run 65 W clean. That's a **5 dB advantage** — noticeably and significantly louder, with massive headroom for clean peaks.

**Why not push to 100 W?**
- TPA3255 max PVCC is 53.5 V → ~78 W max into 16 Ω (still not 100 W)
- Running the DU-100 at its absolute thermal limit shortens its life dramatically
- At 65 W you're at 65% of the driver's rated power — **perfect for reliability + clean headroom**
- Talkbox use is inherently bursty (musical phrases, not continuous sine) — thermal load is well below rated

**Bottom line:** 48 V supply gives us the ideal balance of power, driver longevity, and clean headroom.

---

## Power Supply Sourcing Guide

You need **48 V / ≥3 A** (144 W minimum). You do NOT need 5 A — the amp draws ~1.5 A peak plus ~0.2 A for the analog section. A 3 A supply gives comfortable headroom.

### Option A: Desktop Adapter (Recommended — Simplest)

Plug-and-play, UL/CE certified, fully isolated from mains. Just wire to a DC barrel jack.

| Supply | Rating | Connector | Price | Where to Buy |
|---|---|---|---|---|
| Mean Well GST160A48-R7B | 48 V / 3.34 A (160 W) | 3-pin IEC inlet | ~$30–40 | Mouser, Digi-Key, Amazon |
| Mean Well GST220A48-R7B | 48 V / 4.6 A (220 W) | 3-pin IEC inlet | ~$35–50 | Mouser, Digi-Key |
| Generic "48V 3A adapter" | 48 V / 3 A | Barrel 5.5×2.5 mm | ~$12–20 | Amazon, AliExpress |

> **Mean Well GST160A48** is the sweet-spot pick: reliable brand, exactly the right power, widely stocked.

### Option B: Open-Frame Internal Supply

Mount inside the enclosure with an IEC C14 panel inlet. Cheaper, but requires mains wiring (120/240 VAC inside the box — use caution and proper insulation).

| Supply | Rating | Size | Price |
|---|---|---|---|
| Mean Well LRS-150-48 | 48 V / 3.2 A (150 W) | 159×97×30 mm | ~$18 |
| Mean Well LRS-200-48 | 48 V / 4.4 A (200 W) | 159×97×30 mm | ~$22 |

### Option C: Surplus / Repurposed

48 V is standard for telecom and PoE network equipment. Surplus server and PoE switch power supplies are abundant on eBay for $10–15. Just verify polarity and ripple with a scope before use.

### What about 36 V or 24 V?

| Supply Voltage | Power into 16 Ω | Verdict |
|---|---|---|
| 24 V | ~16 W | Same as TalkStar — defeats the purpose |
| 36 V | ~36 W | Passable, but sacrifices half the power |
| **48 V** | **~65 W** | **Sweet spot for this driver** |
| 51 V (max recommended) | ~75 W | Marginal gain, harder to source |

**Stick with 48 V.** It's the right voltage for 16 Ω and widely available.

---

## Driver Protection

The DU-100 is rated 100 W. Our amp delivers ~65 W clean — 35 W of built-in thermal margin. Additional protection:

1. **Speaker-line fuse** — Add a **2 A slow-blow fuse** in series with the speaker output. At 2 A RMS into 16 Ω = 64 W continuous. Peaks above 2 A pass through (slow-blow), but sustained overload blows the fuse before the voice coil burns. Cost: $0.50.

2. **TPA3255 built-in protection** — Overcurrent (OCP), over-temperature (OTP), and short-circuit protection. These protect the amp IC itself.

3. **Thermal management** — The DU-100 horn body is the heatsink. Ensure adequate ventilation around the driver. Don't enclose it in an airtight box.

4. **Duty cycle** — Talkbox use is inherently intermittent (you play phrases, not continuous tones). Real-world average power is typically 10–20 W even when peaks hit 60 W. This is very safe for the driver.

> **Practical rule of thumb:** If the driver horn gets too hot to touch (~60°C), back off the volume. Otherwise, you're fine.

---

## Op-Amp Allocation (NE5532 = dual)

| IC | Section A | Section B |
|---|---|---|
| **U1** | Preamp (gain stage) | Thru output buffer |
| **U2** | HPF Sallen-Key | Invert stage |
| **U3** | Edger clipping | Tilt tone |
| **U4** | Volume buffer | Envelope follower |

---

## Master BOM

### ICs

| Qty | Part | Package | Notes |
|---|---|---|---|
| 1 | TPA3255DDV | HTSSOP-44 | Class D power amp (hand-solderable) |
| 4 | NE5532P | DIP-8 | Low-noise dual op-amp |
| 1 | CD4013BE | DIP-14 | Dual D flip-flop (bypass toggle) |
| 1 | LM2596S-5.0 | TO-263 | 5V buck regulator |
| 1 | TRACO TEN 5-4823 | DIP-24 | 48V → ±15V isolated DC-DC |

### Semiconductors

| Qty | Part | Notes |
|---|---|---|
| 6 | 1N4148 | Signal diodes (clipping + protection) |
| 1 | SB560 or SS54 | 5A Schottky (reverse polarity protection) |
| 1 | 1N5819 | Schottky (buck freewheeling) |
| 1 | IRLZ44N | Logic-level N-MOSFET (LED driver) TO-220 |
| 1 | 2N3904 | NPN transistor (relay driver) TO-92 |

### Relay

| Qty | Part | Notes |
|---|---|---|
| 1 | Omron G5V-2-DC5 | DPDT 5V coil, signal relay |

### Resistors (1/4W 1% metal film unless noted)

| Qty | Value | Purpose |
|---|---|---|
| 2 | 1 MΩ | Input impedance / pulldown |
| 1 | 47 kΩ | Preamp feedback |
| 1 | 1 kΩ | Preamp gain floor |
| 2 | 8.2 kΩ | HPF 200 Hz pair |
| 2 | 10 kΩ | HPF 150 Hz pair |
| 2 | 15 kΩ | HPF 100 Hz pair |
| 1 | 4.7 kΩ | Edger input R |
| 1 | 47 kΩ | Edger feedback |
| 2 | 10 kΩ | Inverter equal R pair |
| 3 | 20 kΩ | Tilt tone network |
| 1 | 20 kΩ | TPA3255 input bias |
| 2 | 10 Ω 1W | Zobel network |
| 3 | 10k + 100k + 10k | Envelope follower |
| 3 | 150 Ω 1/2W | LED string current limit |
| 1 | 330 Ω | Bypass indicator LED |
| 1 | 8.2 kΩ | Power indicator LED |
| 3 | 10 kΩ | Relay / flip-flop / debounce |

### Capacitors

| Qty | Value | Type | Purpose |
|---|---|---|---|
| 3 | 100 nF | Film | Coupling (input, HPF, output) |
| 1 | 4.7 nF | Film | Tilt bass leg |
| 1 | 47 nF | Film | Tilt mid |
| 1 | 4.7 nF | Film | Tilt treble leg |
| 2 | 1 µF | Film | TPA3255 input coupling |
| 2 | 100 nF | Ceramic C0G | Zobel |
| 4 | 100 nF | MLCC X7R | PVCC bypass |
| 2 | 10 µF | MLCC X5R | PVCC bulk |
| 2 | 100 nF | Ceramic | Bootstrap |
| 2 | 470 µF 63V | Electrolytic | 48V rail bulk |
| 2 | 100 µF 25V | Electrolytic | ±15V rail bulk |
| 8 | 100 nF | Ceramic | Op-amp V+/V− bypass (×4 ICs) |
| 2 | 10 µF 25V | Electrolytic | ±15V extra filtering |
| 1 | 220 µF 10V | Electrolytic | 5V buck output |
| 2 | 100 nF + 10 µF | Ceramic + Elec | HPF Sallen-Key pair |
| 1 | 10 µF | Electrolytic | Envelope filter |
| 1 | 100 nF | Ceramic | Debounce |

### Inductors

| Qty | Value | Rating | Purpose |
|---|---|---|---|
| 2 | 22 µH | 5A, shielded (Bourns SRP1245A) | TPA3255 LC output filter |
| 1 | 33 µH | 1A | LM2596 buck inductor |

### Potentiometers (16mm Alpha, D-shaft)

| Qty | Value | Taper | Label |
|---|---|---|---|
| 1 | 100 kΩ | Log (A) | GAIN |
| 1 | 100 kΩ | Log (A) | EDGER |
| 1 | 50 kΩ | Linear (B) | TONE |
| 1 | 100 kΩ | Log (A) | VOLUME |

### Switches

| Qty | Type | Purpose |
|---|---|---|
| 1 | SPST toggle | Power on/off |
| 1 | 1P3T rotary | HPF frequency select (100/150/200 Hz) |
| 1 | 1P3T rotary | Edger mode (clean/soft/hard) |
| 1 | DPDT toggle | Phase invert |
| 1 | SPST toggle | LED system on/off |

### Connectors

| Qty | Type | Purpose |
|---|---|---|
| 1 | 1/4" mono jack | Guitar INPUT |
| 1 | 1/4" mono jack | THRU output |
| 1 | 1/4" TRS jack | Footswitch |
| 1 | Screw terminal or Speakon NL2 | Speaker output |
| 1 | DC barrel jack 5.5×2.5mm | 48V power input |
| 1 | JST-XH 2-pin header | Internal LED strip connector |

### LEDs

| Qty | Part | Purpose |
|---|---|---|
| 12 | White 5mm (20mA, 3.2V) | Enclosure lighting (3 strings of 4) |
| 1 | Green 3mm | Bypass indicator |
| 1 | Red 3mm | Power indicator |

### Misc Hardware

| Qty | Part |
|---|---|
| 1 | 3A fuse + panel-mount holder |
| 1 | 2A slow-blow fuse (speaker protection) |
| 4 | Knobs (D-shaft, to match pots) |
| 1 | Hammond 1590DD enclosure (187.5×119.5×37mm) |
| 4 | M3×10mm standoffs + screws (PCB mount) |
| 1 | Momentary SPST footswitch (NO) |
| — | 22 AWG hookup wire, heatshrink, solder |

### External (not on PCB)

| Qty | Part | Notes |
|---|---|---|
| 1 | 48V / ≥3A DC power supply | ≥144W (see Sourcing Guide above) |
| 1 | MIYAKO DU-100 | 100W 16Ω compression driver |
| 1 | Vinyl tube 3/8" ID × 3 ft | Medical/food-grade |
| 1 | Tube-to-driver threaded adapter | 3D-printed or machined |

---

## PCB Design Guidelines

1. **4-layer board recommended** — L1: Signal + Power, L2: GND plane, L3: ±15V/5V planes, L4: Signal + Power
2. **Solid ground plane (L2)** — unbroken under TPA3255 and analog sections
3. **Star ground topology** — single point where analog GND, digital GND, and power GND meet at TPA3255 PGND
4. **TPA3255 layout** — follow TI SLAA701 app note; decoupling caps within 3mm of pins; ≥9 thermal vias under exposed pad; differential output traces matched length ±0.5mm
5. **Power trace widths** — 48V rail ≥ 2mm (3A); speaker outputs ≥ 2mm; ±15V ≥ 0.5mm; 5V ≥ 0.75mm
6. **Signal traces** — ≥ 0.25mm (10 mil); route away from Class D outputs and switching inductor
7. **100nF bypass cap at every op-amp V+ and V− pin** — place as close as physically possible
8. **Input wiring** — shielded cable from input jack to PCB; twisted pair from PCB speaker out to Speakon jack

---

## Power Budget

| Rail | Source | Load | Current |
|---|---|---|---|
| 48V | Supply direct | TPA3255 PVCC | ~1.5A peak (65W/48V/0.9 eff) |
| ±15V | TRACO TEN 5-4823 | 4× NE5532 | ~40 mA |
| +5V | LM2596 buck | Relay + CD4013 + LEDs | ~200 mA |
| **Total** | | | **~1.8A from 48V** (3A supply = 40% headroom) |

---

## Signal Chain Summary

```
Guitar → [Input Jack] → [Relay Bypass SW] → C_in →
  Preamp (NE5532, gain pot 1-47×) →
  HPF (Sallen-Key 2nd order, switchable 100/150/200 Hz) →
  Edger (clean / soft-clip / hard-clip, invert switch) →
  Tilt Tone (single knob, ~800 Hz pivot) →
  Volume (log pot + buffer) →
  TPA3255 (BTL, 48V, ~65W clean into 16Ω) →
  LC Filter → [Speaker Terminal] → MIYAKO DU-100 → Vinyl Tube → Mouth
```

---

## References & Datasheets

- [TPA3255 Datasheet + EVM Layout Guide (TI SLAA701)](https://www.ti.com/product/TPA3255)
- [NE5532 Datasheet](https://www.ti.com/product/NE5532)
- [CD4013B Datasheet](https://www.ti.com/product/CD4013B)
- [LM2596 Datasheet](https://www.ti.com/product/LM2596)
- [TRACO TEN 5 Series](https://www.tracopower.com/products/ten5.pdf)
- [MIYAKO DU-100 Specs](https://miyakopro.com)

---

## Build Notes

1. **Power supply first** — solder buck + TRACO + protection. Verify 5V, ±15V, 48V before inserting ICs
2. **Op-amp stages next** — install NE5532 sockets. Test each stage with a signal generator + oscilloscope
3. **TPA3255 last** — HTSSOP-44 package: fine-pitch (0.5mm) but hand-solderable with flux, fine tip iron, and solder wick. Use plenty of flux on the exposed pad and tack corners first. Inspect for solder bridges with magnification
4. **Dummy load test** — use 16Ω / 100W resistor before connecting the compression driver
5. **Grounding** — verify star ground. Touch up any ground loops with scope probe on speaker output
6. **Vinyl tube** — 3/8" ID medical-grade. Thread onto driver horn with 3D-printed adapter. ~3 ft length
7. **Enclosure** — drill template for 4 pots, 5 switches, 4 jacks, 1 fuse, 1 DC jack. Label with rub-on transfers or UV print

---

*Designed to outperform the Banshee TalkStar at every stage. Let's build it.*
