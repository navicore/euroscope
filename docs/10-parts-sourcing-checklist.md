# Parts Sourcing Checklist

## Purpose
Verify availability and pricing at professional distributors (Newark, Mouser, Digi-Key) for Trace module components.

---

## Critical Components - Verify at Newark

### Op-Amps
- [ ] **TL074IPWR** (SOIC-14, SMD)
  - Manufacturer: Texas Instruments
  - Package: SOIC-14 (TSSOP also acceptable)
  - Quantity needed: 2 (one per channel)
  - Specs: Quad JFET op-amp, ±18V supply
  - Newark part number: _____________
  - Price (qty 10): _____________

### Microcontroller Options

**Option A: Raspberry Pi Pico (Development)**
- [ ] **Raspberry Pi Pico** (RP2040 board)
  - Through-hole headers or castellated
  - Quantity needed: 1
  - Newark part number: _____________
  - Price: _____________

**Option B: RP2040 Module (Production)**
- [ ] **RP2040 Stamp** or **Pimoroni Tiny2040**
  - Castellated SMD module
  - Quantity needed: 1
  - Newark/Mouser part number: _____________
  - Price: _____________

**Option C: Bare IC (Advanced)**
- [ ] **RP2040** (QFN-56)
  - Manufacturer: Raspberry Pi
  - Requires: 12MHz crystal, W25Q16 flash, USB circuit
  - Defer this option for V1
  - Newark part number: _____________

### Voltage Regulator (Buck Converter)

- [ ] **AP63203WU-7** (TSOT-26)
  - Manufacturer: Diodes Inc
  - Output: 3.3V, 2A synchronous buck
  - Package: **TSOT-26 (TSOT-23-6 style)** — verify footprint, not SOIC-8
  - Newark part number: _____________
  - Price: _____________

- [ ] **LMR33630ADDAR** (SO PowerPAD-8) - Alternative
  - Manufacturer: Texas Instruments
  - Output: 3.3V, 3A synchronous buck
  - Package: **SO PowerPAD-8 (DDA)** — not standard SOIC-8; requires exposed thermal pad
  - Newark part number: _____________
  - Price: _____________

**Buck Regulator Support Components:**
- [ ] **Inductor**: 4.7µH to 10µH, >2A rating (e.g., Bourns SRP series)
  - Newark part number: _____________
- [ ] **Output cap**: 22µF ceramic, X7R, 10V+ (0805 or 1206)
  - Newark part number: _____________
- [ ] **Input cap**: 10µF ceramic, X7R, 16V+ (0805 or 1206)
  - Newark part number: _____________

### Diodes (Protection)

**Input Clamps (D1, D2 - per channel):**
- [ ] **SS14** (SMA/DO-214AC, SMD) - Preferred
  - 40V, 1A Schottky
  - Quantity needed: 4 (2 per channel)
  - Newark part number: _____________
  - Price (qty 10): _____________

- [ ] **1N5819** (DO-41, through-hole) - Alternative
  - 40V, 1A Schottky
  - Use if preferring through-hole assembly
  - Newark part number: _____________
  - Price (qty 10): _____________

**ADC Clamps (D3, D4 - per channel):**
- [ ] **BAT85** (SOD-123 or DO-35)
  - Low capacitance Schottky (<15pF)
  - 30V, 200mA (sufficient for ADC protection)
  - Quantity needed: 4 (2 per channel)
  - Newark part number: _____________
  - Price (qty 10): _____________

### Rotary Encoders

- [ ] **Bourns PEC11R-4215F-S0024** (or similar)
  - 24 detents per revolution
  - Integrated push switch
  - Panel mount, threaded bushing
  - 6mm D-shaft or knurled
  - Quantity needed: 2
  - Newark part number: _____________
  - Price: _____________

- [ ] **Alps EC11** - Alternative
  - Similar specs to Bourns
  - Newark/Mouser part number: _____________
  - Price: _____________

### Toggle Switches (Run/Stop Controls)

- [ ] **C&K 7101** series (SPDT, panel mount)
  - 2-position toggle
  - Panel mount, threaded bushing (6mm)
  - ON-ON configuration
  - Quantity needed: 2
  - Newark part number: _____________
  - Price: _____________

- [ ] **E-Switch 100SP1T1B1M2QEH** - Alternative
  - SPDT panel mount toggle
  - Newark part number: _____________
  - Price: _____________

### LEDs (Status Indicators)

- [ ] **Dual-color LED** (5mm, common cathode, red/green)
  - Forward voltage: ~2V (red), ~3V (green)
  - Quantity needed: 2 (one per channel)
  - Newark part number: _____________
  - Price: _____________

**Alternative: Separate LEDs**
- [ ] 5mm Red LED (qty 2)
- [ ] 5mm Green LED (qty 2)
- Mount in dual-LED holder or side-by-side

### Jacks (Eurorack 3.5mm)

**Note**: Thonkiconn (PJ398SM) is Eurorack standard but may not be at Newark.

- [ ] **Thonkiconn PJ398SM**
  - 3.5mm mono switched jack
  - Panel mount, hex nut
  - Quantity needed: 4
  - **Source**: Thonk.co.uk or Modular Addict
  - Price: _____________

- [ ] **Alternative at Newark**: Search "3.5mm mono jack panel mount"
  - Must be panel mount (not PCB mount only)
  - Switched or non-switched acceptable
  - Newark part number: _____________

### Power Connector (Eurorack)

- [ ] **Shrouded 2×5 IDC header** (keyed, vertical)
  - **MUST have keying tab** (prevents reverse polarity)
  - 10-pin (2×5), 2.54mm pitch
  - Vertical or right-angle mount
  - Manufacturer: Molex, Amphenol, TE Connectivity
  - Example: Molex 70553 series
  - Quantity needed: 1
  - Newark part number: _____________
  - Price: _____________

---

## Passives - Verify Availability

### Resistors (1% Metal Film, 0805 SMD)

Standard values needed (verify availability in bulk):
- [ ] **1kΩ** (qty: 10+) - Protection, LED current limiting
- [ ] **100kΩ** (qty: 20+) - Op-amp feedback, summing, dividers
- [ ] **620kΩ** (qty: 4) - Input voltage divider (R1)
- [ ] **91kΩ** (qty: 2) - Offset reference (R3)

**Manufacturer preference**: Yageo, Vishay, KOA Speer
- Newark part number (generic 0805 resistor kit): _____________

### Capacitors (0805 SMD preferred)

**Ceramic capacitors (X7R or X5R dielectric):**
- [ ] **100nF** (0.1µF, 50V) - Decoupling (qty: 20+)
  - Newark part number: _____________
- [ ] **10µF** (16V+, X7R) - Power supply filtering (qty: 5)
  - Newark part number: _____________
- [ ] **22µF** (10V+, X7R) - Buck regulator output (qty: 2)
  - Newark part number: _____________

**Manufacturer preference**: Murata, Samsung, TDK

---

## Display - Amazon/AliExpress Only

- [ ] **3.5" 480×320 SPI TFT Display (ILI9488)**
  - Not available at Newark (hobbyist part)
  - **Source**: Amazon or AliExpress
  - Link: https://www.amazon.com/3-5inch-display-interface-ili9488-electronic/dp/B08C7NPQZR
  - Price: ~$12
  - Order for physical fit verification

---

## Knobs - Eurorack Specialist Suppliers

- [ ] **Davies 1900H clone knobs** (burgundy/maroon)
  - 6mm shaft (fits PEC11 encoder)
  - 20-22mm diameter recommended
  - Quantity needed: 2
  - **Source**: Thonk.co.uk, Modular Addict, Love My Switches
  - Price: _____________

**Alternative colors** (if burgundy unavailable):
- [ ] Dark blue
- [ ] Forest green
- [ ] Black (fallback)

---

## Panel Fabrication

- [ ] **Aluminum panel** (2mm, brushed/matte finish)
  - 24HP width (121.92mm)
  - 3U height (128.5mm)
  - **Source**: Front Panel Express, Schaeffer AG, or local machine shop
  - Price estimate: _____________

- [ ] **FR4 PCB panel** (budget alternative)
  - Same dimensions
  - **Source**: JLCPCB, PCBWay (submit as "panel" with silkscreen)
  - Price estimate (qty 5): _____________

---

## PCB Fabrication

- [ ] **Main PCB** (2-layer, SMD)
  - Dimensions: TBD (depends on component layout)
  - Finish: HASL or ENIG
  - **Source**: JLCPCB, PCBWay, OSH Park
  - Price estimate (qty 5-10): _____________

---

## Verification Tasks

### Phase 1: Critical Components (Do First)
- [ ] TL074 op-amp availability and price
- [ ] RP2040 Pico or module availability
- [ ] Buck regulator (AP63203 or LMR33630) + support parts
- [ ] Schottky diodes (SS14 and BAT85)
- [ ] Toggle switches (C&K or E-Switch)
- [ ] Rotary encoders (Bourns PEC11)

### Phase 2: Support Components
- [ ] Resistor and capacitor kits (0805 SMD)
- [ ] LEDs (dual-color or separate red/green)
- [ ] Power connector (keyed 2×5 IDC)
- [ ] Inductor for buck regulator

### Phase 3: Eurorack-Specific Parts
- [ ] Thonkiconn jacks (or suitable alternative)
- [ ] Burgundy knobs (Eurorack suppliers)
- [ ] Panel fabrication options

### Phase 4: Physical Prototyping
- [ ] Order display from Amazon (fit test)
- [ ] Order sample toggle switches (feel test)
- [ ] Order sample encoders (feel test)

---

## Cost Tracking

| Category | Estimated Cost | Actual Cost (Newark) | Notes |
|----------|----------------|----------------------|-------|
| ICs (op-amps, MCU, regulator) | $7 | | |
| Diodes (8 total) | $3 | | |
| Encoders (2) | $6 | | |
| Toggles + LEDs (2 sets) | $6 | | |
| Knobs (2) | $4 | | |
| Jacks (4) | $4 | | |
| Passives (bulk) | $5 | | |
| Display | $12 | | Amazon only |
| PCB | $15 | | PCB fab house |
| Panel | $30 | | Panel fab or FR4 |
| **Total** | **~$92** | | |

**Target**: Stay under $100 for single prototype build (excluding shipping)

---

## Next Steps After Sourcing

1. **Create detailed BOM** with confirmed part numbers
2. **Order samples** of critical parts (encoders, toggles, display)
3. **Complete schematic** in KiCad with verified components
4. **Design PCB layout** with actual footprints
5. **Order prototype PCB + panel** when design locked

---

## Notes

- **Minimum order quantities (MOQ)**: Some parts may have MOQ at Newark. If problematic, use Mouser or Digikey (similar pricing, different stock).
- **Lead times**: Note any parts with >2 week lead time - may need alternatives
- **Substitutions**: If exact part unavailable, find equivalent with same footprint/specs
- **Stock alerts**: Set up stock notifications for critical low-stock parts

---

## Supplier Contact Info

- **Newark**: https://www.newark.com
- **Mouser**: https://www.mouser.com (backup for Newark out-of-stock)
- **Digi-Key**: https://www.digikey.com (backup)
- **Thonk**: https://www.thonk.co.uk (Eurorack specialist - jacks, knobs)
- **Modular Addict**: https://modularaddict.com (Eurorack alternative to Thonk)
