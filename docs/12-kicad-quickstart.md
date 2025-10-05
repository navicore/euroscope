# KiCad Quickstart Guide for Trace Module

## Project Setup

### 1. Create Project Directory and Initialize

```bash
cd /Users/navicore/git/navicore/euroscope
mkdir kicad
cd kicad
```

Open KiCad → **File → New Project** → Name: `trace-eurorack-module`

This creates:
- `trace-eurorack-module.kicad_pro` (project file)
- `trace-eurorack-module.kicad_sch` (root schematic)
- `trace-eurorack-module.kicad_pcb` (PCB layout, for later)

---

### 2. Set Up Hierarchical Sheets

Open the schematic editor (`.kicad_sch` file).

**Root sheet (main schematic):**
- This will only contain hierarchical sheet symbols
- Each sheet symbol links to a sub-schematic file

**Create 6 hierarchical sheets:**

1. **Place → Add Hierarchical Sheet** (or press `S`)
2. Draw a rectangle for the sheet
3. Fill in dialog:
   - **Sheet name**: Power Supply
   - **File name**: `power-supply.kicad_sch`
4. Repeat for each sheet:
   - `power-supply.kicad_sch` → Power Supply
   - `input-ch1.kicad_sch` → Input Channel 1
   - `input-ch2.kicad_sch` → Input Channel 2
   - `mcu-display.kicad_sch` → Microcontroller & Display
   - `user-controls.kicad_sch` → User Controls

**Root sheet layout example:**
```
┌─────────────────┐  ┌─────────────────┐
│  Power Supply   │  │  Input CH1      │
└─────────────────┘  └─────────────────┘

┌─────────────────┐  ┌─────────────────┐
│  Input CH2      │  │  MCU & Display  │
└─────────────────┘  └─────────────────┘

┌─────────────────┐
│  User Controls  │
└─────────────────┘
```

**Add hierarchical pins** to sheets (for power/signals):
- Right-click sheet → **Edit Sheet** → **Hierarchical Pins**
- Add: `+12V`, `-12V`, `+3.3V`, `GND` (output from power sheet)
- Add: `ADC_CH1`, `ADC_CH2` (output from input channels)
- These become connection points between sheets

---

## Sheet 1: Power Supply

### Symbols Needed

Open **Add Symbol** (press `A`) and search for:

#### Connectors
- **`Conn_02x05_Odd_Even`** → Eurorack 2×5 power connector (J_PWR)
  - Library: `Connector_Generic`
  - Alternative: `Conn_02x05_Shrouded` (if available)

#### Diodes
- **`D_Schottky`** → Reverse polarity protection (D_POL)
  - Library: `Device`
  - Footprint: `Diode_SMD:D_SMA` (for SS14)

#### Voltage Regulator
- **`AP63203`** or generic **`Regulator_Switching`**
  - Library: `Regulator_Switching` (may need to create custom symbol)
  - If not available: Use generic symbol, annotate package in properties
  - Footprint: `Package_TO_SOT_SMD:TSOT-23-6` (for AP63203WU-7)

#### Passives
- **`L`** → Inductor (L1, 4.7-10µH)
  - Library: `Device`
  - Footprint: `Inductor_SMD:L_1210_3225Metric` (or similar power inductor)

- **`C`** → Capacitors (input, output, decoupling)
  - Library: `Device`
  - Footprint: `Capacitor_SMD:C_0805_2012Metric`

- **`R`** → Resistors (if needed for regulator feedback)
  - Library: `Device`
  - Footprint: `Resistor_SMD:R_0805_2012Metric`

#### Power Symbols
- **`+12V`**, **`-12V`**, **`+3.3V`**, **`GND`**
  - Library: `power`
  - These are net labels, not physical symbols

### Circuit to Draw

Reference: `docs/11-schematic-design-plan.md` (Block 1, Power Supply)

```
J_PWR (2×5 connector)
  Pin 1 (-12V) ──┬─── D_POL (Schottky) ──> -12V net (to TL074)
                 │
  Pin 8,9 (+12V) ┴─── U_REG (buck regulator)
                        ├── L1 (inductor)
                        ├── C_IN (10µF)
                        ├── C_OUT (22µF)
                        └──> +3.3V net (to RP2040, display, offset ref)

  Pin 2,5,6,7 (GND) ──> GND net
```

**Steps:**
1. Place J_PWR connector symbol
2. Assign pin numbers (match Eurorack standard from design doc)
3. Place D_POL between pin 1 and -12V net
4. Place U_REG, L1, capacitors per buck regulator circuit
5. Add power symbols (+12V, -12V, +3.3V, GND)
6. Wire everything together (press `W` for wire tool)
7. Add decoupling caps near power nets (100nF ceramics)

### Component Values & Annotations

- **J_PWR**: Reference designator `J1`, value `Conn_02x05`
- **D_POL**: `D1`, value `SS14` or `1N5819`
- **U_REG**: `U1`, value `AP63203WU-7`
- **L1**: `L1`, value `10µH, 2A`
- **C_IN**: `C1`, value `10µF, 16V`
- **C_OUT**: `C2`, value `22µF, 10V`

**Set component properties:**
- Right-click symbol → **Properties**
- Set: Value, Footprint, Datasheet (optional)

---

## Sheet 2: Input Channel 1

### Symbols Needed

#### Connectors
- **`AudioJack2_SwitchT`** → Thonkiconn jack with switching (J_IN1, J_OUT1)
  - Library: `Connector_Audio`
  - Footprint: (custom, or use generic `PinHeader_1x03_P2.54mm`)

#### Op-Amps
- **`TL074`** → Quad op-amp (U1)
  - Library: `Amplifier_Operational`
  - Footprint: `Package_SO:SOIC-14_3.9x8.7mm_P1.27mm`
  - Use sections A and B for Channel 1

#### Diodes
- **`D_Schottky`** → Protection diodes
  - D1, D2 (input clamps): Footprint `Diode_SMD:D_SMA` (SS14)
  - D3, D4 (ADC clamps): Footprint `Diode_SMD:D_SOD-123` (BAT85)

#### Resistors & Capacitors
- **`R`** → All resistors (1kΩ, 100kΩ, 620kΩ, 91kΩ)
  - Footprint: `Resistor_SMD:R_0805_2012Metric`
- **`C`** → 100nF bypass cap
  - Footprint: `Capacitor_SMD:C_0805_2012Metric`

### Circuit to Draw

Reference: `docs/11-schematic-design-plan.md` (Block 2, Input Channel)

**Stage 1: Input Protection**
```
J_IN1 (jack) ──┬── R_PROT (1kΩ) ──┬── [to divider]
               │                    │
               │                 D1 ↑ (to +12V)
               │                 D2 ↓ (to -12V)
               │
               └── U1B (+) (passthrough buffer)
                    │
                   OUT ──> J_OUT1
```

**Stage 2: Voltage Divider**
```
[After R_PROT] ── R1 (620kΩ) ──┬── R2 (100kΩ) ── GND
                                │
                             [to summing amp]
```

**Stage 3: Offset & Summing Amplifier**
```
Offset Reference:
+3.3V ── R3 (91kΩ) ──┬── R4 (100kΩ) ── GND
                     │
                    C1 (100nF)
                     │
                  [+1.73V ref]

Summing Amp:
         R5 (100kΩ)
Signal ──────────┬──────(+) U1A
                 │         │
         R6 (100kΩ)       (─) ──┐
Offset ──────────┘         │    │
                          OUT ──┴── [to ADC protection]
```

**Stage 4: ADC Protection**
```
[U1A out] ── R7 (1kΩ) ──┬──> ADC_CH1 (to MCU sheet)
                         │
                      D3 ↑ (to +3.3V)
                      D4 ↓ (to GND)
```

### Component Designators & Values

**Channel 1 specific:**
- **J_IN1**: `J2`, value `Thonkiconn`
- **J_OUT1**: `J3`, value `Thonkiconn`
- **R_PROT**: `R1`, value `1kΩ`
- **R1**: `R2`, value `620kΩ, 1%`
- **R2**: `R3`, value `100kΩ, 1%`
- **R3**: `R4`, value `91kΩ, 1%` (shared with Ch2, place on this sheet)
- **R4**: `R5`, value `100kΩ, 1%` (shared)
- **R5**: `R6`, value `100kΩ, 1%`
- **R6**: `R7`, value `100kΩ, 1%`
- **R7**: `R8`, value `1kΩ`
- **C1**: `C3`, value `100nF` (offset bypass)
- **D1**: `D2`, value `SS14`
- **D2**: `D3`, value `SS14`
- **D3**: `D4`, value `BAT85`
- **D4**: `D5`, value `BAT85`
- **U1A**: Part of `U2` (TL074, section A)
- **U1B**: Part of `U2` (TL074, section B)

**Op-amp power pins (add on sheet):**
- Pin 4 (V-): Connect to -12V + 100nF decoupling
- Pin 11 (V+): Connect to +12V + 100nF decoupling

---

## Sheet 3: Input Channel 2

**This is a duplicate of Channel 1** with different designators:

- Use **U1C** and **U1D** (sections C and D of same TL074)
- Or use second TL074 (U3) with sections A and B
- Update all resistor/capacitor/diode designators (R9-R15, D6-D9, etc.)
- **Share offset reference** (R3/R4/C1 from Ch1) via net labels

**Add normalling circuit:**
- Use switched contact on J_IN2 (Thonkiconn)
- Connect switched contact to Ch1 signal (after R_PROT)
- When J_IN2 unplugged, Ch2 gets Ch1 signal

---

## Sheet 4: Microcontroller & Display

### Symbols Needed

#### Microcontroller
- **`Pico`** or custom **`RP2040_Pico`** symbol
  - May need to create custom symbol for Raspberry Pi Pico
  - Or use generic **`MCU_Module:Raspberry_Pi_Pico`** (if in library)
  - Footprint: Through-hole or castellated module

#### Display Connector
- **`Conn_01x08`** → 8-pin SPI display header
  - Library: `Connector_Generic`
  - Pins: VCC, GND, SCK, MOSI, CS, DC, RST, (BL optional)

### GPIO Connections to Draw

Reference: `docs/11-schematic-design-plan.md` (Block 3, Microcontroller)

```
RP2040 Pico:
  GPIO26 (ADC0) ←─ ADC_CH1 (from Ch1 sheet via hierarchical pin)
  GPIO27 (ADC1) ←─ ADC_CH2 (from Ch2 sheet)

  GPIO10 (SPI1_SCK) ──> Display SCK
  GPIO11 (SPI1_TX)  ──> Display MOSI
  GPIO13 ──> Display CS
  GPIO14 ──> Display DC
  GPIO15 ──> Display RST

  GPIO16-18 ──> Encoder 1 (A, B, SW) (from controls sheet)
  GPIO19-21 ──> Encoder 2 (A, B, SW)
  GPIO22-23 ──> Toggles (from controls sheet)
  GPIO6-9 ──> LEDs (red/green for each channel)

  3.3V ← +3.3V net (from power sheet)
  GND ← GND
```

**Add hierarchical pins:**
- Input: `+3.3V`, `GND`, `ADC_CH1`, `ADC_CH2`
- Output: `ENC1_A`, `ENC1_B`, `ENC1_SW`, `ENC2_A`, `ENC2_B`, `ENC2_SW`
- Output: `TOG1`, `TOG2`, `LED1_R`, `LED1_G`, `LED2_R`, `LED2_G`

**Add decoupling caps:**
- 100nF ceramic on +3.3V near Pico (multiple, one per power pin if bare RP2040)

---

## Sheet 5: User Controls

### Symbols Needed

#### Rotary Encoders
- **`Rotary_Encoder_Switch`** → Encoder with push switch
  - Library: `Device`
  - Footprint: `Rotary_Encoder:RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm`

#### Toggle Switches
- **`SW_SPDT`** → SPDT toggle
  - Library: `Switch`
  - Footprint: (generic, update later with actual part footprint)

#### LEDs
- **`LED_Dual_2pin`** or **`LED_RCGB`** → Dual-color LED
  - Library: `Device`
  - Or use 2× **`LED`** symbols (separate red/green)
  - Footprint: `LED_THT:LED_D5.0mm`

#### Resistors
- **`R`** → Pull-ups (10kΩ) and LED current limiting (220Ω)

### Circuits to Draw

**Encoder 1:**
```
ENC1:
  A ──┬─── 10kΩ pull-up ──> +3.3V
      └──> ENC1_A (to MCU sheet)

  B ──┬─── 10kΩ pull-up ──> +3.3V
      └──> ENC1_B

  SW ─┬─── 10kΩ pull-up ──> +3.3V
      └──> ENC1_SW

  COM ──> GND
```

**Toggle 1:**
```
TOG1 (SPDT):
  COM ──> +3.3V
  NO ──┬──> TOG1 (to MCU GPIO22)
       │
       └─── 10kΩ pull-down ──> GND
  NC ──> (not connected)
```

**LED 1 (dual-color):**
```
LED1:
  R_anode ── 220Ω ── LED1_R (from MCU GPIO6)
  G_anode ── 220Ω ── LED1_G (from MCU GPIO7)
  Common_cathode ──> GND
```

Repeat for Encoder 2, Toggle 2, LED 2.

---

## Component Libraries & Footprints

### Finding Symbols in KiCad

**Press `A` to add symbol**, then search:

| Component Type | Search Term | Library |
|----------------|-------------|---------|
| 2×5 connector | `Conn_02x05` | Connector_Generic |
| Schottky diode | `D_Schottky` | Device |
| TL074 op-amp | `TL074` | Amplifier_Operational |
| Resistor | `R` | Device |
| Capacitor | `C` | Device |
| Inductor | `L` | Device |
| Audio jack | `AudioJack` | Connector_Audio |
| Encoder | `Rotary_Encoder` | Device |
| Toggle switch | `SW_SPDT` | Switch |
| LED | `LED` | Device |
| Power symbols | `+12V`, `GND`, etc. | power |

### Assigning Footprints

After placing symbols, assign footprints:

**Tools → Assign Footprints** (or press `Ctrl+F`)

| Component | Footprint Library:Footprint |
|-----------|----------------------------|
| 0805 resistor | `Resistor_SMD:R_0805_2012Metric` |
| 0805 capacitor | `Capacitor_SMD:C_0805_2012Metric` |
| SOIC-14 (TL074) | `Package_SO:SOIC-14_3.9x8.7mm_P1.27mm` |
| SMA diode (SS14) | `Diode_SMD:D_SMA` |
| SOD-123 diode (BAT85) | `Diode_SMD:D_SOD-123` |
| TSOT-23-6 (AP63203) | `Package_TO_SOT_SMD:TSOT-23-6` |
| 2×5 IDC header | `Connector_IDC:IDC-Header_2x05_P2.54mm_Vertical` |
| Thonkiconn jack | (custom footprint needed) |
| PEC11 encoder | `Rotary_Encoder:RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm` |

**Custom footprints needed:**
- Thonkiconn PJ398SM (may need to create or download from Eurorack library)
- Raspberry Pi Pico module (if not in standard library)

---

## Workflow Checklist

### Phase 1: Setup
- [ ] Create KiCad project (`trace-eurorack-module.kicad_pro`)
- [ ] Set up 6 hierarchical sheets
- [ ] Add hierarchical pins to sheets for power/signal routing

### Phase 2: Draw Schematics
- [ ] **Sheet 1: Power Supply** (start here)
  - [ ] Draw Eurorack connector with correct pinout
  - [ ] Add reverse polarity protection
  - [ ] Draw buck regulator circuit
  - [ ] Add power distribution and decoupling

- [ ] **Sheet 2: Input Channel 1**
  - [ ] Draw input protection (R_PROT, D1, D2)
  - [ ] Draw voltage divider (R1, R2)
  - [ ] Draw offset reference (R3, R4, C1)
  - [ ] Draw summing amplifier (U1A)
  - [ ] Draw ADC protection (R7, D3, D4)
  - [ ] Draw passthrough buffer (U1B)

- [ ] **Sheet 3: Input Channel 2**
  - [ ] Duplicate Channel 1 circuit
  - [ ] Update designators
  - [ ] Add normalling circuit

- [ ] **Sheet 4: MCU & Display**
  - [ ] Place Raspberry Pi Pico symbol
  - [ ] Connect ADC inputs (GPIO26, 27)
  - [ ] Connect SPI display (GPIO10-15)
  - [ ] Add decoupling capacitors

- [ ] **Sheet 5: User Controls**
  - [ ] Draw encoders with pull-ups
  - [ ] Draw toggles with pull-downs
  - [ ] Draw LEDs with current limiting

### Phase 3: Validation
- [ ] Annotate all components (Tools → Annotate Schematic)
- [ ] Assign footprints to all components
- [ ] Run ERC (Inspect → Electrical Rules Checker)
- [ ] Fix all errors and warnings
- [ ] Add labels and documentation text

### Phase 4: Output
- [ ] Generate BOM (Tools → Generate BOM)
- [ ] Generate netlist for PCB (File → Export → Netlist)
- [ ] Review BOM against parts sourcing checklist
- [ ] Commit schematic files to git

---

## Tips & Best Practices

### Net Labels vs. Hierarchical Pins
- **Net labels** (press `L`): Connect signals within a sheet
- **Hierarchical pins**: Connect signals between sheets
- **Global labels** (press `Ctrl+L`): Connect signals across all sheets (use for power: +12V, -12V, +3.3V, GND)

### Power Symbols
- Use power symbols (`+12V`, `-12V`, `+3.3V`, `GND`) instead of wires everywhere
- Makes schematic cleaner and easier to read
- Power symbols are implicitly connected (no wires needed)

### Component Values
- Set value AND footprint for every component
- Use consistent notation (e.g., `100kΩ, 1%` not `100k`)
- Add manufacturer part number in "MPN" field (for BOM generation)

### Wire Routing
- Keep wires straight (horizontal/vertical, avoid diagonals)
- Use junction dots (automatic) where wires cross/connect
- Press `K` to end a wire segment, `W` to start new wire

### Saving Work
- Save often (`Ctrl+S`)
- KiCad auto-saves, but manual saves are safer
- Commit to git after completing each sheet

### Checking Your Work
- Zoom in/out with mouse wheel
- Pan with middle mouse button (or scroll bars)
- Use **Highlight Net** (right-click wire → Highlight Net) to trace connections
- Run ERC early and often (catches mistakes before PCB layout)

---

## Common Mistakes to Avoid

1. **Forgetting power pins on ICs** → TL074 needs pins 4 and 11 connected
2. **Wrong diode polarity** → Check cathode (marked end) orientation
3. **Missing decoupling caps** → Add 100nF near every IC power pin
4. **Incorrect Eurorack pinout** → Pin 1 is -12V (red stripe), verify before drawing
5. **GPIO conflicts** → Double-check GPIO assignments match design doc
6. **Forgetting pull-ups/pull-downs** → Encoders and toggles need these
7. **Wrong footprint size** → Verify 0805 vs 0603 vs 1206 before assigning

---

## Next Steps After Schematic Complete

1. **Run final ERC** → Fix all errors (warnings optional but review them)
2. **Generate BOM** → Compare against `docs/10-parts-sourcing-checklist.md`
3. **Export netlist** → This feeds into PCB layout
4. **Review with design doc** → Make sure nothing was missed
5. **Commit to git** → Save your work!
6. **Start PCB layout** → Import netlist into `.kicad_pcb` file

---

## Resources

- **KiCad Documentation**: https://docs.kicad.org/
- **Schematic Editor Manual**: https://docs.kicad.org/7.0/en/eeschema/eeschema.html
- **ModWiggler DIY Forum**: https://modwiggler.com/forum/viewforum.php?f=17
- **Design Reference**: `docs/11-schematic-design-plan.md`

---

## Ready to Start!

**Recommended order:**
1. Open KiCad, create project
2. Set up hierarchical sheets
3. Start with **Power Supply sheet** (easiest, foundational)
4. Move to **Input Channel 1** (most complex, do once)
5. Duplicate to **Input Channel 2** (quick)
6. Add **MCU & Display** (straightforward)
7. Finish with **User Controls** (simple)

Good luck! Reference the schematic design plan (`docs/11-schematic-design-plan.md`) for all component values and connections.
