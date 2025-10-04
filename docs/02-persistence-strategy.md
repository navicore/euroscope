# Settings Persistence Strategy

## Philosophy: Analog Behavior from Digital Module

**Goal**: Module should "just remember" settings like an analog module with its knobs in position.

- No save buttons
- No preset menus
- No load/recall operations
- Knob turn = automatic save (transparent to user)

## Persistence Approach: Write-Through with Debouncing

### User Experience

1. User turns encoder → UI updates instantly (RAM)
2. System waits 500ms-1s for changes to settle
3. Settings automatically write to flash (background, transparent)
4. On next power-up → settings restored automatically

**No user action required. Ever.**

### Flash Write Strategy

**Debounced writes**:
- Encoder changes update RAM immediately (instant feedback)
- Start/reset 500ms-1s timer on any setting change
- Timer expires with no new changes → write to flash
- Avoids excessive writes during knob turning

**Write operation**:
- ~100ms flash erase+write (non-blocking if possible)
- User never notices

## Data Integrity

### Multi-Copy Storage

Store settings in **2-3 flash locations**:
- Primary copy
- Backup copy 1
- Backup copy 2 (optional)

Each copy includes **CRC32 checksum**.

### Boot Sequence

```
Power On
  ↓
Read primary settings
  ↓
Valid CRC? → YES → Load settings ✓
  ↓ NO
Read backup settings
  ↓
Valid CRC? → YES → Load settings, repair primary
  ↓ NO
Load factory defaults
Show brief "DEFAULTS" indicator on screen
```

### Power Loss Protection

Flash writes can be interrupted:
- Always validate with CRC on read
- Multiple copies ensure recovery
- Worst case: fall back to previous good settings or defaults

## Wear Leveling

### RP2040 Flash Characteristics

- **Capacity**: 2MB total
- **Endurance**: ~10,000 write cycles per sector
- **Sector size**: 4KB minimum erase block

### Strategy

Rotate writes across **10 flash sectors**:
- Track write count per sector
- Use least-worn sector for next write
- 10 sectors × 10K writes = 100K lifetime saves

**Longevity estimate**:
- 100 saves/day = 36,500/year
- 100K saves = **~3 years** of heavy daily use
- Realistic use: **10+ years**

## Settings Data

### What Gets Saved

- Time/div (horizontal scale)
- Volts/div channel 1 (vertical scale)
- Volts/div channel 2 (vertical scale)
- Trigger source (Ch1/Ch2/Auto)
- Trigger level
- Trigger edge (rising/falling)
- Run/stop state (optional - could always start running)

**Data size**: ~16-32 bytes (tiny)

### What Doesn't Get Saved

- Current waveform buffer (resets on power up)
- Temporary UI state
- Diagnostic counters (stored separately)

## Usage Tracking

### Metrics Stored (Separate Flash Region)

- **Boot count**: Number of power cycles
- **Total saves**: Lifetime settings write count
- **Per-sector write count**: Wear leveling validation

### Diagnostic Screen

**Access**: Hold Button 3 for 3 seconds

**Display**:
```
=== DIAGNOSTICS ===
Power Cycles:  1,247
Total Saves:   8,432
Flash Errors:  0

Sector Wear:
 0: ████░░░░░░  823
 1: ████░░░░░░  819
 2: ███░░░░░░░  831
 [...]

[Any button to exit]
```

**Purpose**:
- User confidence (see module health)
- Development/debugging
- Validate wear leveling working correctly

## Factory Reset

**Trigger**: Hold Encoder 1 + Encoder 2 + Button 1 simultaneously for 3 seconds

**Action**:
- Clear all settings sectors
- Reset to factory defaults
- Clear usage counters? (TBD - maybe keep for historical data)
- Reboot
- Display "RESET" briefly on startup

**Use case**: Emergency recovery only (rare)

## Implementation Notes (Rust)

### Recommended Crates

- **sequential-storage**: Wear-leveling flash storage for embedded Rust
- **postcard**: Compact binary serialization (efficient for small settings struct)
- **crc**: CRC32 checksum validation

### Pseudocode Flow

```rust
// Settings change handler
fn on_encoder_change(setting: Setting, value: u8) {
    settings.update(setting, value);
    ui_refresh();  // Instant visual feedback
    save_timer.reset();  // Start/restart 500ms countdown
}

// Timer callback (500ms after last change)
fn on_save_timer_expire() {
    if settings.dirty {
        flash_write_with_crc(&settings);  // ~100ms
        settings.dirty = false;
        usage_stats.increment_save_count();
    }
}

// Boot sequence
fn init() {
    match flash_read_settings() {
        Ok(settings) => load(settings),
        Err(_) => {
            load_defaults();
            show_message("DEFAULTS", 2000ms);
        }
    }
}
```

## Confidence Level

This approach is proven in:
- Meris guitar pedals
- Strymon pedals
- Mutable Instruments Eurorack modules
- Consumer electronics (BIOS, cameras, etc.)

**Risk**: Very low with proper CRC validation and multi-copy storage

**Expected reliability**: Should "just work" for the life of the module
