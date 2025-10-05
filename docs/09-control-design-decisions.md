# Control Design Decisions

## Context

This document captures the design discussion around tactile controls, brand aesthetics, and the evolution from buttons to toggle switches for the Trace module.

---

## Key Decision: Per-Channel Independent Freeze

### The Problem
Initially designed with a single Run/Stop control affecting both channels globally. During design review, we identified a critical use case:

**Scenario**: Tuning one oscillator to match another's waveform
- Freeze Ch1 (reference waveform)
- Adjust Ch2 oscillator while watching live waveform
- Compare frozen Ch1 vs live Ch2 without losing the reference

### The Solution
**Independent Run/Stop control per channel**
- Ch1 has its own toggle switch + LED
- Ch2 has its own toggle switch + LED
- Each channel can be frozen/running independently
- Visual feedback shows state per channel

### Use Cases
1. **Waveform matching**: Freeze reference, tune second oscillator to match shape
2. **Envelope comparison**: Capture one envelope, trigger another later, compare overlaid
3. **Phase analysis**: Freeze one LFO, watch another drift/sync relative to it
4. **FM analysis**: Freeze carrier, watch modulated output change

---

## Key Decision: Toggle Switches Over Buttons

### The Button Problem
Many Eurorack modules use pushbuttons that feel cheap:
- 6mm tactile switches with plastic caps feel fragile
- Easy to accidentally bump during performance
- No definitive tactile feedback
- Imprecise action

### Observation
Quality modules (Intellijel, Random*Source, Doepfer) tend to use:
- 2-position and 3-position toggle switches
- Medium-size knobs
- Minimal or no pushbuttons
- When buttons exist, they often feel like the weak point

### Decision: SPDT Toggle Switches
**Advantages over buttons**:
- **Decisive tactile feedback**: Mechanical click, no ambiguity
- **Cannot accidentally bump**: Requires deliberate action
- **Visual state confirmation**: Physical toggle position + LED
- **Quality feel**: Matches aesthetic of premium modules
- **Proven in critical functions**: Industry standard for important controls

---

## Hardware Selection

### Toggle Switch Specifications
- **Type**: SPDT panel mount (2-position)
- **Candidates**:
  - C&K 7101 series (~$2-3)
  - E-Switch 100SP series (~$2-3)
- **Mounting**: Threaded bushing + nut (6mm thread)
- **Cost**: $2-3 per switch

### LED Indicators
- **Type**: Dual-color (red/green) common cathode
- **Size**: 5mm
- **Position**: Adjacent to toggle (3mm offset)
- **States**:
  - Green = Channel running (live signal)
  - Red = Channel stopped (frozen waveform)
- **Cost**: $0.50 per LED

### Why Separate LED vs Illuminated Toggle?
- **Cost**: $3 total (toggle + LED) vs $8-12 (illuminated toggle)
- **Flexibility**: Can position LED optimally for visibility
- **Availability**: Easier to source standard components
- **Same functionality**: User sees both physical toggle position and LED state

**Total per channel**: ~$3 (toggle + LED)

---

## Alternative Considered: Trigger Input Jack

### Observation
In modular patching, manual triggering is rare:
- Most users trigger from dedicated controllers (e.g., Make Noise Pressure Points)
- Tap tempo functions better served by external trigger jack
- Manual tactile triggers (if needed) use contact plates or other specialized controls

### Scope Triggering vs Manual Triggering
**Scope triggering** (what Trace needs):
- Signal processing function, not hardware button
- Trigger level = voltage threshold for waveform lock
- Trigger edge = rising/falling edge detection
- Controlled via encoder (adjust trigger level in UI)
- **Automatic** - firmware detects threshold crossing
- **No button needed**

**Manual trigger** (not needed):
- Would force single-shot capture
- Unnecessary for audio-rate scope
- Could be encoder push if needed

### Result
No dedicated trigger button. Triggering handled entirely in software with encoder control.

---

## Brand Aesthetics

### Design Philosophy
**Utilitarian silver aesthetic with distinctive accent**

### Preferences
- **Silver/brushed aluminum panels** - matches existing rack (Random*Source, Intellijel, Joranalogue, Schlappi)
- **Burgundy knobs** - distinctive accent color, not copying other manufacturers
- **Quality tactile controls** - toggles over cheap buttons
- **White silkscreen** - clean, legible labels

### What to Avoid
- **White PCB + gold accents** → looks like Non-Linear Circuits
- **All silver with silver knobs** → looks like Schlappi
- **Black panels** → contrary to personal aesthetic
- **Copying existing brand identities** → need distinctive look

### Material Considerations
- **Panel texture**: Matte or bead-blasted finish (reduces fingerprints, diffuses light)
- **Knob style**: TBD (Davies clone, Rogan, or custom - deferred decision)
- **Typography**: TBD (deferred decision)

---

## Final Control Layout

### Channel 1 (Upper Right)
- **Encoder**: Time/div control (with push for menu)
- **Toggle**: Ch1 Run/Stop
- **LED**: Dual-color (red/green) status

### Channel 2 (Lower Right)
- **Encoder**: Volts/div control (with push for menu)
- **Toggle**: Ch2 Run/Stop
- **LED**: Dual-color (red/green) status

### Bottom Row
- 4× Thonkiconn jacks (IN1, IN2, OUT1, OUT2)

### Panel Labels
- CH1 / TIME (encoder 1 + toggle 1)
- CH2 / VOLTS (encoder 2 + toggle 2)
- LED states visible (no text needed - green/red universally understood)

---

## Design Validation

### Tactile Quality ✓
- Toggle switches provide quality feel (no cheap buttons)
- Matches aesthetic of preferred modules (Intellijel, Random*Source)
- Decisive action prevents accidental state changes

### Functionality ✓
- Independent per-channel freeze enables critical use cases
- Visual feedback (toggle position + LED) confirms state
- Encoder push switches available for menu navigation

### Aesthetics ✓
- Silver panel with burgundy knobs creates distinctive look
- Quality components align with utilitarian philosophy
- Doesn't copy existing manufacturer aesthetics

### Cost ✓
- ~$3 per channel for toggle + LED (affordable)
- Total control cost: ~$18 (2 encoders, 2 toggles, 2 LEDs, knobs)
- Well within budget for quality components

---

## Lessons Learned

1. **Quality over cost**: $3 toggle + LED gives better UX than $1 cheap button
2. **Independent controls unlock features**: Per-channel freeze enables new workflows
3. **Tactile matters**: Toggle switches provide decisive action, buttons feel imprecise
4. **Aesthetics inform function**: Preference for quality modules led to better component selection
5. **Use case drives design**: Waveform matching scenario justified per-channel controls

---

## Next Steps

- [x] Document control hardware selection
- [x] Update panel layout with toggles + LEDs
- [x] Update cost estimate
- [ ] Source specific toggle switch part numbers
- [ ] Source dual-color LED part numbers
- [ ] Finalize knob style (burgundy Davies 1900H clones recommended)
- [ ] Create detailed panel CAD drawing with new layout
