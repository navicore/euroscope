# Design Decisions Log

## Module Size: 24HP ✓

**Decision**: 24HP width (no size pressure)

**Rationale**:
- Function deserves space over market trends
- Comfortable viewing area for 4.3" display
- Room for tactile controls
- Easy component layout and assembly

## Input Channels: 2 ✓

**Decision**: 2 channels only

**Rationale**:
- Covers 99% of use cases (compare two signals)
- Simpler analog design to learn from
- Less PCB complexity
- Adequate scope functionality

## Frequency Range: Audio + Sub-Audio ✓

**Decision**: DC to ~20kHz (audio rate and below)

**Rationale**:
- Core use case: See waveform shape and amplitude
- No RF/high-speed scope needed
- Simplifies ADC requirements (10-50kSPS adequate)
- RP2040 built-in ADC is sufficient

## Display: 4.3" Non-Touch SPI TFT ✓

**Decision**: 4.3" SPI display without touch

**Rationale**:
- Fits perfectly in 24HP with room for jacks
- Touch screens are problematic during live performance
- SPI has excellent Rust embedded-graphics support
- ~150-200mA power draw is comfortable for Eurorack bus
- Good visibility without being overpowered

## Controls: Tactile (Encoders + Buttons) ✓

**Decision**: 2 rotary encoders, 2-3 buttons (all tactile)

**Rationale**:
- Performance-friendly (no touch screen issues)
- Muscle memory for improvisation
- Immediate response, no menu diving
- Simple, focused UI

**Control mapping**:
- Encoder 1: Time/div (horizontal)
- Encoder 2: Volts/div (vertical)
- Button 1: Trigger source/mode
- Button 2: Run/Stop/Single
- Button 3: Save/recall (optional)

## MCU Platform: RP2040 ✓

**Decision**: Raspberry Pi RP2040

**Rationale**:
- Excellent Rust support (rp-hal, embassy)
- Built-in 12-bit ADC, 500kSPS (adequate for audio/CV)
- Dual Cortex-M0+ @ 133MHz (plenty fast)
- DMA for display updates
- Flash storage for persistent settings
- Cheap ($4) and readily available
- Room to grow (can swap later if needed)

## Firmware Language: Rust ✓

**Decision**: Rust for embedded firmware

**Rationale**:
- Your preferred embedded language
- Strong RP2040 ecosystem
- embedded-graphics for display
- Type safety for signal processing

## Signal Coupling: DC ✓

**Decision**: DC-coupled inputs

**Rationale**:
- Need to see CV offsets and DC levels
- Essential for envelope and LFO visualization
- Sub-audio rate requires DC coupling
- AC coupling could be future enhancement

## Passthrough Outputs: Buffered ✓

**Decision**: 2 buffered passthrough jacks

**Rationale**:
- No loading of input sources
- Act as clean mult/monitor points
- Unity-gain buffers (no signal modification)
- Useful for performance patching

## Power Supply: Direct ±12V for Analog ✓

**Decision**: Use Eurorack ±12V directly for op-amps

**Rationale**:
- Symmetric bipolar signals (cleaner design)
- No virtual ground complexity (easier to learn)
- Standard Eurorack approach
- Generate +3.3V from +12V via LDO for digital

## Persistent Settings: Yes ✓

**Decision**: Store scope settings in RP2040 flash

**Rationale**:
- Core feature (reason for building this vs buying Mordax)
- RP2040 has built-in flash
- Settings survive power cycles
- No tedious re-dialing on rack power-up
