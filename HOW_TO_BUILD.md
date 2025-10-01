# How to Build ATC_TLSR_Paper Firmware

## Quick Start

### First Time Setup (macOS)

Since you're on macOS ARM64 and the compiler requires Linux x86_64, you need Docker:

1. **Ensure Docker is running:**
   ```bash
   # Start Docker Desktop from Applications, or:
   open -a Docker

   # Verify Docker is running:
   docker ps
   ```

2. **Navigate to Firmware directory:**
   ```bash
   cd /Users/skytsai/code/ATC_TLSR_Paper/Firmware
   ```

3. **Build the firmware (first time - takes 3-5 minutes):**
   ```bash
   ./build-docker.sh
   ```

### Subsequent Builds (Fast - 10-20 seconds)

After the first build, Docker caches everything:

```bash
cd /Users/skytsai/code/ATC_TLSR_Paper/Firmware
./build-docker.sh
```

Or use Docker directly:

```bash
cd /Users/skytsai/code/ATC_TLSR_Paper/Firmware
docker run --platform linux/amd64 --rm -v "$(pwd)":/firmware atc-firmware-builder
```

## Output

After successful build:
- ✅ `ATC_Paper.bin` - Main firmware file (89KB)
- Located in: `/Users/skytsai/code/ATC_TLSR_Paper/Firmware/ATC_Paper.bin`

## Interactive Docker Shell (for debugging)

```bash
cd /Users/skytsai/code/ATC_TLSR_Paper/Firmware
docker run --platform linux/amd64 --rm -it -v "$(pwd)":/firmware atc-firmware-builder /bin/bash

# Inside container:
make              # Compile
ls -la            # List files
exit              # Exit container
```

## When to Rebuild

Only rebuild when you modify source files:
- `src/` - Main source code
- `components/` - Component libraries
- `static_src/` - Static sources

## Flashing the Firmware

After building, flash using web tools:

1. **OTA (Bluetooth):** https://atc1441.github.io/ATC_TLSR_Paper_OTA_writing.html
2. **UART (Serial):** https://atc1441.github.io/ATC_TLSR_Paper_UART_Flasher.html

## Understanding the Firmware

### Telink Signature "KNLT"

The compiled firmware contains a Telink signature at byte offset 0x08:
- **"KNLT"** = "TLNK" reversed (Telink spelled backwards)
- **Hex:** `4b4e4c54`
- **Purpose:** Validates firmware is authentic Telink binary

### Build Details

- **Compiler:** tc32-elf-gcc (Telink toolchain)
- **Platform:** Linux x86_64 (via Docker on macOS ARM64)
- **Output format:** ELF binary converted to .bin
- **Memory layout:**
  - Text: 76,516 bytes
  - Data: 4,788 bytes
  - BSS: 25,345 bytes
  - Total: 106,649 bytes

## Troubleshooting

### Docker not running
```bash
open -a Docker
# Wait 10-30 seconds for Docker to start
```

### Permission denied on build-docker.sh
```bash
chmod +x build-docker.sh
./build-docker.sh
```

### Clean rebuild (if something breaks)
```bash
docker system prune -a  # Remove all Docker cache
./build-docker.sh       # Fresh build from scratch
```

## Alternative: Use Pre-compiled Firmware

If you don't need to modify the code, use the existing firmware:
- Already available: `Firmware/ATC_Paper.bin`
- No build required!

## Web Tools Source Code

Downloaded web tools are available locally:
- `ATC_TLSR_Paper_Image_Upload.html` - Bluetooth image uploader
- `ATC_TLSR_Paper_OTA_writing.html` - Bluetooth OTA firmware flasher with custom name setting

## Custom Features Added

### Custom Name Display
- Replaces MAC address (ESL_XXXXXX) with custom text
- Max 15 characters
- Stored in flash memory (persists across reboots)
- Set via OTA web tool

**To set custom name:**
1. Open `ATC_TLSR_Paper_OTA_writing.html`
2. Connect to device
3. Enter name in "Custom Name" field (e.g., "Room 1", "Office")
4. Click "Set Custom Name"
5. Display will show: "Room 1 BW213" instead of "ESL_A1B2C3 BW213"

### Date Display (yyyy/mm/dd)
- Changed from HH:MM time format to yyyy/mm/dd date format
- Automatically calculated from Unix timestamp
- Set date/time using "Set Time" button in OTA tool

## Build Process Explained

1. **Docker builds image** (first time only):
   - Ubuntu 20.04 x86_64
   - Installs: make, python3, gcc
   - Sets up tc32-elf-gcc toolchain

2. **Compilation:**
   - Runs `make` in Docker container
   - Compiles C sources with tc32-elf-gcc
   - Links into ELF executable
   - Converts to .bin format
   - Outputs `ATC_Paper.bin`

3. **Docker caching:**
   - First build: ~3-5 minutes
   - Later builds: ~10-20 seconds

## Quick Reference

```bash
# Build firmware
cd /Users/skytsai/code/ATC_TLSR_Paper/Firmware
./build-docker.sh

# Check output
ls -lh ATC_Paper.bin

# Interactive debugging
docker run --platform linux/amd64 --rm -it -v "$(pwd)":/firmware atc-firmware-builder /bin/bash
```

---

**Last updated:** 2025-10-01
**Firmware version:** ATC_TLSR_Paper
**Target hardware:** TLSR8359 ARM SOC (Hanshow E-Paper tags)
