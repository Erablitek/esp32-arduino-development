# Arduino CLI Installation

## Windows

**PowerShell (recommended):**
```powershell
Invoke-WebRequest -Uri "https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Windows_64bit.zip" -OutFile "arduino-cli.zip"
Expand-Archive -Path "arduino-cli.zip" -DestinationPath "$env:USERPROFILE\bin"
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:USERPROFILE\bin", "User")
```

Close and reopen any terminal after installing. Then:

```powershell
arduino-cli version
```

**Alternative: Scoop**
```powershell
scoop install arduino-cli
```

## macOS

**Homebrew (recommended):**
```bash
brew install arduino-cli
```

**Manual install:**
```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
sudo mv bin/arduino-cli /usr/local/bin/
```

## Linux

**Manual install (any distro):**
```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
sudo mv bin/arduino-cli /usr/local/bin/
```

**Arch:**
```bash
sudo pacman -S arduino-cli
```

**USB-serial access:** add yourself to the `dialout` group:
```bash
sudo usermod -a -G dialout $USER
# Then log out and back in
```

## Post-install: ESP32 core setup

After installing arduino-cli, configure it for ESP32:

```bash
# Create default config
arduino-cli config init

# Add the ESP32 board support URL
arduino-cli config add board_manager.additional_urls https://dl.espressif.com/dl/package_esp32_index.json

# Update the package index
arduino-cli core update-index

# Install the ESP32 core (latest)
arduino-cli core install esp32:esp32

# Or install a specific version
arduino-cli core install esp32:esp32@<version>

# Verify
arduino-cli core list
```

Expected output includes a line like:
```
ID             Installed   Latest     Name
esp32:esp32    <installed> <latest>   esp32
```

## Verify full installation

```bash
arduino-cli version                        # arduino-cli itself
arduino-cli core list                      # should show esp32:esp32
arduino-cli board listall esp32 | head     # should list ESP32 boards
```

## VS Code integration (optional)

Install these extensions for a better experience (none are required for this skill):

- **C/C++** (Microsoft) — IntelliSense for .ino/.cpp files
- **Arduino** (Microsoft) — note: this is deprecated but still works; if you want
  Arduino-aware IntelliSense without the full IDE, it's the easiest path
- **Serial Monitor** (Microsoft) — alternative to `arduino-cli monitor` with a GUI

With the tasks.json from this skill, you can bind the "ESP32: Compile + Upload"
task to a keybinding for one-key builds.
