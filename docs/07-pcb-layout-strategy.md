# PCB Layout Strategy

## Design Philosophy

**V1 Goals**:
- **Function over form** - No depth restrictions (design for deep cases)
- **Modern assembly** - SMD components for clean, robust build
- **Hand-assembly friendly** - 0805 minimum size, accessible with basic tools
- **Iteration-ready** - Easy to debug, modify, and improve

---

## Board Overview

**Module**: Trace - 24HP Eurorack Oscilloscope
**Complexity**: Medium (mixed-signal: analog inputs + digital processing + display)
**Dimensions**: 121.92mm (W) × 128.5mm (H) max (Eurorack 3U)
**Depth**: No restriction for V1 (optimize in V2 if needed)

---

## PCB Stackup Decision

### 2-Layer Board (V1 Recommendation)

**Layers**:
- **Top**: SMD components, signal traces, ground pour
- **Bottom**: Ground plane (mostly solid), power routing, some signals, SMD components

**Pros**:
- Cheap ($5-20 for 5 boards from JLCPCB/PCBWay)
- Fast turnaround (3-5 days)
- Easy to debug/modify (can cut traces, add jumpers)
- Adequate for audio frequencies and RP2040 SPI speeds (MHz range)
- Both sides available for SMD components

**Cons**:
- Less ground plane coverage than 4-layer
- More careful routing required
- Limited power distribution options

**Verdict**: 2-layer for V1. Upgrade to 4-layer only if noise issues found in testing.

---

## Component Technology: SMD Primary

### SMD Selection (Hand-Assembly Friendly)

**Passives (Resistors, Capacitors, Diodes)**:
- **Size**: 0805 (2.0mm × 1.25mm) - **minimum size**
- Easy to hand-solder with fine-tip iron and flux
- Large enough to see and handle with tweezers
- Visible part markings on some components

**ICs**:
- **Op-amps (TL074)**: SOIC-14 (1.27mm pitch) - very hand-solderable
- **RP2040**: QFN-56 (0.4mm pitch) - requires hot air station
- **Buck regulator**: SOIC-8 or SOIC-8-EP (exposed pad)
- **Diodes (1N5819)**: SMA package (SMD equivalent of DO-41)

**Why SMD**:
- More robust (better vibration resistance, no lead breakage)
- Cleaner boards (professional appearance)
- Better electrical performance (shorter leads = less parasitics)
- Easier PCB routing (components on both sides)
- Industry standard for modern Eurorack modules

### Through-Hole Reserved for Mechanical Components

**Panel-mount only**:
- **Jacks** (Thonkiconn PJ398SM) - need mechanical strength
- **Encoders** (Bourns PEC11) - panel mount with through-hole pins
- **Buttons** - panel mount with through-hole connections
- **Power connector** (2×5 IDC header) - strain relief needs mechanical connection

**Optional through-hole**:
- Test points (header pins or bare vias)
- Debug headers (UART, SWD)

---

## Assembly Tooling (Purchase When Boards Arrive)

**Required when building**:
1. **Hot air station** ($50-80) - Youyue 858D or similar
   - Essential for QFN packages (RP2040)
   - Makes all SMD easier than iron-only
2. **Magnifying lamp** ($40) - Hands-free, built-in light
   - Essential for comfortable SMD work
3. **USB microscope** ($30-50) - Optional but highly recommended
   - 10-200x zoom on laptop screen
   - Game-changer for inspection and fine work
4. **Fine-tip soldering iron** - 0.5mm chisel or conical tip
5. **Flux paste** - Essential for SMD (makes everything easier)
6. **Tweezers** - Fine point, ESD-safe preferred
7. **Solder wick** - For fixing mistakes

**Design now, buy tools when boards ship** (3-5 day lead time gives you time to order tools).

---

## Board Dimensions and Mounting

### Physical Layout

**Eurorack panel**: 121.92mm × 128.5mm (24HP × 3U)

**PCB Architecture**: Separate panel + main board

**Front panel** (aluminum):
- 2mm thickness
- Mounts to Eurorack rails (M3 screws)
- Cutouts for display, encoders, buttons, jacks

**Main PCB**:
- Size: ~100mm × 120mm (leaves margin for standoffs)
- Mounts to front panel via standoffs (M3, 15-20mm length)
- 4-6 mounting holes around perimeter
- **Depth**: PCB sits 15-20mm behind panel (no depth restriction)

**Display mounting**:
- **Option A**: Mount directly to main PCB (perpendicular, saves space)
- **Option B**: Separate display board, ribbon cable to main PCB (cleaner routing)
- V1 can use either (no depth constraint)

---

## Component Placement Strategy

### Functional Zones

Divide PCB into logical sections to minimize crosstalk:

```
┌─────────────────────────────────────────┐
│  POWER (top-left)                       │
│  - Eurorack connector                   │
│  - Buck regulator (12V→3.3V)            │
│  - Filtering, decoupling                │
├─────────────────────────────────────────┤
│  ANALOG (left side)                     │
│  - Input jacks                          │
│  - Protection (D1,D2 - SMD SMA package) │
│  - Dividers (R1,R2 - 0805)              │
│  - Op-amps (TL074 - SOIC-14)            │
│  - Offset ref (R3,R4,C1 - 0805)         │
│  - Output jacks                         │
├─────────────────────────────────────────┤
│  DIGITAL (right side)                   │
│  - RP2040 (QFN-56)                      │
│  - Crystal, flash, USB                  │
│  - ADC input traces                     │
│  - Encoder/button inputs                │
│  - SPI display connector                │
├─────────────────────────────────────────┤
│  DISPLAY (separate or integrated)       │
│  - 3.5" LCD module                      │
│  - SPI connection to main board         │
└─────────────────────────────────────────┘
```

### Placement Rules

**1. Power Section** (top-left):
- Buck regulator close to Eurorack connector (<30mm)
- Input filtering caps immediately after connector (100nF + 10µF)
- Heat dissipation area for buck IC (copper pour)
- Keep switching noise away from analog section (>20mm separation)

**2. Analog Section** (left side):
- Input jacks at panel edge
- Protection diodes (SMD SMA) **immediately** after jacks (<5mm)
- High-Z nodes (divider outputs) kept short (<10mm traces)
- Op-amps (SOIC-14) central to analog area
- Output jacks at panel edge
- **Ground pour** around entire analog section

**3. Digital Section** (right side):
- RP2040 in center of digital area
- Crystal and load caps <5mm from RP2040 XTAL pins
- Flash IC close to RP2040 (<20mm, short SPI traces)
- USB connector at board edge (or omit if programming via SWD)
- Decoupling caps <5mm from each RP2040 power pin (0805, 100nF)

**4. Display**:
- Mount perpendicular to main board (no depth limit allows this)
- SPI connector near RP2040
- Can use right-angle header + ribbon cable

---

## Grounding Strategy (Critical for Mixed-Signal)

### Ground Plane Architecture (2-Layer)

**Top Layer**:
- Ground pour in unused areas (flood fill)
- Keep ground continuous (avoid splitting with traces where possible)
- Stitch to bottom ground via vias (grid pattern, 5-10mm spacing)

**Bottom Layer**:
- Solid ground plane as primary reference
- Only interrupt for power traces and critical signals
- Use ground plane for return current paths

### Analog/Digital Ground Approach

**Strategy: Unified ground with careful placement** (simpler, usually works)

- Single continuous ground plane
- Analog components on left, digital on right
- Ground current naturally divides
- No explicit split (avoids accidental loops)

**If noise issues arise** (can add in V2):
- Star ground with single connection point
- 0Ω resistor jumper between analog/digital grounds (easy to test)
- Ferrite bead connection (higher frequency isolation)

**Recommendation**: Start unified. Add star ground only if measurements show noise coupling.

---

## Power Distribution

### Power Rails

1. **+12V** (Eurorack) → Op-amps, buck input
2. **-12V** (Eurorack) → Op-amps
3. **GND** (Eurorack) → Common ground
4. **+3.3V** (regulated) → RP2040, display, offset reference

### Power Routing

**Buck Regulator Output (+3.3V)**:
- Wide traces (30mil / 0.75mm) from regulator to loads
- Star distribution from output capacitor
- Separate branches to:
  - RP2040 (with local 10µF + 100nF filtering)
  - Display (with local 10µF + 100nF filtering)
  - Offset reference (R3/R4 divider, with 100nF bypass)

**±12V Distribution**:
- Wide traces (20mil / 0.5mm) from Eurorack connector
- Direct routing to op-amp power pins
- Decoupling at each op-amp: 100nF (close) + 10µF (nearby)

**Via stitching on power traces**:
- Multiple vias connecting top and bottom copper
- Reduces trace resistance and improves current capacity

**Trace Width Reference**:
- +3.3V (500mA): 30mil (0.75mm) - conservative
- +12V (100mA): 20mil (0.5mm) - adequate
- -12V (100mA): 20mil (0.5mm) - adequate

---

## Signal Routing

### Analog Paths (Highest Priority)

**Input Jack → Protection → Divider → Op-Amp → ADC**

**Routing guidelines**:
- Keep traces **short and direct** (minimize length)
- Use 45° bends (not 90°)
- High-Z nodes (after R1/R2 divider): <10mm trace length
- Route away from digital signals (>5mm clearance minimum)
- Guard with ground pour on both sides (ground "moat")

**Critical trace widths**:
- Signal traces: 10-15mil (0.25-0.4mm) - adequate for low-current signals
- High-Z divider output: Keep short, width less critical than length

**Sensitive nodes**:
1. **Divider output** (high-Z): <10mm to op-amp input
2. **Op-amp output to ADC**: Shield with ground guard traces
3. **Offset reference**: Keep R3/R4/C1 tightly clustered (<15mm triangle)

**Passthrough path**:
- Input → Op-amp follower → Output jack
- Short, direct route
- No crossing of ADC signal path

### Digital Paths

**SPI to Display** (RP2040 → LCD):
- MOSI, SCK, CS, DC signals
- Keep traces similar length (within 10mm of each other)
- Can route as pseudo-differential pairs (MOSI+SCK adjacent)
- Trace impedance not critical at low SPI speeds (<10MHz)

**ADC Inputs** (Op-Amp → RP2040 GPIO26/27):
- Short, direct traces (<50mm)
- Guard with ground traces/pour on both sides
- Add RC filter at RP2040 pin: 1kΩ series + 100pF to GND (noise immunity)

**Encoders/Buttons**:
- Not speed-critical (human interaction)
- Can use longer, meandering traces
- Pull-up resistors (10kΩ) near RP2040 GPIO pins

**Crystal** (12MHz for RP2040):
- **Critical**: Keep traces <5mm from RP2040 to crystal
- Guard with ground pour (ground "fence" around crystal traces)
- Load capacitors (22pF) right at crystal pins

---

## Decoupling and Bypassing

### Power Supply Decoupling

**Buck Regulator**:
- Input: 10µF + 100nF ceramic (close to VIN pin)
- Output: 22µF + 100nF ceramic (close to VOUT pin)
- Low-ESR ceramics (X5R or X7R dielectric)

**RP2040** (critical - has multiple power domains):
- **Every power pin**: 100nF ceramic (0805) within 5mm
- VREG input: 1µF ceramic
- VREG output: 1µF ceramic
- USB power: 1µF + 100nF ceramic
- **Total**: ~12× 100nF caps for RP2040 alone

**TL074 Op-Amp**:
- **Each IC**: 100nF (close, <5mm) + 10µF electrolytic (nearby, <20mm)
- Place 100nF on both +12V and -12V pins

**Display Module**:
- 10µF + 100nF at power input connector

### Signal Bypassing

**Offset reference** (R3/R4 divider):
- 100nF ceramic at the 1.73V node (C1)
- Stabilizes reference, filters supply noise

**ADC inputs** (optional, add if needed):
- 100pF capacitor to GND at RP2040 ADC pin
- Forms RC filter with 1kΩ series resistor (R7)
- Low-pass filter: ~1.6MHz cutoff (well above audio, blocks digital noise)

---

## Thermal Management

### Buck Regulator (Primary Heat Source)

**Heat dissipation** (worst case):
- Input: 12V × 160mA = 1.92W
- Output: 3.3V × 500mA = 1.65W
- Loss: ~0.3W @ 85% efficiency

**Cooling strategy**:
- Use IC with **exposed thermal pad** (SOIC-8-EP package)
- Connect thermal pad to ground plane via **thermal vias** (4-6 vias, 0.3mm drill)
- Copper pour around IC (acts as heatsink, ~500mm² area)
- **No external heatsink needed** for <0.5W dissipation

**Via stitching for thermal pad**:
- Multiple vias inside thermal pad footprint
- Connects to bottom ground plane (large copper area)
- Via-in-pad is acceptable for hand assembly (may need solder paste filling)

### Op-Amps

**Heat**: Negligible (<50mW per channel)
**Cooling**: None needed, standard ground plane sufficient

### RP2040

**Heat**: Low (<200mW typical)
**Cooling**: Thermal pad connected to ground via vias (like buck regulator)

---

## SMD Assembly Considerations

### Footprint Selection

**0805 Passives**:
- Pad size: 1.4mm × 1.6mm (standard)
- Gap: 0.6mm between pads
- Easy hand-soldering with fine-tip iron

**SOIC-14 (TL074)**:
- Pin pitch: 1.27mm
- Hand-solderable with flux and drag-soldering technique
- Exposed pins (easy to inspect/rework)

**QFN-56 (RP2040)**:
- Pitch: 0.4mm (requires hot air or reflow)
- Thermal pad: Connect to ground with vias
- Solder paste stencil recommended (or careful hand-paste application)

**SMA Diodes (DO-214AC package)**:
- **Dedicated land pattern** (not same as 0805)
- Pad dimensions: 2.0-2.3mm length × 1.5mm width (each pad)
- Overall pad-to-pad span: 4.6-5.3mm
- Body size: ~2.5mm × 1.3mm × 2.3mm height
- Polarized - mark cathode clearly on silkscreen (band end)

### Hand-Assembly Tips (Documented on Silkscreen)

**Polarity indicators**:
- Diodes: Cathode mark (line) on silkscreen
- Electrolytic caps: "+" symbol clearly marked
- ICs: Pin 1 indicator (dot or line)

**Component values on silkscreen**:
- Print resistor/cap values near component (100k, 100n, etc.)
- Makes assembly easier (no constant BOM lookup)

**Assembly order silkscreen hints**:
- "SMD FIRST" note on board
- "INSTALL BEFORE PANEL" for jacks/encoders

---

## Design for Testing and Debug

### Test Points

Add accessible test points (via or pad):
- **Power**: +12V, -12V, +3.3V, GND (4 points)
- **Analog**: Divider output, op-amp output, ADC input (6 points, 2 channels)
- **Digital**: SPI signals (MOSI, SCK, CS), I2C if used
- **Critical**: Offset reference (1.73V) - verify accuracy

**Location**: Along board edge or in designated test area
**Marking**: Clear silkscreen labels (TP1: +12V, TP2: CH1_DIV, etc.)

### Debug Headers

**SWD Programming/Debug** (RP2040):
- SWDIO, SWCLK, GND, +3.3V (4 pins)
- 2.54mm (0.1") header or Tag-Connect footprint
- Essential for firmware development

**UART Debug** (optional):
- TX, RX, GND (3 pins)
- 2.54mm header
- Useful for serial debugging

**Placement**: Along board edge, clearly labeled

### Status LEDs (SMD 0805)

**Power indicator**:
- LED on +3.3V rail (green)
- Series resistor (1kΩ, 0805) for ~2mA current

**Status/heartbeat** (optional):
- LED connected to RP2040 GPIO
- User-programmable (blink patterns for debug)

---

## PCB Fabrication Specs

### Target Manufacturer: JLCPCB/PCBWay (Cheap, Fast, Reliable)

**Specifications**:
- **Layers**: 2
- **Dimensions**: ~100mm × 120mm
- **Thickness**: 1.6mm (standard)
- **Copper weight**: 1oz (35µm) - standard
- **Surface finish**: ENIG (gold) - recommended for fine-pitch SMD
  - Alternative: HASL (lead-free) - cheaper, adequate for 0805+
- **Silkscreen**: White on green soldermask (classic look)
- **Soldermask color**: Green (standard, cheapest)
- **Min trace/space**: 6mil/6mil (0.15mm) - use 8mil/8mil for hand-routing headroom
- **Min drill**: 0.3mm (avoid smaller for reliability)

**Cost estimate**: $10-25 for 5 boards (1-week delivery)

### Design Rules (JLCPCB Standard)

- Trace width: 8mil (0.2mm) minimum, use 10mil (0.25mm) for signals, 20-30mil for power
- Trace spacing: 8mil (0.2mm) minimum
- Via size: 0.3mm drill, 0.6mm pad (standard)
- Annular ring: 0.15mm minimum (DRC will check)
- Silkscreen width: 6mil (0.15mm) minimum
- SMD pad to silkscreen: 4mil (0.1mm) clearance

---

## Bill of Materials Organization

### BOM Structure for Hand Assembly

Organize by section and assembly order:

**1. SMD Components (solder first)**:
- Power: Buck regulator, input/output caps
- Analog: Op-amps, resistors (0805), diodes (SMA), caps (0805)
- Digital: RP2040, crystal, flash, decoupling caps (0805)
- LEDs and current-limiting resistors

**2. Through-Hole (solder after SMD)**:
- Power connector (2×5 IDC)
- Jacks (Thonkiconn)
- Encoders (after panel mockup)
- Buttons (after panel mockup)

**BOM Columns**:
- Ref Des (R1, C5, U3)
- Value (100kΩ, 100nF)
- Package (0805, SOIC-14, SMA)
- Part Number (manufacturer specific)
- Supplier (Mouser/Digikey/LCSC)
- Quantity
- Notes (polarity, orientation, assembly tips)

---

## Pre-Fabrication Checklist

### Before Ordering PCBs

- [ ] All component footprints verified against datasheets (especially QFN RP2040)
- [ ] Decoupling caps within 5mm of IC power pins (measure in layout tool)
- [ ] Ground plane continuous (no accidental splits) - visual check + DRC
- [ ] Power trace widths adequate (30mil for 500mA, 20mil for 100mA)
- [ ] High-Z analog traces <10mm (divider output to op-amp)
- [ ] Analog and digital sections physically separated (>20mm)
- [ ] Protection diodes <5mm from input jacks
- [ ] All mounting holes correct size and placement (M3, verify against panel)
- [ ] Thermal vias in buck regulator and RP2040 thermal pads
- [ ] Silkscreen labels clear, not covered by components
- [ ] Component reference designators visible (not under parts)
- [ ] Polarity marks for diodes, LEDs, electrolytics
- [ ] Design Rule Check (DRC) passed - zero errors
- [ ] Electrical Rule Check (ERC) passed - zero errors
- [ ] 3D model visualization checked (component collisions, clearances)
- [ ] Gerber files reviewed in external viewer (gerbv or online viewer)
- [ ] Board outline correct (matches panel dimensions)

---

## Iteration Strategy (V1 → V2)

### V1 Prototype Goals

**Must work perfectly**:
- Power supply (buck regulator, ±12V distribution)
- One input channel (protection, divider, op-amp, ADC)
- RP2040 boots and programs via SWD
- ADC reads input voltages correctly

**Can be bodge-wired if needed**:
- Second input channel (clone of first, easy to add)
- Display connection (jumper wires acceptable for V1)
- Encoders/buttons (can breadboard for firmware dev)

**Design for fixes**:
- 0Ω resistor jumpers for ground experiments (analog/digital split testing)
- Extra via holes for jumper wires
- Test points on all critical signals
- Wide component spacing (easy rework access)

### V2 Improvements (After V1 Testing)

Based on V1 learnings:
- Fix any noise/crosstalk issues (add ground guards, split grounds if needed)
- Optimize layout for manufacturability (tighter spacing if SMD assembly went well)
- Add missing features discovered during firmware development
- Reduce board size if possible (move toward shallow design)
- Consider 4-layer stackup if noise was problematic

---

## Next Steps

1. **Complete schematic in KiCad** (all sections: power, analog, digital, display)
2. **Assign footprints** to all components (verify 0805, SOIC, QFN packages)
3. **Generate netlist** for PCB layout
4. **PCB layout** following this strategy
5. **DRC/ERC** checks (iterate until clean)
6. **3D visualization** (check mechanical fit)
7. **Gerber generation** and review
8. **Order prototype boards** (5-10 units)
9. **Order assembly tools** (hot air station, magnifier) while boards ship
10. **Assembly and testing**

---

## Tools and Resources

### PCB Design Software

**Recommended**: **KiCad 7+** (free, open-source, excellent for Eurorack)
- Built-in RP2040 symbols/footprints
- 3D visualization
- Strong community libraries

**Component Libraries Needed**:
- Eurorack-specific (Thonkiconn, power headers) - github.com/TheSlowGrowth/KiCad-Eurorack-libs
- RP2040 (built-in to KiCad 7)
- Display connector (verify against actual display part number)

### Learning Resources

**KiCad tutorials**:
- Official KiCad documentation (getting started guide)
- DigiKey KiCad video series (YouTube)
- Contextual Electronics "Shine On You Crazy KiCad" series

**PCB layout best practices**:
- Phil's Lab (YouTube) - mixed-signal PCB design
- Rick Hartley talks - grounding and layer stackup
- Altium Academy - signal integrity basics (concepts apply to KiCad)

**Eurorack references**:
- Mutable Instruments open-source designs (github.com/pichenettes) - gold standard
- Look at Plaits, Stages schematics/layouts for inspiration

### Manufacturers

- **JLCPCB** - Cheap, fast, good quality (recommended for prototype)
- **PCBWay** - Similar to JLCPCB, sometimes better for small batches
- **OSH Park** - US-based, premium quality, great for 2-layer (purple boards!)

---

## Design Confidence Level

**This strategy provides**:
✓ Clear component placement rules
✓ Grounding approach for mixed-signal design
✓ Thermal management for power components
✓ Hand-assembly friendly SMD choices (0805 minimum)
✓ Test and debug infrastructure
✓ Iteration path (V1 → V2 improvements)

**Ready to proceed with**: Schematic completion → PCB layout → Prototype order

**Depth restriction**: None (design for function, optimize later)
**Assembly method**: SMD primary (cleaner, more robust)
**Minimum feature size**: 0805 passives (hand-solderable with basic tools + magnification)
