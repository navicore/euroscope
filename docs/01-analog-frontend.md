# Analog Front-End Design

## Requirements

- **Input range**: ±12V (safe handling)
- **Typical signals**: ±5V audio, 0-8V envelopes, ±8V LFOs
- **Input impedance**: >100kΩ (no source loading)
- **Coupling**: DC (see CV offsets and slow modulation)
- **ADC target**: 0-3.3V (RP2040 compatible)
- **Passthrough**: Buffered, unmodified signal

## Signal Chain (Per Channel)

```
Input Jack → Protection → Voltage Divider → Buffer → ADC (0-3.3V)
                             ↓
                          Buffer → Output Jack (passthrough)
```

## Stage Breakdown

### Stage 1: Input Protection

**Purpose**: Protect against overvoltage and mis-patching

**Implementation**: Schottky diode clamps to ±12V rails
- **Diodes**: BAT54S or 1N4148
- Clamps to slightly beyond ±12V (failsafe only)
- Not part of normal signal path

### Stage 2: Voltage Scaling

**Purpose**: Map ±12V input range → 0-3.3V ADC range

**Voltage mapping**:
- Input: -12V → ADC: 0V
- Input: 0V   → ADC: 1.65V (center)
- Input: +12V → ADC: 3.3V

**Divider ratio**: ÷7.27 + 1.65V offset

**Resistor divider** (values TBD):
- Sets input impedance
- Provides attenuation
- Creates offset voltage

### Stage 3: Buffer (Input Side)

**Purpose**:
- Isolate input from ADC loading
- Drive passthrough output
- High Z input, low Z output

**Op-amp configuration**: Voltage follower (unity gain)

**Op-amp choice**:
- Dual supply (±12V from Eurorack bus)
- Rail-to-rail I/O
- Low noise for audio
- **Candidates**: TL072 (classic), OPA2134 (premium)

### Stage 4: Passthrough Buffer

**Purpose**:
- Buffered copy for daisy-chaining/mult
- No loading of main signal path
- Isolated from downstream modules

**Implementation**: Second unity-gain buffer from input buffer output

## Power Supply Strategy

**Op-amps**: Powered by Eurorack ±12V directly
- Symmetric supply for clean bipolar signal handling
- No virtual ground tricks needed

**Digital**: +3.3V regulated from +12V via LDO
- Powers RP2040 and display
- Isolated from analog supply (minimize noise)

## Design Constraints

1. **No signal modification**: Passthrough must be bit-perfect copy (within op-amp specs)
2. **No loading**: >100kΩ input impedance
3. **Protection**: Safe with any Eurorack signal
4. **Accuracy**: 1% resistors for divider precision

## Next Steps

- [ ] Calculate exact resistor divider values
- [ ] Select specific op-amp part numbers
- [ ] Draw schematic for one channel
- [ ] Validate voltage scaling math
- [ ] Consider AC coupling option (future enhancement)
