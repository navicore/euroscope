# Schematic Design Plan

## Purpose
Complete schematic design plan for Trace module, ready for KiCad implementation.

---

## Schematic Organization

### Sheet Structure (Hierarchical Design)

**Sheet 1: Power Supply**
- Eurorack power input (±12V, GND)
- Reverse polarity protection
- Buck regulator (3.3V generation)
- Power distribution and decoupling

**Sheet 2: Input Channel 1**
- Input protection and conditioning
- Voltage divider and offset circuit
- ADC protection
- Passthrough buffer

**Sheet 3: Input Channel 2**
- Duplicate of Channel 1 circuit
- Independent signal path

**Sheet 4: Microcontroller & Display**
- RP2040 connections
- Display interface (SPI)
- Programming/debug interface

**Sheet 5: User Controls**
- Encoders (with debouncing)
- Toggle switches
- Status LEDs (dual-color)

**Sheet 6: Connectors**
- Input jacks (with normalling)
- Output jacks
- Power connector

---

## Circuit Block Details

### Block 1: Power Supply (Sheet 1)

#### Input Protection
```
Eurorack Bus (±12V, GND)
    │
    ├─── Reverse Polarity Protection (Schottky diode or P-FET)
    │
    ├─── +12V Rail → Buck Regulator → +3.3V
    │                    ↓
    │              (filtering caps)
    │
    ├─── -12V Rail → Direct to analog circuits
    │
    └─── GND (analog/digital ground strategy)
```

**Components:**
- **J1**: 2×5 keyed IDC header (Eurorack power)
- **D_POL**: Reverse polarity protection diode (or P-channel MOSFET)
- **U_REG**: Buck regulator (AP63203 or LMR33630)
- **L1**: Inductor (4.7µH-10µH, >2A)
- **C_IN**: Input capacitor (10µF ceramic)
- **C_OUT**: Output capacitor (22µF ceramic)
- **C_FILT**: Additional filtering (10µF + 100nF on +3.3V rail)

**Design notes:**
- Star ground point for analog section
- Separate +3.3V trace routing to digital (RP2040) vs analog (offset reference)
- ±12V decoupling near TL074 (100nF ceramics)

#### Power Distribution
- +12V → TL074 op-amps (both channels)
- -12V → TL074 op-amps (both channels)
- +3.3V → RP2040, display, offset voltage reference
- GND → Analog ground plane + digital ground (single-point connection)

---

### Block 2: Input Channel (Sheets 2 & 3)

Full circuit per docs/06-input-channel-schematic.md:

#### Stage 1: Input Protection
```
J_IN (3.5mm jack)
    │
    ├─── R_PROT (1kΩ) ───┬─── [to divider]
    │                    │
    │                 D1 ↑ (to +12V, SS14)
    │                 D2 ↓ (to -12V, SS14)
```

**Components (per channel):**
- **J_IN**: Thonkiconn PJ398SM (input jack)
- **R_PROT**: 1kΩ, 0805, 1/4W (current limiting)
- **D1**: SS14 Schottky (anode to signal, cathode to +12V)
- **D2**: SS14 Schottky (cathode to signal, anode to -12V)

#### Stage 2: Voltage Divider
```
[After protection] ──R1 (620kΩ)──┬──R2 (100kΩ)──GND
                                  │
                               [to summing amp]
```

**Components:**
- **R1**: 620kΩ, 0805, 1% metal film
- **R2**: 100kΩ, 0805, 1% metal film

**Calculation**: Divides ±12V → ±1.67V (7.2:1 attenuation)

#### Stage 3: Offset & Buffer (TL074 op-amp)
```
Offset Reference:
+3.3V ──R3 (91kΩ)──┬──R4 (100kΩ)──GND
                   │
                  C1 (100nF)
                   │
                [+1.73V reference]

Summing Amp (U1A or U1C):
         R5 (100kΩ)
Signal ──────────┬──────(+) U1A
                 │         │
         R6 (100kΩ)       (─) ────┐
Offset ──────────┘         │      │
                          GND    OUT ──> [to ADC protection]
```

**Components:**
- **R3**: 91kΩ, 0805, 1% (offset reference top)
- **R4**: 100kΩ, 0805, 1% (offset reference bottom)
- **C1**: 100nF ceramic, 0805 (reference bypass)
- **R5**: 100kΩ, 0805, 1% (signal input to summing)
- **R6**: 100kΩ, 0805, 1% (offset input to summing)
- **U1A** (Ch1) or **U1C** (Ch2): TL074 op-amp section

**Op-amp connections:**
- Pin 1 (OUT), Pin 2 (IN-), Pin 3 (IN+) for U1A
- Pin 10 (OUT), Pin 9 (IN-), Pin 8 (IN+) for U1C
- Non-inverting summing config (IN- tied to OUT for unity gain)

#### Stage 4: ADC Protection
```
[Op-amp out] ──R7 (1kΩ)──┬──────> RP2040 ADC pin
                          │
                       D3 ↑ (to +3.3V, BAT85)
                       D4 ↓ (to GND, BAT85)
```

**Components:**
- **R7**: 1kΩ, 0805, 1/4W (current limiting)
- **D3**: BAT85 Schottky (anode to ADC, cathode to +3.3V)
- **D4**: BAT85 Schottky (cathode to ADC, anode to GND)

**ADC connections:**
- Ch1 → RP2040 GPIO26 (ADC0)
- Ch2 → RP2040 GPIO27 (ADC1)

#### Stage 5: Passthrough Buffer
```
[After R_PROT] ──┬──(+) U1B
                 │   │
                GND (─) ────> J_OUT (buffered output)
```

**Components:**
- **U1B** (Ch1) or **U1D** (Ch2): TL074 op-amp section
- **J_OUT**: Thonkiconn PJ398SM (output jack)

**Op-amp connections:**
- Pin 7 (OUT), Pin 6 (IN-), Pin 5 (IN+) for U1B
- Pin 14 (OUT), Pin 13 (IN-), Pin 12 (IN+) for U1D
- Voltage follower config (IN- tied to OUT)

#### TL074 Power Connections (Shared)
- **Pin 4** (V-): -12V rail + 100nF decoupling to GND
- **Pin 11** (V+): +12V rail + 100nF decoupling to GND

**Decoupling caps (per IC):**
- **C_V+**: 100nF ceramic, 0805 (close to pin 11)
- **C_V-**: 100nF ceramic, 0805 (close to pin 4)

---

### Block 3: Microcontroller (Sheet 4)

#### RP2040 Connections

**ADC Inputs:**
- GPIO26 (ADC0) ← Channel 1 signal (via R7, D3, D4)
- GPIO27 (ADC1) ← Channel 2 signal (via R7, D3, D4)

**SPI Display Interface:**
- GPIO10 (SPI1_SCK) → Display SCK
- GPIO11 (SPI1_TX / MOSI) → Display MOSI
- GPIO12 (SPI1_RX) → (not connected; available for future use)
- GPIO13 → Display CS (chip select)
- GPIO14 → Display DC (data/command)
- GPIO15 → Display RST (reset, optional)

**Display Power:**
- 3.3V → Display VCC
- GND → Display GND

**I2C / Encoder Interface (via GPIO):**
- GPIO16 → Encoder 1 A
- GPIO17 → Encoder 1 B
- GPIO18 → Encoder 1 SW (push switch)
- GPIO19 → Encoder 2 A
- GPIO20 → Encoder 2 B
- GPIO21 → Encoder 2 SW (push switch)

**Toggle Switches & LEDs:**
- GPIO22 → Toggle 1 (Ch1 Run/Stop) read
- GPIO23 → Toggle 2 (Ch2 Run/Stop) read
- GPIO6 → LED 1 Red control
- GPIO7 → LED 1 Green control
- GPIO8 → LED 2 Red control
- GPIO9 → LED 2 Green control

**USB & Programming:**
- GPIO0, GPIO1 → USB D+, D- (if using bare RP2040)
- Or: Use Pico module with built-in USB
- SWD debug: SWCLK, SWDIO (for programming/debug)

**RP2040 Power:**
- 3.3V → IOVDD, DVDD (all power pins)
- GND → GND (all ground pins)
- **Decoupling**: 100nF ceramic on each power pin pair (close to IC)

**Crystal (if bare RP2040):**
- 12MHz crystal + 2× load capacitors (15-27pF, calculate per formula)
- Or: Use Pico module (crystal included)

**Flash (if bare RP2040):**
- W25Q16 or W25Q32 (QSPI flash)
- Or: Use Pico module (flash included)

**Decision for V1**: Use **Raspberry Pi Pico module** to avoid crystal/flash complexity.

---

### Block 4: User Controls (Sheet 5)

#### Rotary Encoders

**Encoder 1 (Ch1 - Time/div):**
- **A phase** → GPIO16 (with 10kΩ pull-up to +3.3V)
- **B phase** → GPIO17 (with 10kΩ pull-up to +3.3V)
- **Switch** → GPIO18 (with 10kΩ pull-up to +3.3V)
- **Common** → GND

**Encoder 2 (Ch2 - Volts/div):**
- **A phase** → GPIO19 (with 10kΩ pull-up to +3.3V)
- **B phase** → GPIO20 (with 10kΩ pull-up to +3.3V)
- **Switch** → GPIO21 (with 10kΩ pull-up to +3.3V)
- **Common** → GND

**Debouncing:**
- Hardware: 100nF capacitor on each switch line to GND (optional)
- Software: Debounce in firmware (preferred)

**Pull-up resistors:**
- **R_ENC1A, R_ENC1B, R_ENC1SW**: 10kΩ, 0805 (qty 3)
- **R_ENC2A, R_ENC2B, R_ENC2SW**: 10kΩ, 0805 (qty 3)

#### Toggle Switches

**Toggle 1 (Ch1 Run/Stop):**
- **Common** → +3.3V
- **Output** → GPIO22 (with 10kΩ pull-down to GND)
- **NC** (not connected)

**Toggle 2 (Ch2 Run/Stop):**
- **Common** → +3.3V
- **Output** → GPIO23 (with 10kΩ pull-down to GND)
- **NC** (not connected)

**Logic:**
- Toggle ON (up) → GPIO reads HIGH (3.3V) → Running
- Toggle OFF (down) → GPIO reads LOW (GND via pull-down) → Stopped

**Pull-down resistors:**
- **R_TOG1, R_TOG2**: 10kΩ, 0805 (qty 2)

#### Dual-Color LEDs

**LED 1 (Ch1 status):**
- **Red anode** → 220Ω → GPIO6
- **Green anode** → 220Ω → GPIO7
- **Common cathode** → GND

**LED 2 (Ch2 status):**
- **Red anode** → 220Ω → GPIO8
- **Green anode** → 220Ω → GPIO9
- **Common cathode** → GND

**Current limiting resistors (assuming 3.3V, 10mA LED current):**
- **R_LED1R, R_LED1G, R_LED2R, R_LED2G**: 220Ω, 0805 (qty 4)
- Calculate: R = (3.3V - V_LED) / 10mA
  - Red LED: (3.3V - 2.0V) / 10mA = 130Ω → use 220Ω (safer, ~6mA)
  - Green LED: (3.3V - 3.0V) / 10mA = 30Ω → use 220Ω (dimmer, acceptable)

**LED control logic:**
- Running: GPIO drives Green HIGH (red LOW) → Green LED on
- Stopped: GPIO drives Red HIGH (green LOW) → Red LED on

---

### Block 5: Connectors (Sheet 6)

#### Input Jacks (with Normalling Option)

**Channel 1:**
- **J_IN1**: Thonkiconn PJ398SM
- **Tip** → Input signal (to protection circuit)
- **Switched contact** → Can be used for normalling (Ch2 normalized to Ch1 if unplugged)
- **Sleeve** → GND

**Channel 2:**
- **J_IN2**: Thonkiconn PJ398SM
- **Tip** → Input signal (or normalized from Ch1)
- **Switched contact** → Connected to J_IN1 tip (normalling)
- **Sleeve** → GND

**Normalling circuit** (optional for V1):
```
J_IN1 (tip) ───┬──> Ch1 input
               │
               └──> J_IN2 switched contact
                        │
J_IN2 (tip) ────────────┴──> Ch2 input
```

**Logic**: If J_IN2 unplugged, J_IN2 switched contact connects to J_IN1, duplicating signal to Ch2.

**Decision**: Implement normalling (useful for single-signal testing).

#### Output Jacks

**Channel 1:**
- **J_OUT1**: Thonkiconn PJ398SM
- **Tip** → Buffered Ch1 signal (from U1B)
- **Sleeve** → GND

**Channel 2:**
- **J_OUT2**: Thonkiconn PJ398SM
- **Tip** → Buffered Ch2 signal (from U1D)
- **Sleeve** → GND

#### Power Connector

**J_PWR**: 2×5 keyed IDC header (shrouded, vertical)

**Standard Eurorack 10-pin (2×5) power connector pinout:**
```
        RED STRIPE (−12V side)
  Pin 1: −12V    Pin 2: GND
  Pin 3: +5V     Pin 4: NC (or CV on some buses)
  Pin 5: GND     Pin 6: GND
  Pin 7: GND     Pin 8: +12V
  Pin 9: +12V    Pin 10: +5V (optional)
```

**Trace module power connections:**
- **Pin 1 (-12V)** → Reverse polarity protection → TL074 V-
- **Pins 2, 5, 6, 7 (GND)** → System ground
- **Pins 8, 9 (+12V)** → Buck regulator input, TL074 V+
- **Pins 3, 4, 10** → Not connected (NC)

**Notes:**
- Red stripe on ribbon cable marks -12V side (pin 1)
- Keyed shrouded header prevents backwards insertion
- Some buses provide CV/Gate on pins 4/10, but we don't use them

---

## Component Summary (BOM Preview)

### ICs
- **U_REG**: AP63203WU-7 (TSOT-26) or LMR33630ADDAR (SO PowerPAD-8) — qty 1
- **U1**: TL074IDR (SOIC-14) — qty 1 (Ch1 uses A/B, Ch2 shares same IC or use 2nd TL074)
- **U2**: TL074IDR (SOIC-14) — qty 1 (Ch2 uses C/D) — OR share U1 if pin count allows
- **MCU**: Raspberry Pi Pico (RP2040 module) — qty 1

**Note**: 1× TL074 can handle both channels (4 op-amps: Ch1 A/B, Ch2 C/D). Use 2× TL074 if layout cleaner.

### Diodes
- **D_POL**: Reverse polarity protection (SS14 or P-FET) — qty 1
- **D1, D2** (Ch1 input clamps): SS14 Schottky — qty 2
- **D1, D2** (Ch2 input clamps): SS14 Schottky — qty 2
- **D3, D4** (Ch1 ADC clamps): BAT85 Schottky — qty 2
- **D3, D4** (Ch2 ADC clamps): BAT85 Schottky — qty 2
- **Total diodes**: 9 (1 polarity + 8 signal)

### Resistors (0805, 1% metal film)
**Per channel (×2):**
- R_PROT: 1kΩ — qty 2
- R1: 620kΩ — qty 2
- R2: 100kΩ — qty 2
- R3: 91kΩ — qty 1 (shared offset reference)
- R4: 100kΩ — qty 1 (shared offset reference)
- R5: 100kΩ — qty 2
- R6: 100kΩ — qty 2
- R7: 1kΩ — qty 2

**Encoders:**
- R_ENC (10kΩ pull-ups): qty 6

**Toggles:**
- R_TOG (10kΩ pull-downs): qty 2

**LEDs:**
- R_LED (220Ω current limiting): qty 4

**Total resistors**: ~30 (many 100kΩ, can buy in bulk)

### Capacitors (0805 ceramic, X7R)
- **100nF**: ~10 (decoupling, bypass)
- **10µF**: 2-3 (power filtering)
- **22µF**: 1-2 (buck output)

### Inductors
- **L1**: 4.7µH-10µH, >2A (e.g., Bourns SRP series) — qty 1

### Connectors
- **J_PWR**: 2×5 keyed IDC header — qty 1
- **J_IN1, J_IN2**: Thonkiconn PJ398SM — qty 2
- **J_OUT1, J_OUT2**: Thonkiconn PJ398SM — qty 2

### Mechanical
- **SW_ENC1, SW_ENC2**: Bourns PEC11 encoders — qty 2
- **SW_TOG1, SW_TOG2**: SPDT toggles (C&K 7101) — qty 2
- **LED1, LED2**: Dual-color 5mm LEDs (red/green, common cathode) — qty 2

### Display
- **DISP**: 3.5" 480×320 SPI TFT (ILI9488) — qty 1

---

## Schematic Checklist

### Pre-Design
- [ ] Review all component values from docs/06-input-channel-schematic.md
- [ ] Verify RP2040 GPIO assignments (no conflicts)
- [ ] Confirm Eurorack power pinout (2×5 connector)
- [ ] Double-check buck regulator package (TSOT-26 or SO PowerPAD-8)

### Sheet 1: Power Supply
- [ ] Draw Eurorack power connector (J_PWR)
- [ ] Add reverse polarity protection (D_POL)
- [ ] Draw buck regulator circuit (U_REG + L1 + caps)
- [ ] Add ±12V decoupling for TL074 (100nF near ICs)
- [ ] Add +3.3V distribution and decoupling

### Sheet 2: Input Channel 1
- [ ] Draw input protection (R_PROT, D1, D2)
- [ ] Draw voltage divider (R1, R2)
- [ ] Draw offset reference (R3, R4, C1)
- [ ] Draw summing amplifier (U1A, R5, R6)
- [ ] Draw ADC protection (R7, D3, D4)
- [ ] Draw passthrough buffer (U1B)
- [ ] Connect to J_IN1, J_OUT1

### Sheet 3: Input Channel 2
- [ ] Duplicate Channel 1 circuit
- [ ] Use U1C (summing) and U1D (buffer)
- [ ] Implement normalling from Ch1 to Ch2 (via jack switching)
- [ ] Connect to J_IN2, J_OUT2

### Sheet 4: Microcontroller & Display
- [ ] Draw Raspberry Pi Pico module (or bare RP2040 if chosen)
- [ ] Connect ADC inputs (GPIO26, GPIO27)
- [ ] Connect SPI display interface (GPIO10-15)
- [ ] Add decoupling caps on +3.3V
- [ ] Add USB connector (if bare RP2040)
- [ ] Add SWD debug header

### Sheet 5: User Controls
- [ ] Draw Encoder 1 with pull-ups (GPIO16-18)
- [ ] Draw Encoder 2 with pull-ups (GPIO19-21)
- [ ] Draw Toggle 1 with pull-down (GPIO22)
- [ ] Draw Toggle 2 with pull-down (GPIO23)
- [ ] Draw LED 1 with current limiting (GPIO6-7)
- [ ] Draw LED 2 with current limiting (GPIO8-9)

### Sheet 6: Connectors
- [ ] Draw all jacks (J_IN1, J_IN2, J_OUT1, J_OUT2)
- [ ] Implement normalling circuit (Ch2 normalized to Ch1)
- [ ] Add labels and test points

### Final Review
- [ ] Check all power connections (no floating pins)
- [ ] Verify all ground connections (star ground for analog)
- [ ] Add net labels for inter-sheet connections
- [ ] Number all components (U1, U2, R1-R30, etc.)
- [ ] Run ERC (Electrical Rule Check) in KiCad
- [ ] Export BOM and verify against parts checklist

---

## KiCad Next Steps

1. **Create new KiCad project**: `trace-eurorack-module.kicad_pro`
2. **Set up hierarchical sheets** (6 sheets as outlined)
3. **Import custom symbols** if needed (Thonkiconn, PEC11, etc.)
4. **Draw each sheet** following the block diagrams above
5. **Run ERC** and fix all errors/warnings
6. **Export netlist** for PCB layout
7. **Create BOM** with part numbers from sourcing checklist

---

## Design Notes & Decisions

### GPIO Conflict Resolution
- **Original error**: GPIO26/27 assigned to both ADC and LEDs
- **Fix**: Moved LEDs to GPIO6-9 (were unused)
- **Final assignment**: GPIO26/27 dedicated to ADC inputs

### Offset Reference Sharing
- **Option A**: Separate R3/R4/C1 per channel (total isolation)
- **Option B**: Share single offset reference between channels (cost savings)
- **Decision**: Share single reference (adequate stability, fewer parts)

### TL074 Usage
- **Option A**: 1× TL074 (U1A/B for Ch1, U1C/D for Ch2)
- **Option B**: 2× TL074 (one per channel, cleaner layout)
- **Decision**: Try 1× TL074 first (fewer parts), split if routing difficult

### Normalling Implementation
- **Use**: Thonkiconn switched jacks
- **Logic**: Ch2 input jack switched contact connects to Ch1 signal when Ch2 unplugged
- **Benefit**: Easy single-channel testing (plug Ch1, get both traces)

### Buck Regulator Choice
- **AP63203WU-7** (TSOT-26): Smaller, adequate current (2A)
- **LMR33630ADDAR** (SO PowerPAD-8): Higher current (3A), thermal pad complexity
- **Recommendation**: AP63203 for the V1 (simpler layout, 2A sufficient)

---

## Open Questions

- [ ] Use 1 or 2 TL074 ICs? (layout-dependent)
- [ ] Share offset reference between channels? (yes, recommended)
- [ ] Add test points for debugging? (yes, at key nodes)
- [ ] Include mounting holes in schematic? (no, add in PCB layout)
- [ ] SWD debug header necessary? (yes, for firmware development)

---

## Ready for KiCad!

This document provides complete circuit block definitions. Next step: **Open KiCad and start drawing schematics** following this plan.
