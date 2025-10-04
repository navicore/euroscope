# EuroScope - DIY 24HP Eurorack Oscilloscope

A custom 2-channel Eurorack oscilloscope module designed for audio and CV visualization with persistent settings.

## Design Philosophy

- **Function over size pressure**: 24HP for comfortable viewing and control
- **Tactile controls**: Encoders and buttons for performance-friendly operation
- **Focused scope**: Audio rate and sub-audio CV visualization
- **No signal loading**: High impedance inputs, clean buffered passthroughs
- **Persistent settings**: Settings survive power cycles

## Specifications

### Physical
- **Width**: 24HP (121.92mm)
- **Display**: 4.3" SPI TFT (non-touch)
- **Controls**: 2 rotary encoders, 2-3 momentary buttons

### Electrical
- **Input Channels**: 2
- **Input Range**: ±12V (safe), ±5V typical
- **Input Impedance**: >100kΩ
- **Coupling**: DC-coupled
- **Passthrough**: 2 buffered outputs
- **Power**: Eurorack ±12V bus

### Processing
- **MCU**: RP2040 (Raspberry Pi Pico)
- **ADC**: 12-bit, 0-3.3V range
- **Sample Rate**: 10-50kSPS (audio + sub-audio)
- **Firmware**: Rust (rp-hal/embassy-rp)

## Repository Structure

```
/design/          - Schematics, PCB layouts, mechanical drawings
/docs/            - Design decisions, calculations, reference material
/firmware/        - Rust firmware source
/bom/             - Bill of materials, part sourcing
```

## Current Status

**Phase**: Initial design and planning

See `/docs/` for detailed design decisions.

## License

This project uses a dual-license approach:

- **Firmware/Software**: MIT License (see LICENSE-SOFTWARE)
- **Hardware Designs**: Creative Commons BY-SA 4.0 (see LICENSE-HARDWARE)

This follows the practice of open hardware projects.
