# Parts Selection

## Op-Amps

### Requirements
- Dual supply (±12V operation)
- Rail-to-rail I/O (preferred)
- Low noise for audio signals
- Low offset voltage (<5mV)
- Dual or quad package (need 2-3 per channel × 2 channels)

### Candidates

#### TL072 (Classic Choice)
- **Type**: Dual JFET op-amp
- **Supply**: ±5V to ±18V ✓
- **Noise**: 18nV/√Hz (good)
- **Offset**: 3mV typical
- **Package**: DIP-8, SOIC-8
- **Cost**: ~$0.50
- **Pros**: Proven in audio, cheap, available everywhere
- **Cons**: Not rail-to-rail output

#### TL074 (Quad version)
- Same as TL072 but 4 op-amps in one package
- **Package**: DIP-14, SOIC-14
- **Better for this design**: Yes (need 3 op-amps per channel)

#### OPA2134 (Premium Audio)
- **Type**: Dual FET op-amp
- **Supply**: ±2.5V to ±18V ✓
- **Noise**: 8nV/√Hz (excellent)
- **Offset**: 0.5mV typical (very low)
- **Package**: DIP-8, SOIC-8
- **Cost**: ~$4
- **Pros**: Excellent audio specs, low distortion
- **Cons**: More expensive

#### MCP6002 (Single Supply Alternative)
- **Type**: Dual CMOS op-amp
- **Supply**: 1.8V to 6V (single supply only)
- **Rail-to-rail**: Yes
- **Cost**: ~$0.60
- **Note**: Would require different circuit (single supply design)

### Recommendation: **TL074** ✓

**Rationale**:
- Quad package = 1 IC per channel (clean layout)
- Proven in countless Eurorack modules
- Good audio performance
- Cheap and available
- DIP-14 available for breadboard prototyping
- SOIC-14 for final PCB

**Pinout**: Standard 14-pin quad op-amp
- Powered by ±12V from Eurorack bus
- 4 op-amps: Input buffer, passthrough buffer, summing amp, spare

---

## Display

### Requirements
- 4.3" diagonal (fits 24HP)
- SPI interface
- 3.3V compatible
- Non-touch
- Good availability

### Candidates

#### 4.3" 480×272 SPI TFT
- **Resolution**: 480×272 pixels (WQVGA)
- **Controller**: ILI9486 or similar
- **Interface**: SPI (4-wire)
- **Voltage**: 3.3V logic, backlight varies
- **Power**: ~150-200mA total
- **Cost**: $15-25
- **Availability**: AliExpress, Amazon, Adafruit

#### Specific Module Examples

**Leading Candidate: Waveshare 4.3inch 480×272 Touch LCD (B)**
- **Part**: 4.3inch 480×272 Touch LCD (B)
- **Resolution**: 480×272 pixels
- **Interface**: SPI + 24-bit RGB
- **Touch**: Resistive (can be ignored/unused for our application)
- **Link**: https://www.waveshare.com/4.3inch-480x272-touch-lcd-b.htm
- **Status**: Physical dimensions need verification against 24HP panel constraints
- **Concern**: Has touch screen (not preferred - we chose tactile controls). Touch can be left disconnected, but adds cost and complexity. Prefer true non-touch alternative if available.

**Other Options**:
- **Generic 4.3" ILI9486 SPI modules** from AliExpress
- Various Adafruit 4.3" TFT options (need specific part numbers)

### Recommendation: **Pending panel design** ⚠️

**Selection criteria**:
- Physical PCB dimensions must fit 24HP panel (active area < 110mm wide)
- Mounting hole spacing must align with panel design
- SPI interface compatible with RP2040
- Driver support in Rust embedded-graphics ecosystem
- Non-touch or touch-optional (we're using encoders/buttons)

**Critical blocker**: Cannot finalize display selection until panel layout is designed. Display dimensions drive panel cutout and component placement.

**Next steps**:
1. Design panel layout (see panel design decisions doc)
2. Measure available space for display
3. Research non-touch 4.3" SPI displays (preferred)
4. Verify Waveshare 4.3" or find alternative that fits
5. Order sample for physical fit verification

**Note**: Display selection deferred until panel design phase. Paying for unused touch capability is acceptable if no better option exists, but non-touch display preferred for cost and simplicity.

---

## Microcontroller

### Selected: **RP2040** ✓

**Development Board Options**:

#### Raspberry Pi Pico
- **Cost**: $4
- **Form factor**: Through-hole DIP
- **Pros**: Cheap, breadboard-friendly, widely available
- **Cons**: Larger footprint for final PCB

#### RP2040 Stamp / Tiny2040
- **Cost**: $6-10
- **Form factor**: Castellated SMD module
- **Pros**: Compact, easy to solder to custom PCB
- **Cons**: Slightly more expensive

#### Bare RP2040 IC
- **Cost**: $1-2
- **Package**: QFN-56
- **Pros**: Smallest, cheapest (in volume)
- **Cons**: Requires crystal, flash chip, USB circuit on PCB (more complex)

### Recommendation: **Raspberry Pi Pico** for prototype, **RP2040 module** for final

**Rationale**:
- Prototype: Use Pico on breadboard or socketed on PCB
- Final: Castellated RP2040 module soldered to custom PCB (cleaner)
- Avoid bare chip complexity for first module build

---

## Rotary Encoders

### Requirements
- Mechanical rotary encoder (not potentiometer)
- Quadrature output (A/B phases)
- Push switch integrated
- Panel mount
- Smooth feel (not clicky detents - or detents okay too)

### Candidates

#### Bourns PEC11 Series
- **Type**: Incremental encoder with switch
- **Detents**: 24 per revolution
- **Shaft**: 6mm diameter, D-shaft or knurled
- **Mounting**: Panel mount, threaded bushing
- **Cost**: $2-4
- **Availability**: Mouser, Digikey, Amazon
- **Pros**: Industry standard, reliable, good feel

#### Alps EC11 / EC12
- **Type**: Incremental encoder with switch
- **Detents**: 24 per revolution (EC11) or smooth (EC12)
- **Similar**: to Bourns, slightly different dimensions
- **Cost**: $2-3

### Recommendation: **Bourns PEC11** or **Alps EC11** ✓

**Quantity needed**: 2 encoders

**Selection**:
- Part number: PEC11R-4215F-S0024 (or similar)
- Verify shaft length works with panel thickness + knob

---

## Pushbuttons

### Requirements
- Momentary SPST
- Panel mount
- LED optional (could be useful for Run/Stop indicator)
- Tactile click

### Candidates

#### Standard 6mm Tactile Switch (PCB mount)
- **Mounting**: PCB mount, actuator extends through panel hole
- **Cost**: $0.10-0.50
- **Pros**: Cheap, tiny, standard
- **Cons**: No LED, requires precise panel alignment

#### Panel Mount Momentary Pushbutton (e.g., E-Switch TL1105)
- **Mounting**: Snap-in panel mount
- **Cost**: $1-2
- **Pros**: Easy panel mounting, good feel
- **Cons**: Larger footprint

#### Illuminated Panel Mount Pushbutton
- **Mounting**: Panel mount, threaded or snap-in
- **LED**: Integrated (useful for status indication)
- **Cost**: $2-5
- **Pros**: Status feedback, professional look
- **Cons**: More expensive, more wiring

### Recommendation: **Illuminated panel mount buttons** ✓

**Rationale**:
- Run/Stop button benefits from LED (red = stopped, green = running)
- Trigger button could show trigger status
- Professional look for a scope module
- Worth the extra $6-10 for UX improvement

**Quantity needed**: 2-3 buttons

---

## Jacks (3.5mm)

### Requirements
- Eurorack standard: 3.5mm (1/8") mono jacks
- Panel mount, switched or non-switched
- Thonkiconn style (Eurorack standard)

### Recommendation: **Thonkiconn (PJ398SM)** ✓

- **Type**: 3.5mm mono jack, switched (normalling)
- **Mounting**: Panel mount, hex nut
- **Cost**: $0.50-1.00
- **Availability**: Thonk, Modular Addict, Mouser
- **Standard**: De facto Eurorack standard

**Quantity needed**: 4 jacks (2 input, 2 passthrough output)

**Note**: Switched jacks useful if you want normalling behavior (e.g., Ch2 input normalized to Ch1)

---

## Power Connector

### Eurorack Standard: **Shrouded 2×5 IDC header** ✓

- **Type**: 10-pin (2×5) **keyed shrouded header** (mandatory)
- **Orientation**: Keyed for correct insertion (prevents reverse polarity)
- **Pins**: +12V, -12V, GND, +5V (optional), Gate/CV (unused)
- **Cost**: $0.50-1.00
- **Part**: Molex 70553 or equivalent (with keying tab)
- **Cable**: Use keyed ribbon cable (red stripe = -12V)

**Quantity needed**: 1 per module

**Critical**: The shroud key prevents backwards insertion. Never use non-keyed headers - reverse polarity will destroy the module instantly.

---

## Voltage Regulator (3.3V)

### Requirements
- Input: +12V (from Eurorack bus)
- Output: +3.3V, >500mA (for RP2040 + display)
- Low dropout (LDO) or switching regulator

### Candidates

#### LM1117-3.3 (LDO)
- **Type**: Linear LDO
- **Output**: 3.3V, 800mA
- **Dropout**: ~1.2V
- **Cost**: $0.50
- **Pros**: Simple, cheap, low noise
- **Cons**: Wastes power as heat ((12V - 3.3V) × >500mA ≈ >4.35W!)

**Problem**: Too much heat dissipation for this application ❌

**Thermal analysis**:
- LM1117 in typical small package (SOT-223/TO-220) can handle ~1W without heatsink
- 4.35W dissipation would require **large heatsink** (impractical in Eurorack)
- Even with heatsink: wasteful, generates heat in rack, loads +12V bus unnecessarily
- **Decision**: Linear regulators are not suitable for this power level

#### AMS1117-3.3 (LDO)
- Similar to LM1117, same heat problem ❌

#### AP63203 (Switching Buck Regulator)
- **Type**: Synchronous buck converter
- **Output**: 3.3V, 2A
- **Efficiency**: ~85-90%
- **Cost**: $1-2
- **Package**: SOIC-8
- **Pros**: Efficient, minimal heat, integrated switches
- **Cons**: Requires inductor and caps (more components)

#### LMR33630 (Buck Regulator)
- **Type**: Synchronous buck
- **Output**: 3.3V, 3A
- **Efficiency**: ~90%
- **Cost**: $2-3
- **Pros**: Very efficient, built-in compensation
- **Cons**: Slightly more expensive

### Recommendation: **AP63203 or similar buck regulator** ✓

**Rationale**:
- Power consumption: 500mA × 3.3V = 1.65W output
- With 85% efficiency: input power ~1.94W, dissipates only **~0.3W as heat**
- Draws ~160mA from +12V bus (acceptable for Eurorack)
- No heatsink required
- More components than LDO, but necessary for thermal management

**Comparison to LDO approach**:
| Parameter | LDO (LM1117) | Buck (AP63203) |
|-----------|--------------|----------------|
| Heat dissipated | 4.35W | ~0.3W |
| Heatsink needed | Large (impractical) | None |
| +12V current | 500mA | 160mA |
| Component count | Low | Medium |
| **Verdict** | ❌ Not feasible | ✓ Correct choice |

**TODO**: Select specific part and design buck circuit

---

## Protection Diodes

### Schottky Diodes (Input and ADC Clamps)

**Part**: Schottky Diode - Multiple Options

**SMD Option (Recommended)**:
- **Part Number**: SS14 or SS16
- **Package**: SMA (DO-214AC)
- **Voltage**: SS14: 40V, SS16: 60V
- **Current**: 1A forward
- **Forward voltage**: ~0.4V @ 1A
- **Cost**: ~$0.10 each
- **Quantity**: 8 total (4 per input channel: 2 for input clamps, 2 for ADC clamps)

**Through-Hole Option**:
- **Part Number**: 1N5819
- **Package**: DO-41 (axial lead)
- **Specs**: 40V, 1A, ~0.3V forward drop
- **Use if**: Prefer through-hole assembly

**Note**: Do not specify "1N5819 SMA" - this creates BOM ambiguity. Use SS14/SS16 for SMD.

**Why discrete diodes (not dual packages)?**
- Need opposite polarities for bidirectional clamping
- BAT54S is series configuration (won't work for our application)
- Discrete diodes provide correct polarity for each clamp direction

### Reverse Polarity Protection (Power Input)

**Part**: 1N5819 or similar Schottky
- **Type**: Power Schottky diode (or use P-channel MOSFET for lower drop)
- **Voltage**: 40V
- **Current**: 1A
- **Package**: DO-41 or SMD
- **Cost**: $0.20

**Alternative**: Use TVS diode or polyfuse for protection

---

## Passives Summary

### Resistors
- **1% metal film**, standard E96 values
- **Values needed**:
  - **100kΩ**: Op-amp feedback and input resistors, voltage dividers (many)
  - **620kΩ**: Voltage divider for input attenuation (2)
  - **1kΩ**: LED current limiting and signal line protection (2+)
- **Package**: 0805 SMD or 1/4W through-hole

### Capacitors
- **100nF ceramic** (0.1µF) - decoupling (many needed)
- **10µF electrolytic** - power supply filtering
- **22µF ceramic/tantalum** - buck regulator output
- **Package**: 0805 SMD (ceramic), radial (electrolytic)

---

## Next Steps

- [ ] Finalize display part number (verify dimensions)
- [ ] Design buck regulator circuit (or find reference design)
- [ ] Create complete BOM with part numbers and sources
- [ ] Estimate total cost

## Cost Estimate (Rough)

| Component | Qty | Unit Cost | Total |
|-----------|-----|-----------|-------|
| TL074 op-amp | 2 | $0.50 | $1.00 |
| RP2040 Pico | 1 | $4.00 | $4.00 |
| 4.3" Display | 1 | $20.00 | $20.00 |
| Encoders | 2 | $3.00 | $6.00 |
| Buttons (LED) | 3 | $3.00 | $9.00 |
| Jacks | 4 | $1.00 | $4.00 |
| Buck regulator | 1 | $2.00 | $2.00 |
| Passives/misc | - | - | $5.00 |
| PCB | 1 | $10-20 | $15.00 |
| Panel | 1 | $10-30 | $20.00 |
| **Total** | | | **~$86** |

**Note**: Excludes tools, shipping, mistakes/revisions. Double for prototype safety margin → **~$150-200** realistic first build cost.
