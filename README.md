---
# Talkbox Driver PCB Project - Bill of Materials (BoM)

This project powers a 100 W, 16 Ω compression driver for use in a talkbox. It supports both guitar (instrument level) and synth (line level) inputs. The design includes volume control, amplification, and output filtering to drive the speaker effectively.

---

## 🎛️ Inputs
### 1. Instrument Level Input (Guitar)
- **Connector**: 1× 6.35 mm mono jack (panel-mount)
- **Coupling Capacitor**: 1× 10 µF film or bipolar electrolytic capacitor
- **Resistor Divider / Pad** (optional): 1× 1 MΩ to ground

### 2. Line Level Input (Synth)
- **Connector**: 1× 6.35 mm or 3.5 mm stereo jack (panel-mount)
- **Coupling Capacitor**: 1× 10 µF film or bipolar electrolytic capacitor
- **Series Resistor**: 1× 10 kΩ (to mix to mono and protect input)

---

## 🎚️ Volume Control
- **Potentiometer**: 1× 10 kΩ logarithmic (audio taper), panel-mount
- **Knob**: 1× plastic or aluminum knob to fit above pot shaft

---

## 🔊 Amplifier Section
### Prebuilt Module (Recommended)
- **Module**: 1× TDA7492P 25 W×2 stereo amplifier board or 1× TPA3116D2 mono amplifier
- **Gain Setting**: Usually configurable with onboard resistors or jumpers
- **Heatsink**: Should be included; necessary for high power

### Or Discrete Option (DIY Amp)
- **Op-amp**: 1× TL072 or NE5532 (dual op-amp)
- **Power Amp IC**: 1× LM3886 or TDA7294 (but requires large heatsink and power supply)

> 💡 Prebuilt amp modules save time and are optimized for reliability.

---

## 🔋 Power Supply Section
- **DC Jack**: 1× 2.1 mm barrel jack (panel-mount)
- **Switch**: 1× SPST toggle or rocker switch (12 V or 24 V @ 5 A rated)
- **Power Supply**:
  - 1× 12 V–24 V DC @ 3–5 A wall adapter or external supply (matched to amp module specs)
- **Capacitors for Decoupling**:
  - 2× 100 µF 25 V electrolytic
  - 2× 100 nF ceramic (X7R)
- **Power LED**:
  - 1× 3 mm or 5 mm red LED
  - 1× 330 Ω resistor (for 12 V)

---

## 📤 Output
- **Speaker Terminals**:
  - 1× 2-pin screw terminal or 1× panel-mount speakON or banana jack pair
- **Compression Driver**:
  - External speaker with 100 W @ 16 Ω

---

## 🧩 Optional Add-ons
- **Fuse**: Inline or PCB fuse holder (2 A slow-blow)
- **Indicator LED for Signal**: Use an op-amp rectifier circuit if needed
- **Mute Switch**: SPDT toggle, shorts input to GND

---

## 🖨️ PCB Recommendations
- Use thick traces for speaker output and power (at least 50 mil width)
- Use ground plane under op-amps and signal areas
- Separate analog ground from power ground if using discrete amps
- Mount pots and jacks on enclosure and wire to board

---

## 📦 Summary BoM Table

| Qty | Description                          | Value / Notes                       |
|-----|--------------------------------------|-------------------------------------|
| 1   | Audio Potentiometer                  | 10 kΩ logarithmic, panel-mount      |
| 1   | Guitar Input Capacitor               | 10 µF film or bipolar electrolytic  |
| 1   | Line Input Capacitor                 | 10 µF film or bipolar electrolytic  |
| 1   | Line Input Resistor                  | 10 kΩ                               |
| 1   | Guitar Pull-down Resistor            | 1 MΩ                                |
| 1   | Amplifier Module                     | TDA7492P / TPA3116D2                |
| 1   | DC Barrel Jack                       | 2.1 mm center positive              |
| 1   | Power Switch                         | SPST 12–24 V 5 A rated              |
| 2   | Power Supply Capacitors              | 100 µF 25 V electrolytic            |
| 2   | Decoupling Capacitors                | 100 nF ceramic (X7R)                |
| 1   | Power LED                            | 3 mm red                            |
| 1   | LED Current Limiting Resistor        | 330 Ω                               |
| 1   | Speaker Terminal                     | 2-pin screw or banana jack pair     |
| 2   | Input Jacks                          | 6.35 mm (1/4") mono/stereo          |
| 1   | Volume Knob                          | To fit pot shaft                    |
| —   | PCB + Enclosure                      | Custom milled or hand-drilled      |

---

Let me know if you'd like this exported as a `.kicad_sch` file or turned into a full layout!
