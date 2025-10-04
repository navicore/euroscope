# Analog Input Circuit - Resistor Calculations

## Design Goal

Map Eurorack input range (±12V) to RP2040 ADC range (0-3.3V)

## Voltage Mapping Requirements

| Input Voltage | ADC Voltage |
|--------------|-------------|
| -12V         | 0V          |
| 0V           | 1.65V       |
| +12V         | 3.3V        |

## Calculations

### Total Scaling Factor

Input range: 24V span (-12V to +12V)
Output range: 3.3V span (0V to 3.3V)

**Attenuation ratio**: 24V ÷ 3.3V = 7.27:1

**Offset needed**: +1.65V (to center 0V input at mid-ADC)

### Resistor Divider Design

We need two things:
1. Divide input by 7.27
2. Add 1.65V offset

#### Approach: Voltage Divider with Dual Supply

Using ±12V power rails from Eurorack bus:

```
        +12V
         |
        R1 (offset from +12V rail)
         |
         +---- To Op-Amp Buffer ---- ADC
         |
    Vin ----R2 (input resistor)
         |
        R3 (to -12V rail)
         |
        -12V
```

This is a **summing network** that:
- Attenuates the input signal
- References to both rails to create offset
- Maintains high input impedance

### Final Component Values

### Input Voltage Divider (Stage 1)
- **R1**: 620kΩ, 1% metal film
- **R2**: 100kΩ, 1% metal film
- **Input impedance**: 720kΩ
- **Attenuation**: 7.2:1

### Offset Reference (Stage 2)
- **R3**: 100kΩ, 1% metal film (from +3.3V)
- **R4**: 100kΩ, 1% metal film (to GND)
- **Offset voltage**: 1.65V

### Summing Resistors (Stage 2)
- **R5**: 100kΩ (signal path to op-amp)
- **R6**: 100kΩ (offset path to op-amp)
- Equal values → unity gain summing

### Current Limiting (ADC Protection)
- **R7**: 1kΩ (between op-amp output and ADC input)

## Verification

| Input | After Divider | After Offset | To ADC |
|-------|---------------|--------------|--------|
| -12V  | -1.67V        | -0.02V       | ~0V ✓  |
| 0V    | 0V            | +1.65V       | 1.65V ✓|
| +12V  | +1.67V        | +3.32V       | 3.3V ✓ |

Small error at extremes is acceptable - ADC has 12-bit resolution (0.8mV steps), and we're within spec.

## Selected Approach: Simple Two-Stage Design ✓

**Stage 1**: Voltage divider (attenuation)
**Stage 2**: Op-amp offset/buffer

Benefits:
- Clear signal flow
- Easy to calculate and verify
- Each stage testable independently
- Simple to troubleshoot and tune
- Better for learning

### Stage 1: Voltage Divider (Attenuation)

Divide ±12V input → ±1.65V

**Divider ratio**: 12V ÷ 1.65V = 7.27:1

Using standard resistor values:
- **R1 (top)**: 620kΩ
- **R2 (bottom)**: 100kΩ

**Actual ratio**: (620k + 100k) ÷ 100k = 7.2:1

**Output voltage**:
- Input +12V → 12V ÷ 7.2 = **1.67V** ✓ (close enough to 1.65V)
- Input -12V → -12V ÷ 7.2 = **-1.67V** ✓
- Input 0V → **0V**

**Input impedance**: R1 + R2 = 620k + 100k = **720kΩ** ✓ (well above 100kΩ target)

### Stage 2: Offset & Buffer

Need to shift ±1.67V → 0-3.3V range (add +1.65V offset)

**Op-amp non-inverting summing amplifier**:
- Input: ±1.67V signal from divider
- Reference: +1.65V offset (from resistor divider off +3.3V rail)
- Output: 0-3.3V to ADC

**Offset voltage source** (from +3.3V rail):
- **R3**: 100kΩ (top, from +3.3V)
- **R4**: 100kΩ (bottom, to GND)
- **Voffset**: 3.3V × (100k ÷ 200k) = **1.65V** ✓

**Op-amp configuration**: Voltage follower with offset
- Signal input and offset input summed through resistors into non-inverting input
- Unity gain buffer (follower)

## Protection Components

**Schottky diode clamps** at ADC input:
- D1: Anode to ADC pin, Cathode to +3.3V (clamps high)
- D2: Cathode to ADC pin, Anode to GND (clamps low)
- **Part**: BAT54S (dual Schottky, SOT-23)

**Series resistor** before clamps (current limiting):
- 1kΩ between op-amp output and ADC input
- Limits current if voltage exceeds rails

## Power Consumption

**From +3.3V rail** (offset reference divider):
- Current: 3.3V ÷ 200kΩ = 16.5µA (negligible)

**From input signal** (worst case):
- Current at +12V input: 12V ÷ 720kΩ = 16.7µA (negligible)

**Op-amp quiescent current**: ~1-5mA per channel (depends on op-amp choice)

**Total per channel**: ~5mA (dominated by op-amp, not resistors)

## Next Steps

- [x] Choose simple two-stage approach
- [x] Calculate resistor values
- [x] Verify input impedance >100kΩ (720kΩ ✓)
- [x] Select standard E96 resistor values
- [ ] Draw complete schematic for one channel
- [ ] Breadboard/simulate to verify
- [ ] Select op-amp part numbers

## Notes

- All resistors: **1% tolerance** metal film for accuracy
- Consider **0.1% matched resistor networks** if offset accuracy critical
- May need **trimmer potentiometer** for fine offset calibration (can tune out op-amp offset voltage)
