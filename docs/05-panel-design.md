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

### Layout: Side-by-Side with Per-Channel Controls ✓

```
┌─────────────────────────┐  121.92mm (24HP)
│  TRACE                  │
│                         │
│  ┌──────────┐           │
│  │          │   [E1] ●  │  ← Ch1: Time/div encoder, Run/Stop toggle + LED
│  │ DISPLAY  │   ⬍─⬍    │
│  │  3.5"    │           │
│  │ 480×320  │   [E2] ●  │  ← Ch2: Volts/div encoder, Run/Stop toggle + LED
│  │          │   ⬍─⬍    │
│  └──────────┘           │
│                         │
│  ○IN1  ○IN2  ○OUT1 ○OUT2│  ← Jacks (bottom)
└─────────────────────────┘
```

**Key Design Decision: Independent Per-Channel Freeze**
- Each channel has its own Run/Stop toggle switch
- Dual-color LED (red/green) indicates state per channel
- **Use case**: Freeze Ch1 as reference, tune Ch2 to match waveform shape
- **Tactile quality**: Toggle switches preferred over buttons (more precise, better feel)

**Rationale for Layout**:
- **Ergonomic**: Controls next to display (look at waveform while adjusting)
- **Scope-like**: Resembles bench scope layout
- **Efficient**: Maximizes display width for time-axis viewing
- **Natural hand position**: Left hand near display, right hand on controls
- **Per-channel control**: Each channel has dedicated freeze control for independent operation

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

#### Channel 1 Controls (Right Side, Upper)
- **Encoder 1** (Time/div): 85mm from left edge
- **Toggle 1** (Ch1 Run/Stop): 85mm from left edge, ~55mm from top
- **LED 1** (dual-color red/green): Adjacent to toggle (3mm offset)
- **Vertical spacing**: ~15mm between encoder and toggle

#### Channel 2 Controls (Right Side, Lower)
- **Encoder 2** (Volts/div): 85mm from left edge, ~75mm from top
- **Toggle 2** (Ch2 Run/Stop): 85mm from left edge, ~90mm from top
- **LED 2** (dual-color red/green): Adjacent to toggle (3mm offset)
- **Vertical spacing**: ~15mm between encoder and toggle

**Control group spacing**: ~20mm vertical gap between Ch1 and Ch2 control groups

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

**Control labels** (near encoders/toggles):
- CH1 / TIME (encoder 1 + toggle 1)
- CH2 / VOLTS (encoder 2 + toggle 2)
- LED states: Green = Running, Red = Stopped (frozen)

**Module name**:
- "TRACE" in top-right corner or above display
- Font: Arial, Helvetica, or system sans-serif

**Panel finish**:
- **Silver/brushed aluminum** (recommended - matches existing rack aesthetic)
- Burgundy knobs for encoders (distinctive accent color)
- White text/labels
- Matte or bead-blasted surface texture (reduces fingerprints)

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
   - Link: [Amazon](https://www.amazon.com/3-5inch-display-interface-ili9488-electronic/dp/B08C7NPQZR)

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

## Aesthetic Decisions ✓

- [x] **Panel finish**: Silver/brushed aluminum (matches existing rack: Random*Source, Intellijel, Joranalogue)
- [x] **Knob color**: Burgundy (distinctive accent, not copying other manufacturers)
- [x] **Control type**: Toggle switches (not buttons - better tactile quality)
- [x] **LED indicators**: Dual-color (red/green) per channel for Run/Stop state
- [ ] Module name typography (font choice - deferred)
- [ ] Encoder knob style (Davies clone, Rogan, custom? - deferred)

**Design Philosophy**: Utilitarian silver aesthetic with burgundy accent, quality tactile controls (no cheap buttons)
