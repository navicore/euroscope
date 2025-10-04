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

**D1, D2: Schottky Clamp Diodes**
- Part: **BAT54S** (dual Schottky in SOT-23) or discrete 1N5819
- D1: Clamps to +12V rail (cathode to +12V)
- D2: Clamps to -12V rail (anode to -12V)
- Forward voltage: ~0.3V (Schottky)
- Clamps input to +12.3V / -12.3V (safe overvoltage range)

**Alternative: TVS Diode**
- **SMBJ15A** (bidirectional 15V TVS)
- Clamps both polarities
- More robust but higher capacitance

**Why Schottky?**
- Fast response (important for audio signals)
- Low forward voltage (minimal signal clipping in normal operation)
- Cheap and available

---

### Stage 2: Voltage Divider (Attenuation)

**R1: 620kΩ, 1% metal film**
- Top resistor of divider

**R2: 100kΩ, 1% metal film**
- Bottom resistor of divider (to GND)

**Divider ratio**: (620k + 100k) ÷ 100k = 7.2:1

**Input impedance**: 620k + 100k = **720kΩ** (well above 100kΩ requirement)

**Voltage mapping**:
- Input +12V → 12V ÷ 7.2 = **+1.67V**
- Input 0V → **0V**
- Input -12V → -12V ÷ 7.2 = **-1.67V**

**Output range**: ±1.67V (bipolar, centered at 0V)

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

**Offset Voltage Reference** (+1.65V from +3.3V rail):
- **R3**: 100kΩ (from +3.3V regulated supply)
- **R4**: 100kΩ (to GND)
- **Voffset**: 3.3V × (100k ÷ (100k + 100k)) = 3.3V × 0.5 = **1.65V**

**C1: 100nF ceramic capacitor** (bypass cap on +1.65V reference node)
- Stabilizes offset voltage
- Filters noise from +3.3V supply

**Voltage summing math**:
- Signal: ±1.67V
- Offset: +1.65V
- Theoretical output: Signal + Offset = -0.02V to +3.32V

**ADC protection for negative voltage**:
- The calculated -0.02V is within resistor tolerance (1% on 620kΩ and 100kΩ)
- **D4 (Schottky diode to GND) clamps any negative excursion to ~0V**
- In practice: ADC input never goes below 0V due to clamp
- **ADC sees**: 0V to 3.3V ✓ (D4 ensures minimum is 0V, not -0.02V)

---

### Stage 4: ADC Protection (Final Safety Layer)

**R7: 1kΩ, 1/4W**
- Series resistor between op-amp output and RP2040 ADC pin
- Limits current if voltage exceeds ADC rails
- Works with ADC input clamp diodes (internal to RP2040)

**D3, D4: Schottky Clamp Diodes at ADC Input**
- D3: BAT54S to +3.3V (clamps high)
- D4: BAT54S to GND (clamps low - **critical for negative voltage protection**)
- Final protection if op-amp output somehow exceeds 0-3.3V range

**Why double protection?**
- Op-amps can output rail voltage (±12V in fault conditions)
- R7 + D3/D4 ensure ADC never sees >3.3V or <0V
- RP2040 ADC is fragile - **redundant protection is cheap insurance**

**Note on negative voltage**: The theoretical calculation shows -0.02V minimum (due to resistor tolerances), but D4 clamps this to 0V in practice. The ADC never sees negative voltage.

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
- R3: 100kΩ (offset reference top)
- R4: 100kΩ (offset reference bottom)
- R5: 100kΩ (signal summing)
- R6: 100kΩ (offset summing)
- R7: 1kΩ (ADC protection)

### Capacitors
- C1: 100nF ceramic (offset reference bypass)

### Diodes
- D1, D2: BAT54S or 1N5819 Schottky (input clamps to ±12V)
- D3, D4: BAT54S Schottky (ADC clamps to 3.3V/GND)

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
| Normal audio | ±5V | (5V ÷ 7.2) + 1.65V = 2.34V @ ADC | None (normal operation) |
| Hot CV | ±10V | (10V ÷ 7.2) + 1.65V = 3.04V @ ADC | None (within range) |
| Max negative | -12V | Calculated -0.02V, clamped to 0V @ ADC | D4 clamp (prevents negative) |
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

**BAT54S Pinout** (SOT-23):
```
    BAT54S (Dual Schottky)
       ___
   1 |   | 3  (Common Cathode at pin 3)
   2 |___|

   Pin 1: Anode 1
   Pin 2: Anode 2
   Pin 3: Common Cathode
```

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
- Diodes (4×): $0.40
- Caps: $0.10
- Jacks (2×): $1.00

**This is the heart of the module** - get this right and everything else follows.
