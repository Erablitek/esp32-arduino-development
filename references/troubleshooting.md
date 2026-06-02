# ESP32 Arduino CLI Troubleshooting

## Upload failures

### "stray '##' in program" — `##line` in compiled output

**Error:**
```
error: stray '##' in program
    1 | ##line 11 "path/to/005_setup.ino"
```

**Cause:** Multi-file Arduino sketch (split into 001_*.ino, 002_*.ino, etc.) where the main `.ino` file does NOT contain explicit prototypes for `setup()` and `loop()`. arduino-cli's preprocessor auto-generates these prototypes and the first `#line` directive it writes is corrupted to `##line`. Reproducible with arduino-cli 1.5.0 + esp32:esp32 core 3.3.8.

**Fix:** Add `void setup(); void loop();` prototype declarations at the end of the main `.ino` file's prototypes section. Or migrate to `.h`/`.cpp` structure where `setup()`/`loop()` live directly in the `.ino`.

**Diagnostic:** Check the generated `.cpp` in the Arduino build cache; if the first lines show `##line` instead of `#line`, this is the bug.

### "Failed to connect to ESP32: Timed out waiting for packet header"

**Cause:** ESP32 is not in download/bootloader mode.

**Fix:**
1. Hold the **BOOT** button
2. Tap the **EN** (reset) button
3. Release **BOOT**
4. Retry the upload immediately

Some boards (NodeMCU-32S, most dev kits) have auto-reset circuitry and don't need
this. Boards with bare modules or broken auto-reset (bad CP2102, missing capacitor)
require manual intervention.

### "A fatal error occurred: Invalid head of packet"

**Cause:** Usually a baudrate mismatch, or the serial port is held open by another
process.

**Fix:**
1. Close any open serial monitors (including other terminals, PuTTY, etc.)
2. Lower `upload.upload_speed` in `project.json` to `115200`
3. Try a different USB cable — many "charging-only" cables have no data lines
4. On Linux, check for processes holding the port: `sudo lsof /dev/ttyUSB0`

### "No serial data received"

**Cause:** Driver not installed, or wrong port selected.

**Fix:**
- **Windows:** install CP210x (Silicon Labs) or CH340 driver depending on the USB-serial chip on your board
- **macOS:** Apple ships CP210x drivers since 2020 but they sometimes break — check System Report → USB
- **Linux:** drivers are built into the kernel, check `dmesg` when you plug in

## Compile failures

### "esp32:esp32 not installed"

### "expected constructor, destructor, or type conversion before '...'" (C++ comment bug)

**Error:**
```
sensors.cpp:8:50: error: expected constructor, destructor, or type conversion before 'pour'
    8 |  * a changé. Pas de flags pending*/sendToCloud() pour les données capteur.
```

**Cause:** A `/* */` block comment containing the sequence `*/` (e.g. `pending*/sendToCloud()`).
The C preprocessor treats the first `*/` as the end of the comment block, leaking the
remaining text into source code.

**Fix:** Replace `*/` inside block comments with `/` (e.g. `pending/sendToCloud`).
Never use `*/` inside a `/* */` block, even as part of a word.

### "region `iram0_0_seg' overflowed by N bytes"
arduino-cli core install esp32:esp32
# or with pinned version
arduino-cli core install esp32:esp32@<version>
```

### "<Library.h>: No such file or directory"

```bash
# Search for the library
arduino-cli lib search "LibraryName"

# Install it
arduino-cli lib install "LibraryName"

# Install specific version
arduino-cli lib install "ArduinoJson@6.21.3"
```

### "region `iram0_0_seg' overflowed by N bytes"

**Cause:** Too much code is forced into IRAM (instruction RAM). Often caused by
heavy use of `IRAM_ATTR` or interrupt handlers.

**Fix:**
- Review code for unnecessary `IRAM_ATTR` decorations
- Remove large literals from interrupt contexts
- Keep ISRs (Interrupt Service Routines) minimal

### "region `app0' overflowed by N bytes" (or `flash`)

**Cause:** Compiled firmware is too big for the app partition.

**Fix (in order of preference):**
1. Change `board.partition_scheme` in `project.json` to `huge_app` or `no_ota`
2. Create a custom `partitions.csv` with a larger app slot
3. Reduce firmware size: disable unused features, enable compiler optimizations
4. Upgrade to a board with more flash (8MB or 16MB)

### "xtensa-esp32-elf-gcc: not found"

### `__DATE__`/`__TIME__` stale after recompile (`compileDate` unchanged)

**Error:** After recompiling (especially after `rm -rf build`), the binary still contains
the old compile date. `strings firmware.bin | grep "May 30"` shows yesterday's timestamp.

**Cause:** `arduino-cli` caches compiled `.o` files globally under `~/.cache/arduino/sketches/`.
If a source file containing `__DATE__` or `__TIME__` (e.g. `globals.cpp` with
`const char compileDate[] = __DATE__ " " __TIME__`) hasn't changed between compiles,
the cached `.o` is reused even after deleting the local `build/` output dir.

**Fix:**
```bash
rm -rf ~/.cache/arduino/sketches/*    # global arduino-cli cache
rm -rf <project>/build                 # local output dir
arduino-cli compile ...
```

**Verify:**
```bash
strings <project>/<bin_name> | grep "$(date +%b) $(date +%d) $(date +%Y)"
```

### "xtensa-esp32-elf-gcc: not found"

**Cause:** Toolchain files are corrupt or missing.

**Fix:**
```bash
arduino-cli core uninstall esp32:esp32
arduino-cli cache clean
arduino-cli core install esp32:esp32@<version>
```

## Runtime failures

### ESP32 reboots continuously after upload

**Common causes:**
- Brown-out: PSU can't deliver enough current (ESP32 peaks at 500 mA during WiFi TX)
- Bad flash mode: try `dio` instead of `qio` in `project.json` `board.flash_mode`
- Wrong flash size declared: verify against the actual chip (esptool.py flash_id)

**Diagnostic:**
```bash
arduino-cli monitor -p <port> --config baudrate=115200
```

Read the first crash trace. If you see "Brownout detector was triggered", it's power.
If you see "invalid header", flash mode/size is wrong.

### Serial monitor shows garbage

**Cause:** baudrate mismatch.

**Fix:**
- Verify `monitor.baudrate` in `project.json` matches the `Serial.begin(XXXXX)` in code
- ESP32 boot messages (from ROM bootloader) are always at **74880 baud** — that's normal
- After boot, your code's baud takes over

### WiFi doesn't connect

**Diagnostic checklist:**
- Signal strength: `WiFi.RSSI()` should be better than -75 dBm
- Channel: some 5 GHz-only routers won't work (ESP32 classic is 2.4 GHz only)
- Password encoding: special characters sometimes need escaping
- Router may have MAC filtering or band steering enabled

## Linux-specific

### "Permission denied" on `/dev/ttyUSB0` or `/dev/ttyACM0`

```bash
# Add user to dialout group (permanent fix)
sudo usermod -a -G dialout $USER

# Log out and back in for group changes to take effect
# Or start a new shell with the group:
newgrp dialout
```

### `brltty` grabs USB-serial ports

On some distros (Ubuntu, Debian), `brltty` (screen reader for braille displays)
claims CH340 devices incorrectly.

```bash
sudo apt remove brltty
# or mask it
sudo systemctl mask brltty
```

## Windows-specific

### Arduino-cli not found after install

Add the install directory to your PATH:
1. Search "Environment Variables" in Start menu
2. Edit **User** variables → `Path`
3. Add `%USERPROFILE%\bin` (or wherever you extracted arduino-cli)
4. Close and reopen any terminal / VS Code

### COM port number keeps changing

Windows assigns COM ports per USB-serial chip serial number. If you use multiple
ESP32 boards with the same CH340 chip (which all share the same serial number),
Windows juggles them.

**Fix:** In Device Manager, right-click the COM port → Properties → Port Settings
→ Advanced → assign a fixed high COM number (e.g., COM20).

## macOS-specific

### Port appears as `/dev/cu.SLAB_USBtoUART` or `/dev/cu.usbserial-XXXX`

That's normal — macOS uses these names for CP210x and CH340 respectively. Use the
full path in `upload.port`.

### Gatekeeper blocks esptool

If you see "esptool can't be opened because Apple cannot check it for malicious
software":

```bash
xattr -d com.apple.quarantine ~/.arduino15/packages/esp32/tools/esptool_py/*/esptool
```

Or right-click the binary → Open → Open anyway.
