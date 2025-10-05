# Input Channel Schematic

## Design Philosophy: Protection First

**Modular synthesis is experimental** - users patch freely without worrying about signal routing. Our circuit must:
- Survive any Eurorack signal on any input (±12V worst case)
- Protect the RP2040 ADC (which dies instantly above 3.3V)
- Never load down signal sources
- Pass clean signals to outputs

---

## Single Input Channel - Complete Signal Path

```
INPUT JACK                PROTECTION           ATTENUATION         OFFSET/BUFFER              ADC
   ○────┬────────────┬────[R_PROT]────┬────[R1: 620k]────┬────[R5: 100k]────┬────[R7: 1k]────> RP2040 ADC
        │            │                │                  │                  │                  (0-3.3V)
        │         [DIODES]            │                  │                  │
        │         (clamp)             │                  │              [U1A: TL074]
        │                             │                  │              Non-inv summer
        │                          [R2: 100k]         [R6: 100k]           │
        │                             │                  │                  │
        │                            GND              +1.65V ref           GND
        │                                            (from +3.3V)
        │
        │         PASSTHROUGH BUFFER
        └────────────[U1B: TL074]────────────> OUTPUT JACK ○
                     Voltage follower
```

---

## Component Details

### Stage 1: Input Protection (Critical!)

**R_PROT: Series Protection Resistor**
- Value: **1kΩ**, 1/4W
- Purpose: Current limiting before clamp diodes
- Limits fault current to safe levels

**D1, D2: Discrete Schottky Clamp Diodes**
- Part: **SS14** (SMA/DO-214AC SMD) or **1N5819** (DO-41 through-hole)
- D1: Anode to signal input, Cathode to +12V rail (clamps high)
- D2: Cathode to signal input, Anode to -12V rail (clamps low)
- Forward voltage: ~0.3-0.4V (Schottky)
- Clamps input to +12.3V / -12.3V (safe overvoltage range)

**Why discrete diodes (not dual packages)?**
- Need opposite polarities (one up, one down)
- BAT54S is series configuration (both cathodes common) - won't work for bidirectional clamp
- Discrete diodes give correct polarity for each clamp direction
- Easy to verify in schematic

**Why Schottky?**
- Fast response (important for audio signals)
- Low forward voltage (minimal signal clipping in normal operation)
- Cheap and available (~$0.10 each)

---

### Stage 2: Voltage Divider (Attenuation)

**R1: 620kΩ, 1% metal film**
- Top resistor of divider

**R2: 100kΩ, 1% metal film**
- Bottom resistor of divider (to GND)

**Voltage attenuation calculation**:
- Output voltage: Vout = Vin × (R2 ÷ (R1 + R2))
- Attenuation factor: 100k ÷ (620k + 100k) = 100k ÷ 720k = 1/7.2
- **Voltage division ratio: 7.2:1** (input is divided by 7.2)

**Input impedance**: R1 + R2 = 620k + 100k = **720kΩ** (well above 100kΩ requirement)

**Voltage mapping (nominal)**:
- Input +12V → 12V × (1/7.2) = **+1.67V**
- Input 0V → **0V**
- Input -12V → -12V × (1/7.2) = **-1.67V**

**Output range (nominal)**: ±1.67V (bipolar, centered at 0V)

---

### Stage 3: Offset & Buffer (Convert to ADC Range)

**U1A: TL074 Op-Amp (1/4 of quad package)**
- Configuration: Non-inverting summing amplifier (unity gain)
- Powered by ±12V Eurorack rails
- Inputs: Signal (±1.67V) + Offset (+1.65V)
- Output: 0-3.34V (ADC compatible)

**R5: 100kΩ, 1% metal film**
- Signal path resistor to op-amp (+) input

**R6: 100kΩ, 1% metal film**
- Offset voltage path resistor to op-amp (+) input

**Equal resistors → Unity gain summing**

**Offset Voltage Reference** (+1.73V from +3.3V rail):
- **R3**: 91kΩ, 1% metal film (from +3.3V regulated supply)
- **R4**: 100kΩ, 1% metal film (to GND)
- **Voffset (nominal)**: 3.3V × (100k ÷ (91k + 100k)) = 3.3V × 0.524 = **1.73V**

**C1: 100nF ceramic capacitor** (bypass cap on offset reference node)
- Stabilizes offset voltage
- Filters noise from +3.3V supply

**Voltage summing math (nominal)**:
- Signal: ±1.67V
- Offset: +1.73V
- Output: Signal + Offset = **0.06V to 3.40V**

**Worst-case tolerance analysis** (ensures positive voltage even with component variation):

*Divider worst case (max negative signal):*
- R1_min = 613.8kΩ, R2_max = 101kΩ
- Max attenuation: -12V × (101k ÷ 714.8k) = **-1.70V**

*Offset worst case (min offset):*
- R3_max = 91.91kΩ, R4_min = 99kΩ
- Min offset: 3.3V × (99k ÷ 190.91k) = **1.71V**

*Combined worst case:*
- **Output: 1.71V - 1.70V = +0.01V** ✓ (positive even with worst-case tolerances!)

**ADC protection**:
- Designed to stay positive even with 1% resistor tolerances
- D4 clamp to GND provides backup protection if calculation wrong
- **ADC sees**: 0.01V to 3.40V (nominal: 0.06V to 3.40V) ✓

---

### Stage 4: ADC Protection (Final Safety Layer)

**R7: 1kΩ, 1/4W**
- Series resistor between op-amp output and RP2040 ADC pin
- Limits current if voltage exceeds ADC rails
- Works with ADC input clamp diodes (internal to RP2040)

**D3, D4: Discrete Schottky Clamp Diodes at ADC Input**
- Part: **SS14** (SMA SMD), **1N5819** (DO-41 TH), or **BAT85** (SOD-123 SMD)
- D3: Anode to ADC input, Cathode to +3.3V (clamps high)
- D4: Cathode to ADC input, Anode to GND (clamps low - **backup protection**)
- Final protection if op-amp output somehow exceeds 0-3.3V range

**Why double protection?**
- Op-amps can output rail voltage (±12V in fault conditions)
- R7 + D3/D4 ensure ADC never sees >3.3V or <0V
- RP2040 ADC is fragile - **redundant protection is cheap insurance**

**Note on voltage range**: Circuit is designed to stay 0.01V-3.40V even with worst-case 1% resistor tolerances. D4 clamp provides backup if tolerances stack worse than calculated.

---

### Stage 5: Passthrough Buffer

**U1B: TL074 Op-Amp (2/4 of quad package)**
- Configuration: Voltage follower (unity gain buffer)
- Input: Directly from input jack (after R_PROT and clamp diodes)
- Output: Buffered copy to output jack

**Purpose**:
- High input impedance (doesn't load signal source)
- Low output impedance (can drive downstream modules)
- Acts as clean mult/buffered passthrough
- Signal is **unmodified** (just impedance conversion)

**No additional components needed** - simple voltage follower.

---

## Power Supply Connections

**U1 (TL074) Power Pins**:
- Pin 4 (V-): -12V from Eurorack bus
- Pin 11 (V+): +12V from Eurorack bus
- **Decoupling**: 100nF ceramic capacitors on each rail, close to IC
  - C_V+: 100nF from +12V to GND (pin 11)
  - C_V-: 100nF from -12V to GND (pin 4)

**+3.3V Regulated Supply** (for offset reference):
- Generated by buck regulator from +12V bus
- Used for R3/R4 offset divider
- Separate decoupling: 10µF + 100nF near reference circuit

---

## Component Summary (Per Channel)

### Resistors (1% metal film)
- R_PROT: 1kΩ (protection)
- R1: 620kΩ (divider top)
- R2: 100kΩ (divider bottom)
- **R3: 91kΩ** (offset reference top) ← Updated for tolerance margin
- R4: 100kΩ (offset reference bottom)
- R5: 100kΩ (signal summing)
- R6: 100kΩ (offset summing)
- R7: 1kΩ (ADC protection)

### Capacitors
- C1: 100nF ceramic (offset reference bypass)

### Diodes (discrete Schottky)
- **D1: SS14 (SMD)** or **1N5819 (TH)** Schottky (input clamp to +12V) ← Changed from BAT54S
- **D2: SS14 (SMD)** or **1N5819 (TH)** Schottky (input clamp to -12V) ← Changed from BAT54S
- **D3: SS14/1N5819/BAT85** Schottky (ADC clamp to +3.3V) ← Changed from BAT54S
- **D4: SS14/1N5819/BAT85** Schottky (ADC clamp to GND) ← Changed from BAT54S

### ICs
- U1A: TL074 op-amp (offset/buffer stage)
- U1B: TL074 op-amp (passthrough buffer)

### Connectors
- J_IN: Thonkiconn PJ398SM (input jack)
- J_OUT: Thonkiconn PJ398SM (output jack)

---

## Two-Channel Implementation

**Channel 1**:
- Uses U1A (pin 1-2-3) for offset/buffer
- Uses U1B (pin 5-6-7) for passthrough buffer
- ADC input: RP2040 GPIO26 (ADC0)

**Channel 2**:
- Uses U1C (pin 8-9-10) for offset/buffer
- Uses U1D (pin 12-13-14) for passthrough buffer
- ADC input: RP2040 GPIO27 (ADC1)

**One TL074 handles both channels (4 op-amps total, 1 IC)**

**Shared components**:
- +3.3V offset reference (R3, R4, C1) can be shared between channels
- Power supply decoupling shared

---

## Protection Test Cases

| Scenario | Input Voltage | Result | Protection Active |
|----------|---------------|--------|-------------------|
| Normal audio | ±5V | (5V ÷ 7.2) + 1.73V = 2.42V @ ADC | None (normal operation) |
| Hot CV | ±10V | (10V ÷ 7.2) + 1.73V = 3.12V @ ADC | None (within range) |
| Max negative | -12V | (−12V ÷ 7.2) + 1.73V = 0.06V @ ADC | None (positive by design) |
| Worst tolerance | -12V + worst case | 1.71V - 1.70V = 0.01V @ ADC | Tolerance margin holds |
| Over-voltage | +15V | Clamped to +12.3V by D1 | R_PROT + D1 |
| Reverse voltage | -15V | Clamped to -12.3V by D2 | R_PROT + D2 |
| Op-amp fault | U1A outputs +12V | ADC sees 3.3V (clamped by D3) | R7 + D3 |
| Op-amp fault | U1A outputs -12V | ADC sees 0V (clamped by D4) | R7 + D4 |

**ADC survives all cases** ✓

---

## PCB Layout Considerations

**Critical paths**:
1. Input jack → Protection → Divider (keep traces short to minimize pickup)
2. Op-amp output → ADC (guard with GND pour, minimize noise coupling)

**Grounding**:
- Star ground for analog section
- Separate digital ground for RP2040
- Single point connection between analog/digital grounds

**Decoupling placement**:
- 100nF caps **as close as possible** to IC power pins
- Use short, direct traces to IC (not shared via traces)

**Component placement**:
- Protection diodes immediately after input jack
- Offset reference (R3/R4/C1) near op-amp
- Keep high-impedance nodes (after R1/R2 divider) short

---

## Schematic Symbol Reference

**TL074 Pinout** (DIP-14 or SOIC-14):
```
      TL074 (Quad Op-Amp)
           ___
  OUT1  1 |   | 14  OUT4
   IN- 1 2 |   | 13  IN- 4
   IN+ 1 3 |   | 12  IN+ 4
   V-    4 |   | 11  V+
   IN+ 2 5 |   | 10  IN+ 3
   IN- 2 6 |   |  9  IN- 3
  OUT2  7 |   |  8  OUT3
          |___|
```

**Schottky Diode Options**:

**SS14/SS16 (SMA/DO-214AC SMD)**:
- Mark indicates cathode (line/band on package)
- Anode: Unmarked end
- Cathode: Marked end (line/band)
- Forward voltage: ~0.4V @ 1A
- Package: 4.5-5.3mm × 2.5-2.8mm × 2.0-2.3mm

**1N5819 (DO-41 through-hole)**:
- Axial package with cathode band
- Forward voltage: ~0.3V @ 1A
- Use if preferring through-hole assembly

**BAT85 (SOD-123 SMD)**:
- Smaller SMD option (200mA rating)
- Adequate for ADC protection only

**Connection reference**:
- To clamp HIGH: Anode to signal, Cathode to positive rail
- To clamp LOW: Cathode to signal, Anode to ground/negative rail

---

## Next Steps

1. Draw complete schematic in KiCad or similar EDA tool
2. Add power supply section (buck regulator, Eurorack connector)
3. Add RP2040 section (MCU, crystal, flash, USB)
4. Add display interface (SPI connections)
5. Add encoder/button inputs (with debouncing)
6. Perform full design review before PCB layout

---

## Design Notes

**Why this protection strategy?**
- **Multiple layers**: Input clamps, current limiting, ADC clamps
- **Redundant**: If one stage fails, others protect ADC
- **Proven**: Similar approach used in many Eurorack modules (Mutable Instruments, etc.)
- **Cheap**: <$1 in protection parts per channel

**Cost per channel**: ~$2.50
- TL074 (half): $0.25
- Resistors (8×): $0.80
- Diodes (4× SS14 or 1N5819): $0.40
- Caps: $0.10
- Jacks (2×): $1.00

**Component changes from initial design**:
- ❌ Removed BAT54S (wrong configuration for bidirectional clamp)
- ✓ Using discrete Schottky diodes: SS14 (SMD) or 1N5819 (through-hole)
- ✓ Changed R3: 100kΩ → 91kΩ (ensures positive voltage with tolerances)

**This is the heart of the module** - get this right and everything else follows.
