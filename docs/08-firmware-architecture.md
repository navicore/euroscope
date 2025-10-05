# Firmware Architecture

## Development Strategy

**Hybrid approach**: Develop core logic on desktop, hardware drivers on embedded target.

### Phase 1: Desktop Development (Before Hardware)
- Core algorithms (trigger detection, waveform processing)
- Display rendering (using embedded-graphics simulator)
- Settings persistence logic
- Unit tests for all logic

### Phase 2: Hardware Integration (When PCB Arrives)
- ADC driver and sampling
- Real display via SPI (mipidsi)
- Encoder/button GPIO
- Performance tuning

**Iteration speed**: ~5 seconds (edit → build → flash via USB bootloader)

---

## Technology Stack

### Rust Embedded Ecosystem

**Hardware Abstraction Layer**:
- **embassy-rp** (recommended) - async/await, modern Rust
  - Built-in async ADC, SPI, GPIO
  - Efficient multitasking without RTOS overhead
  - Active development, great RP2040 support
- Alternative: **rp-hal** (blocking API, simpler but less efficient)

**Display Driver**:
- **mipidsi** crate - ILI9488/ILI9486 support
- **embedded-graphics** - drawing primitives, text, waveforms

**Storage**:
- **sequential-storage** - wear-leveling flash storage
- **postcard** - compact serialization for settings

**Input Handling**:
- Custom encoder state machine (GPIO + interrupts)
- Button debouncing (software or hardware timer-based)

### Project Structure

```
firmware/
├── Cargo.toml
├── src/
│   ├── main.rs              # Entry point, hardware init
│   ├── hardware/            # HAL-specific (RP2040 only)
│   │   ├── mod.rs
│   │   ├── adc.rs           # ADC sampling
│   │   ├── display.rs       # SPI display driver
│   │   ├── inputs.rs        # Encoders, buttons
│   │   └── storage.rs       # Flash persistence
│   ├── core/                # Platform-agnostic logic
│   │   ├── mod.rs
│   │   ├── waveform.rs      # Waveform buffer, circular buffer
│   │   ├── trigger.rs       # Trigger detection
│   │   ├── renderer.rs      # Waveform rendering to framebuffer
│   │   └── settings.rs      # Settings struct, defaults
│   └── simulator/           # Desktop testing (feature-gated)
│       ├── mod.rs
│       └── mock_hardware.rs # Simulated ADC, display window
└── tests/
    ├── trigger_tests.rs
    └── waveform_tests.rs
```

---

## Core Data Structures

### Waveform Buffer

**Circular buffer** for continuous ADC samples:

```rust
pub struct WaveformBuffer {
    samples: [u16; BUFFER_SIZE],  // ADC values (12-bit)
    write_index: usize,
    trigger_index: Option<usize>, // Where trigger occurred
}

const BUFFER_SIZE: usize = 2048; // 2K samples per channel
```

**Why circular**:
- Continuous sampling (no gaps)
- Trigger can occur anywhere in buffer
- Display window slides over buffer

### Settings

**Persistent configuration**:

```rust
#[derive(Serialize, Deserialize, Clone)]
pub struct ScopeSettings {
    // Timebase
    time_div: TimeDiv,              // µs, ms, s per division

    // Vertical (per channel)
    volts_div_ch1: VoltsDiv,        // V per division
    volts_div_ch2: VoltsDiv,

    // Trigger
    trigger_source: TriggerSource,  // Ch1, Ch2, Auto
    trigger_level: u16,             // ADC value (0-4095)
    trigger_edge: TriggerEdge,      // Rising, Falling

    // UI state
    run_mode: RunMode,              // Running, Stopped, Single
}

impl Default for ScopeSettings {
    fn default() -> Self {
        Self {
            time_div: TimeDiv::Ms1,     // 1ms/div
            volts_div_ch1: VoltsDiv::V1, // 1V/div
            volts_div_ch2: VoltsDiv::V1,
            trigger_source: TriggerSource::Ch1,
            trigger_level: 2048,        // Mid-scale
            trigger_edge: TriggerEdge::Rising,
            run_mode: RunMode::Running,
        }
    }
}
```

### Display State

**Framebuffer or direct rendering**:

```rust
pub struct DisplayState {
    // Pre-rendered waveform points (x,y pixel coordinates)
    ch1_points: Vec<(i32, i32), 480>, // Max 480 points (display width)
    ch2_points: Vec<(i32, i32), 480>,

    // UI elements
    grid_visible: bool,
    labels_visible: bool,
    trigger_indicator: Option<i32>, // X position of trigger
}
```

---

## Core Algorithms

### 1. ADC Sampling Strategy

**DMA-based round-robin sampling** (time-aligned channels):

```rust
// Pseudo-code
async fn adc_task(
    mut adc: Adc<'static>,
    dma_channel: DmaChannel,
    buffer_ch1: &mut WaveformBuffer,
    buffer_ch2: &mut WaveformBuffer,
) {
    // Circular DMA buffer: interleaved samples [CH0, CH1, CH0, CH1, ...]
    let mut dma_buffer = [0u16; 2048]; // 1024 samples per channel

    // Configure ADC for round-robin on GPIO26 (CH0) and GPIO27 (CH1)
    let adc_config = Config {
        round_robin: true,
        channels: &[Channel::Gpio26, Channel::Gpio27],
        sample_rate: calculate_sample_rate(settings.time_div),
    };

    // Start free-running ADC with DMA
    adc.run_free_running(&dma_channel, &mut dma_buffer, adc_config).await;

    loop {
        // Wait for DMA half-complete (double buffering)
        dma_channel.wait_half_complete().await;

        // Process interleaved samples
        let samples = &dma_buffer[0..1024]; // First half
        for chunk in samples.chunks_exact(2) {
            let sample_ch1 = chunk[0];
            let sample_ch2 = chunk[1];

            buffer_ch1.push(sample_ch1);
            buffer_ch2.push(sample_ch2);
        }

        // Check for trigger condition
        if check_trigger(buffer_ch1, &settings) {
            buffer_ch1.mark_trigger();
            signal_display_update();
        }
    }
}
```

**Why DMA + round-robin**:
- **Time-aligned samples**: CH1 and CH2 sampled back-to-back (few µs apart, consistent)
- **No CPU overhead**: DMA handles transfer while CPU processes
- **Continuous acquisition**: No gaps in sampling
- **Critical for dual-channel scope**: Phase relationships preserved

**Sample rate calculation**:
- Time/div setting determines pixels/second
- Display width: 480 pixels
- Example: 1ms/div × 10 div = 10ms full screen
  - Need 480 samples over 10ms = 48kSPS per channel
  - ADC runs at 96kSPS (48k × 2 channels round-robin)
- Adjust ADC sample rate based on time/div

### 2. Trigger Detection

**Edge trigger algorithm**:

```rust
pub fn check_trigger(
    buffer: &WaveformBuffer,
    settings: &ScopeSettings,
) -> bool {
    let current = buffer.latest_sample();
    let previous = buffer.previous_sample();

    match settings.trigger_edge {
        TriggerEdge::Rising => {
            previous < settings.trigger_level &&
            current >= settings.trigger_level
        }
        TriggerEdge::Falling => {
            previous > settings.trigger_level &&
            current <= settings.trigger_level
        }
    }
}
```

**Auto trigger** (if no trigger found):
- Timeout after N samples
- Force trigger at arbitrary point
- Keeps display updating even with no signal

### 3. Waveform Rendering

**Convert ADC samples to screen coordinates**:

```rust
pub fn render_waveform(
    buffer: &WaveformBuffer,
    settings: &ScopeSettings,
    display_width: u32,
    display_height: u32,
) -> Vec<(i32, i32), 480> {
    let mut points = Vec::new();

    // Get display window around trigger point
    let window = buffer.get_window_around_trigger(display_width as usize);

    for (x, sample) in window.iter().enumerate() {
        // Scale ADC value (0-4095) to screen Y (0-display_height)
        let y = map_adc_to_screen(
            *sample,
            settings.volts_div,
            display_height
        );

        points.push((x as i32, y as i32));
    }

    points
}

fn map_adc_to_screen(
    adc_value: u16,
    volts_div: VoltsDiv,
    screen_height: u32,
) -> i32 {
    // Guard against division by zero
    if screen_height == 0 {
        return 0;
    }

    // ADC: 0 = 0V, 4095 = 3.3V (after our offset circuit: 0.06V-3.4V range)
    // Center screen at 1.65V (ADC ~2048)

    let center_adc = 2048;
    let offset_from_center = (adc_value as i32) - center_adc;

    // Scale based on volts/div
    let pixels_per_adc = screen_height as f32 / (volts_div.to_adc_range() as f32);

    let y_offset = (offset_from_center as f32 * pixels_per_adc) as i32;
    let y = (screen_height / 2) as i32 - y_offset; // Invert Y (screen grows down)

    y.clamp(0, screen_height as i32 - 1)
}
```

### 4. Settings Persistence

**Save on change, load on boot**:

```rust
use sequential_storage::cache::NoCache;
use sequential_storage::map;

pub async fn save_settings(
    flash: &mut Flash<'static, FLASH>,
    settings: &ScopeSettings,
) -> Result<(), Error> {
    // Serialize settings
    let data = postcard::to_vec::<_, 128>(settings)?;

    // Write to flash with wear leveling
    map::store_item(
        flash,
        SETTINGS_RANGE, // Flash address range
        &mut NoCache::new(),
        &data,
        SETTINGS_KEY,
    ).await?;

    Ok(())
}

pub async fn load_settings(
    flash: &mut Flash<'static, FLASH>,
) -> ScopeSettings {
    match map::fetch_item::<Vec<u8, 128>>(
        flash,
        SETTINGS_RANGE,
        &mut NoCache::new(),
        SETTINGS_KEY,
    ).await {
        Ok(Some(data)) => {
            postcard::from_bytes(&data).unwrap_or_default()
        }
        _ => ScopeSettings::default()
    }
}
```

**Debounced writes**:
- Don't save on every encoder click
- Start 500ms timer on setting change
- Save when timer expires (no new changes)
- Prevents flash wear from rapid knob turning

---

## Input Handling

### Encoder State Machine

**Quadrature decoding**:

```rust
pub struct Encoder {
    pin_a: Input<'static>,
    pin_b: Input<'static>,
    last_state: u8,
    count: i32,
}

impl Encoder {
    pub async fn read_delta(&mut self) -> i32 {
        let a = self.pin_a.is_high() as u8;
        let b = self.pin_b.is_high() as u8;
        let current_state = (a << 1) | b;

        let delta = match (self.last_state, current_state) {
            (0b00, 0b01) | (0b01, 0b11) | (0b11, 0b10) | (0b10, 0b00) => 1,
            (0b00, 0b10) | (0b10, 0b11) | (0b11, 0b01) | (0b01, 0b00) => -1,
            _ => 0, // No change or invalid
        };

        self.last_state = current_state;
        self.count += delta;

        delta
    }
}
```

**Encoder tasks** (one per encoder):

```rust
#[embassy_executor::task]
async fn encoder1_task(
    mut encoder: Encoder,
    settings_channel: Sender<'static, SettingChange>,
) {
    loop {
        let delta = encoder.read_delta().await;

        if delta != 0 {
            // Encoder 1 controls time/div
            settings_channel.send(SettingChange::TimeDiv(delta)).await;
        }

        Timer::after_millis(5).await; // Poll at 200Hz
    }
}
```

### Button Debouncing

**Software debounce** (simple timer-based):

```rust
pub struct DebouncedButton {
    pin: Input<'static>,
    debounce_time: Duration,
}

impl DebouncedButton {
    pub async fn wait_for_press(&mut self) {
        // Wait for press
        self.pin.wait_for_low().await;

        // Debounce delay
        Timer::after(self.debounce_time).await;

        // Verify still pressed
        if self.pin.is_low() {
            // Wait for release
            self.pin.wait_for_high().await;
            Timer::after(self.debounce_time).await;
        }
    }
}
```

---

## Task Architecture (embassy-rp)

**Concurrent tasks** using async/await:

```rust
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Hardware init
    let p = embassy_rp::init(Default::default());

    // Channels for inter-task communication
    let (setting_tx, setting_rx) = channel::<SettingChange>();
    let (display_tx, display_rx) = channel::<DisplayUpdate>();

    // Spawn tasks
    spawner.spawn(adc_task(p.ADC, setting_rx)).unwrap();
    spawner.spawn(display_task(p.SPI0, display_rx)).unwrap();
    spawner.spawn(encoder1_task(p.PIN_0, setting_tx.clone())).unwrap();
    spawner.spawn(encoder2_task(p.PIN_2, setting_tx.clone())).unwrap();
    spawner.spawn(button_task(p.PIN_4, setting_tx.clone())).unwrap();
    spawner.spawn(settings_save_task(p.FLASH, setting_rx)).unwrap();
}
```

**Task responsibilities**:

1. **ADC Task**: Sample channels, detect trigger, update waveform buffer
2. **Display Task**: Render waveforms, UI elements, update screen
3. **Encoder Tasks**: Read encoder state, send setting changes
4. **Button Tasks**: Handle button presses (run/stop, trigger mode)
5. **Settings Task**: Debounce and save settings to flash

**Communication**: Embassy channels (async MPSC queues)

---

## Display Rendering Pipeline

### Frame Update Sequence

1. **Trigger occurs** → ADC task signals display task
2. **Display task wakes** → fetch waveform window from buffer
3. **Render to framebuffer** (or direct draw):
   - Clear screen (or draw grid background)
   - Draw channel 1 waveform (line or dots)
   - Draw channel 2 waveform (different color)
   - Draw UI overlays (trigger level, volts/div labels)
4. **Flush to display** → SPI transfer

**Optimization**: Only update when needed (trigger, settings change, or timeout)

### embedded-graphics Drawing

```rust
use embedded_graphics::{
    pixelcolor::Rgb565,
    prelude::*,
    primitives::{Line, PrimitiveStyle},
    text::{Text, TextStyle},
    mono_font::MonoTextStyle,
};

pub fn draw_waveform(
    display: &mut impl DrawTarget<Color = Rgb565>,
    points: &[(i32, i32)],
    color: Rgb565,
) -> Result<(), E> {
    let style = PrimitiveStyle::with_stroke(color, 1);

    for window in points.windows(2) {
        let (x1, y1) = window[0];
        let (x2, y2) = window[1];

        Line::new(
            Point::new(x1, y1),
            Point::new(x2, y2),
        )
        .into_styled(style)
        .draw(display)?;
    }

    Ok(())
}
```

---

## Desktop Simulator (Development)

**Mock hardware for testing on PC**:

```rust
#[cfg(feature = "simulator")]
mod simulator {
    use embedded_graphics_simulator::{
        SimulatorDisplay, Window, OutputSettingsBuilder,
    };

    pub fn run_simulator() {
        let mut display = SimulatorDisplay::<Rgb565>::new(Size::new(480, 272));

        // Mock ADC data (sine wave, square wave, etc.)
        let mock_samples = generate_test_waveform();

        // Run core rendering logic
        let settings = ScopeSettings::default();
        let points = render_waveform(&mock_samples, &settings, 480, 272);

        draw_waveform(&mut display, &points, Rgb565::GREEN);

        // Show in window
        let output_settings = OutputSettingsBuilder::new().build();
        Window::new("Trace Simulator", &output_settings).show_static(&display);
    }
}
```

**Development workflow**:
1. Write algorithm in `core/` module
2. Test with simulator (instant feedback)
3. Run unit tests (`cargo test`)
4. Flash to hardware for real-world testing

---

## Performance Considerations

### ADC Sample Rate

**RP2040 ADC**: 500kSPS max, 12-bit
- For audio scope: 10-50kSPS adequate
- Nyquist: 20kHz audio needs 40kSPS minimum
- Use 48kSPS for clean audio + sub-audio CV

**Sample rate vs time/div**:
- Fast time/div (µs range): Higher sample rate
- Slow time/div (ms/s range): Lower sample rate, average samples

### Display Update Rate

**Target**: 30 FPS minimum (33ms per frame)

**Budget**:
- Trigger detection: <1ms
- Waveform rendering: <5ms (480 points × 2 channels)
- SPI transfer: <10ms (480×272 @ 10MHz SPI)
- **Total**: ~16ms → 60 FPS achievable

**Optimization**:
- Only redraw on trigger or settings change
- Use DMA for SPI transfer (non-blocking)
- Render to framebuffer, then bulk transfer

### Flash Wear Leveling

**Flash endurance**: ~10,000 cycles per sector
**sequential-storage** handles rotation automatically

**Write frequency**:
- Debounce setting changes (500ms delay)
- Rotate across 10 sectors → 100K total writes
- At 100 saves/day → ~3 years

**Monitoring**: Track write count in diagnostics screen

---

## Error Handling Strategy

**Embedded errors** (no panic in production):

```rust
pub enum ScopeError {
    AdcError,
    DisplayError,
    FlashError,
    InvalidSettings,
}

// Graceful degradation
match load_settings(&mut flash).await {
    Ok(settings) => settings,
    Err(_) => {
        // Log error (optional UART debug)
        // Fall back to defaults
        ScopeSettings::default()
    }
}
```

**Display error indicators**:
- Red LED blink pattern for critical errors
- On-screen error message if display works
- UART debug output (optional debug header)

---

## Development Milestones

### Milestone 1: Core Logic (Desktop)
- [ ] Waveform buffer implementation
- [ ] Trigger detection algorithm
- [ ] Waveform rendering to simulated display
- [ ] Settings struct and serialization
- [ ] Unit tests for all core logic

### Milestone 2: Hardware Drivers (RP2040)
- [ ] Embassy project setup
- [ ] ADC driver and continuous sampling
- [ ] Display driver (mipidsi + embedded-graphics)
- [ ] Encoder GPIO and state machine
- [ ] Button input with debouncing

### Milestone 3: Integration (PCB Required)
- [ ] ADC → waveform buffer → display pipeline
- [ ] Trigger detection on real signals
- [ ] Encoder/button control of settings
- [ ] Flash persistence working
- [ ] Performance tuning (sample rate, FPS)

### Milestone 4: Polish
- [ ] UI refinement (grid, labels, colors)
- [ ] Diagnostics screen (boot count, saves, etc.)
- [ ] Factory reset (button combo)
- [ ] User testing and tweaks

---

## Rust Crates and Dependencies

**Cargo.toml** (embedded target):

```toml
[dependencies]
embassy-rp = { version = "0.1", features = ["time-driver"] }
embassy-executor = { version = "0.5", features = ["arch-cortex-m", "executor-thread"] }
embassy-time = "0.3"

# Display
mipidsi = "0.7"
embedded-graphics = "0.8"
display-interface = "0.5"

# Storage
sequential-storage = "2.0"
postcard = "1.0"
serde = { version = "1.0", default-features = false }

# Utilities
heapless = "0.8"  # Vec without heap allocation
defmt = "0.3"     # Efficient logging
```

**For desktop simulator** (feature-gated):

```toml
[dev-dependencies]
embedded-graphics-simulator = "0.5"
```

---

## Tooling Setup

### Initial Setup

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add ARM Cortex-M0+ target (RP2040)
rustup target add thumbv6m-none-eabi

# Install tools
cargo install probe-rs --features cli
cargo install elf2uf2-rs  # Convert ELF to UF2 for bootloader
```

### Build and Flash

```bash
# Build for RP2040
cargo build --release --target thumbv6m-none-eabi

# Flash via USB bootloader (drag-and-drop)
elf2uf2-rs target/thumbv6m-none-eabi/release/trace

# Or flash via SWD debugger
probe-rs run --chip RP2040 target/thumbv6m-none-eabi/release/trace
```

### Debugging

**Option 1: USB bootloader** (no debugger needed)
- Hold BOOTSEL button, plug in USB
- Drag .uf2 file to drive
- Reset and run

**Option 2: SWD debugger** (real debugging)
- Use another Pico as picoprobe
- Or use J-Link, ST-Link, etc.
- Set breakpoints, inspect variables in VS Code

---

## Testing Strategy

### Unit Tests (Desktop)

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_rising_edge_trigger() {
        let mut buffer = WaveformBuffer::new();
        let settings = ScopeSettings {
            trigger_level: 2048,
            trigger_edge: TriggerEdge::Rising,
            ..Default::default()
        };

        buffer.push(2000); // Below threshold
        buffer.push(2100); // Crosses threshold

        assert!(check_trigger(&buffer, &settings));
    }

    #[test]
    fn test_waveform_scaling() {
        let points = render_waveform(
            &mock_buffer(),
            &ScopeSettings::default(),
            480,
            272,
        );

        // Verify points are within screen bounds
        for (x, y) in points {
            assert!(x >= 0 && x < 480);
            assert!(y >= 0 && y < 272);
        }
    }
}
```

### Integration Tests (Hardware)

**Test scenarios**:
1. Known test signals (sine, square, triangle from function generator)
2. Trigger detection accuracy
3. Sample rate verification (measure with scope)
4. Display update rate (FPS counter)
5. Settings persistence (power cycle test)

---

## Next Steps

1. **Set up Rust project** (`cargo new --lib trace-firmware`)
2. **Implement core logic** (waveform buffer, trigger, renderer)
3. **Desktop simulator** (test rendering with mock data)
4. **Unit tests** (verify algorithms)
5. **Wait for PCB** (or get a Pico for early hardware tests)
6. **Hardware integration** (embassy-rp tasks)
7. **Real signal testing** (patch cables, CV, audio)

---

## Open Questions / Future Enhancements

- [ ] XY mode (Lissajous figures) - feed CH1→X, CH2→Y
- [ ] FFT / spectrum analyzer mode (math-heavy, may need optimization)
- [ ] Waveform measurements (Vpp, frequency, duty cycle)
- [ ] Multiple trigger modes (normal, auto, single)
- [ ] Preset banks (save/recall multiple settings)
- [ ] USB communication (settings backup, waveform export)

**For V1**: Focus on solid scope functionality. Add features in V2 based on real-world use.
