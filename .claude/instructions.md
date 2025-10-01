# ATC TLSR Paper E-Ink Display Project

## Project Overview
Firmware for Telink TLSR E-Paper displays with BLE control interface. Used for food storage tracking in refrigerators.

## Build Instructions
Always use incremental Docker build (not full rebuild):
```bash
cd /Users/skytsai/code/ATC_TLSR_Paper/Firmware
docker run --rm -v "$(pwd)":/firmware atc-firmware-builder make
```

## Timezone Configuration
- **Target timezone**: Taiwan (UTC+8)
- **HTML hour offset**: 8 (default value in interface)
- **Firmware**: Displays time as-is (no additional offset)

## Display Behavior
The e-paper display shows:
1. **"Stored: YYYY/MM/DD HH:MM"** - Fixed timestamp when food was placed in refrigerator (doesn't change)
2. **"Days: X"** - Number of days elapsed since storage start time (auto-increments)

### Important: Time vs Storage Timer
- Device maintains internal clock (`time_is` / `current_unix_time`)
- Display shows `storage_start_time` (not current time) when timer is active
- Storage timer is the reference point for counting days

## Key Features Implemented
1. **Combined Set Time & Start Storage Timer**
   - Single button sets both device time and storage start time
   - Automatically refreshes display after setting
   - Function: `setTimeAndStartTimer()` in HTML

2. **Auto-refresh triggers**
   - Setting storage timer (command 0xE2) → immediate display refresh
   - Setting time (command 0xDD) → no auto-refresh
   - Normal operation → refreshes every minute, full refresh every hour

3. **Custom name support**
   - Max 15 characters
   - Stored in flash
   - Command 0xE1

## BLE Commands (via cmd_parser.c)
- `0xDD` - Set device time (4-byte Unix timestamp, big-endian)
- `0xE0` - Force EPD model
- `0xE1` - Set custom name (15 chars max + null terminator)
- `0xE2` - Set storage start time (4-byte Unix timestamp) + auto-refresh display
- `0xDE` - Reset settings to default
- `0xDF` - Save current settings to flash

## File Locations
- **Firmware source**: `Firmware/src/`
  - `epd.c` - Display rendering and time formatting
  - `cmd_parser.c` - BLE command handling
  - `time.c` - Internal clock management
- **Web interface**: `ATC_TLSR_Paper_OTA_writing.html` (root directory)
- **Build output**: `Firmware/ATC_Paper.bin`

## Recent Changes (2025-10-01)
1. Simplified HTML interface to single "Set Time & Start Storage Timer" button
2. Removed separate "Set Time" and "Start Storage Timer Now" buttons
3. Added auto-refresh on storage timer start (0xE2 command)
4. Fixed timezone handling - firmware displays time as-is, HTML adds UTC+8 offset
5. Display always shows storage start time (fixed), not current time

## Usage Workflow
1. Connect to device via BLE using HTML interface
2. Click "Set Time & Start Storage Timer" button
3. Display immediately shows: "Stored: [current date/time]" and "Days: 0"
4. Days counter increments automatically based on device's internal clock
5. Use "Clear Storage Timer" to reset

## Notes
- HTML `Date.now()` returns UTC time, so +8 hours offset needed for Taiwan time
- Display refreshes on command 0xE2, shows stored time from `settings.storage_start_time`
- If storage timer not set, display falls back to current time (`time_is`)
