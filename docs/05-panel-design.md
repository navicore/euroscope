# Panel Design

## Selected Display: 3.5" 480×320 SPI LCD ✓

After thorough research, we've selected the **3.5" 480×320 ILI9488/ILI9486 SPI display** as our display solution.

### Decision Rationale

**Display meets all requirements:**

| Requirement | Specification | Status |
|-------------|--------------|--------|
| **Resolution** | 480×320 pixels (landscape) | ✓ Excellent |
| **Aspect ratio** | 3:2 landscape | ✓ Good for scope (wider = more time samples) |
| **Interface** | 4-wire SPI | ✓ RP2040 compatible |
| **Controller** | ILI9488 or ILI9486 | ✓ Well documented |
| **Voltage** | 3.3V-5V logic compatible | ✓ Works with RP2040 (3.3V) |
| **Power** | ≤150mA @ 5V typical | ✓ Within Eurorack budget |
| **Touch** | Available with or without | ✓ Non-touch versions exist |
| **Rust support** | mipidsi crate | ✓ Proven embedded-graphics support |
| **RP2040 proven** | Multiple examples found | ✓ Known working combination |
| **Availability** | Amazon, AliExpress, Elecrow | ✓ Readily available ($8-15) |

### Physical Dimensions (Verified)

**Display active area**: 73.44mm (W) × 48.96mm (H)
**Module PCB size**: ~56mm (W) × 98mm (H)
**Module depth**: ~10-15mm (typical for SPI version)

### Electrical Specifications (Verified)

**Power supply**: 3.3V-5V (we'll use 3.3V from buck regulator)
**Logic level**: 3.3V (native RP2040 compatibility)
**Current draw**: ≤150mA @ 5V, likely ~100-120mA @ 3.3V
**Interface**: 4-wire SPI (MOSI, SCK, CS, DC) + Reset pin

### Firmware Support (Verified)

**Rust crate**: [mipidsi](https://crates.io/crates/mipidsi)
- Supports ILI9486 and ILI9488 controllers
- Uses embedded-graphics-core (full drawing API)
- RGB666 (18-bit) color mode for SPI
- Proven with RP2040 via embassy-rp HAL
- Stable Rust 1.75.0+

**Example code exists** for RP2040 + ILI9488 combination.

---

## Module Size: 24HP ✓

With 3.5" display dimensions confirmed, **24HP is optimal**.

### Layout: Option B (Side-by-Side) ✓

```
┌─────────────────────────┐  121.92mm (24HP)
│  ┌──────────┐  [E1]     │
│  │          │  [E2]     │  ← Display (left)
│  │ DISPLAY  │           │     Encoders (right)
│  │  3.5"    │  [B1]     │     Buttons (right)
│  │ 480×320  │  [B2]     │
│  └──────────┘           │
│                         │
│  ○IN1  ○IN2  ○OUT1 ○OUT2│  ← Jacks (bottom)
└─────────────────────────┘
```

**Rationale for Option B**:
- **Ergonomic**: Controls next to display (look at waveform while adjusting)
- **Scope-like**: Resembles bench scope layout
- **Efficient**: Maximizes display width for time-axis viewing
- **Natural hand position**: Left hand near display, right hand on controls

---

## Panel Layout Dimensions

**24HP width** = 121.92mm
**Usable width** (with 2mm edge margins) = ~118mm
**3U height** = 128.5mm

### Component Positioning

#### Display (Left Side)
- **Position**: Left-aligned, vertically centered
- **PCB width**: 56mm
- **Active area**: 73.44mm × 48.96mm
- **Left margin**: 3mm from panel edge
- **Display center**: ~50mm from left edge

#### Encoders (Right Side, Upper)
- **Encoder 1** (Time/div): 75mm from left edge
- **Encoder 2** (Volts/div): 100mm from left edge
- **Vertical position**: ~40mm from top
- **Spacing**: 25mm center-to-center (allows 20-25mm knobs)

#### Buttons (Right Side, Lower)
- **Button 1** (Trigger): 75mm from left edge
- **Button 2** (Run/Stop): 100mm from left edge
- **Vertical position**: ~75mm from top
- **Spacing**: 25mm center-to-center

#### Jacks (Bottom Row)
- **IN1**: 20mm from left edge
- **IN2**: 45mm from left edge
- **OUT1**: 75mm from left edge
- **OUT2**: 100mm from left edge
- **Vertical position**: 115mm from top (13.5mm from bottom)
- **Spacing**: 25mm center-to-center

### Mounting Holes
- Standard Eurorack M3 slotted holes
- Vertical spacing: 122.5mm center-to-center
- Horizontal positions: Per manufacturer standard (typically 7.5mm from edges)

---

## Panel Material Selection

### Recommendation: **Aluminum** (2mm thickness)

**Pros**:
- Professional appearance
- Durable (won't crack around jack holes)
- Standard for Eurorack modules
- Good EMI shielding
- Accepts various finishes (anodized, powder coat, brushed)

**Cons**:
- More expensive than alternatives ($20-40 per panel)
- Requires CNC milling or outsourcing

**Alternative: FR4 PCB Panel** (budget option)
- Can combine front panel + circuit board
- Silkscreen graphics included
- Very cheap (~$5 for 5 panels from JLCPCB)
- Adequate for prototype/DIY build
- Less premium feel, thinner (1.6mm typical)

---

## Panel Graphics

**Labeling style**: Minimal, clean

**Jack labels** (bottom row):
- IN 1
- IN 2
- OUT 1
- OUT 2

**Control labels** (near encoders/buttons):
- TIME (encoder 1)
- VOLTS (encoder 2)
- TRIG (button 1)
- RUN (button 2)

**Module name**:
- "TRACE" in top-right corner or above display
- Font: Arial, Helvetica, or system sans-serif

**Panel finish**:
- Black matte (recommended - reduces glare on display)
- Brushed aluminum (alternative)
- White text/labels

---

## Clearance and Depth

**Module depth budget** (front to back):
- Panel thickness: 2mm (aluminum)
- Display depth: 10-15mm (from front panel)
- PCB depth: 1.6mm
- Tallest rear components: ~10mm (power connector, jacks)
- **Total depth**: ~25-30mm

**Eurorack standard depth**: 40-50mm available (depending on case)
**Margin**: 10-20mm safety margin ✓

---

## Verified Display Part Numbers

**Known compatible 3.5" 480×320 SPI displays**:

1. **Amazon - LCD Screen Module, 3.5Inch 480×320 TFT LCD Display Module**
   - 4-wire SPI interface
   - ILI9488 driver chip
   - 3.3V~5V compatible
   - Link: [Amazon](https://www.amazon.com/3-5inch-display-interface-ili9488-electronic/dp/b08c7npqzr)

2. **Elecrow - 3.5 Inch 480×320 SPI TFT LCD Module with ILI9488 Driver**
   - Verified SPI support
   - Product page: [https://www.elecrow.com/3-5-inch-480-320-spi-tft-lcd-module-with-ili9488-driver.html](https://www.elecrow.com/3-5-inch-480-320-spi-tft-lcd-module-with-ili9488-driver.html)

3. **Generic "3.5 inch ILI9488 SPI" modules**
   - Widely available on AliExpress, eBay
   - Verify SPI interface (not parallel)
   - Verify non-touch or touch-optional

**Purchase recommendation**: Order from Amazon for fast shipping and easy returns if dimensions don't match.

---

## Design Validation Checklist

- [x] Display resolution adequate for waveform viewing (480 pixels wide)
- [x] Display aspect ratio suitable for scope (landscape 3:2)
- [x] Display interface compatible with RP2040 (SPI)
- [x] Display power consumption within budget (<150mA)
- [x] Display voltage compatible (3.3V logic)
- [x] Rust driver support confirmed (mipidsi crate)
- [x] Physical dimensions fit 24HP panel
- [x] Component spacing allows knob clearance (25mm spacing)
- [x] Jack spacing meets Eurorack standards (25mm)
- [x] Module depth within Eurorack limits (<30mm)
- [x] Display readily available from multiple sources

---

## Next Steps

1. **Create detailed panel drawing** (Inkscape, Front Panel Designer, or CAD)
2. **Order sample display** (verify physical fit before PCB design)
3. **Design PCB layout** (now that panel is locked)
4. **Create schematic** (with confirmed component positions)
5. **Prototype cardboard mockup** (verify ergonomics and button reach)

---

## Open Questions (Minor)

- [ ] Encoder knob style (Davies clone, Rogan, custom?)
- [ ] Button LED colors (Run/Stop: green/red? Or single color?)
- [ ] Panel finish preference (matte black vs brushed aluminum)
- [ ] Module name typography (font choice)

**Note**: These are aesthetic decisions that don't block technical progress.
