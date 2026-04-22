---
name: esp32-arduino-development
description: |
  Complete ESP32 firmware development workflow using Arduino CLI (no Arduino IDE required).
  Use this skill whenever the user mentions ESP32, Arduino CLI, compiling or flashing
  firmware, serial monitoring, partition schemes (including custom partitions.csv files),
  arduino-esp32 core version management, or managing ESP32 projects from VS Code.
  Also trigger on errors like "Board not found", "Failed to connect to ESP32",
  "Timed out waiting for packet header", partition overflow, or when the user asks
  to create, configure, compile, upload, monitor, or check for core updates on an
  ESP32 project. Supports all ESP32 variants (ESP32, S2, S3, C3, C6, H2, etc.) and
  pins the arduino-esp32 core version per project for reproducible builds. This skill
  replaces the Arduino IDE entirely and drives arduino-cli directly, reading project
  settings from a project.json file in each project.
author: esp32-arduino-development contributors
version: 1.1.0
date: 2026-04-22
---

# ESP32 Arduino Development (Arduino CLI Workflow)

## Purpose

Replaces the Arduino IDE with a fully CLI-driven workflow for ESP32 firmware
development. Each project contains a `project.json` file that defines the board,
port, framework version, build options, and metadata. Claude reads this file and
drives `arduino-cli` directly to compile, flash, and monitor.

**No custom build scripts required** — the skill itself orchestrates `arduino-cli`.
An optional `.vscode/tasks.json` template is provided as a Claude-independent
fallback for builds from within VS Code.

---

## Core actions

| Action | Trigger phrases | What it does |
|---|---|---|
| **Compile** | "compile", "build" | Builds the `.ino` with FQBN + options from `project.json` |
| **Flash** | "flash", "upload", "téléverse" | Compiles + uploads to ESP32 via serial |
| **Monitor** | "monitor", "serial", "moniteur" | Opens serial monitor at project baudrate |
| **Create project** | "new project", "create project.json" | Interactive project setup |
| **Check updates** | "check updates", "check framework updates" | Scans for arduino-esp32 core updates |
| **Diagnose** | Automatic on error | Parses errors, proposes fixes |

---

## project.json schema

```json
{
  "metadata": {
    "name": "project-name",
    "description": "What this firmware does",
    "author": "Your name",
    "client": "Internal / customer name (optional)",
    "firmware_version": "see main .ino file"
  },
  "framework": {
    "core": "esp32:esp32",
    "core_version": "<X.Y.Z>"
  },
  "board": {
    "fqbn": "esp32:esp32:esp32",
    "cpu_freq_mhz": 240,
    "flash_size": "4MB",
    "flash_mode": "dio",
    "flash_freq": "80",
    "partition_scheme": "default"
  },
  "upload": {
    "port": "auto",
    "upload_speed": 921600,
    "verify": true
  },
  "monitor": {
    "baudrate": 115200
  },
  "build": {
    "debug_level": "none",
    "extra_flags": []
  }
}
```

### Field reference

| Field | Values | Notes |
|---|---|---|
| `framework.core` | `esp32:esp32` | Always this for ESP32 work |
| `framework.core_version` | Exact version string in `X.Y.Z` format | **Pinned** — enforced before every compile. Never invent a value; always fetch from `arduino-cli core search esp32` |
| `board.fqbn` | See `references/fqbn-reference.md` | Fully-qualified board name |
| `board.cpu_freq_mhz` | 80, 160, 240 | Lower = less power |
| `board.flash_size` | 2MB, 4MB, 8MB, 16MB | Must match hardware |
| `board.flash_mode` | `qio`, `qout`, `dio`, `dout` | `dio` is safest default |
| `board.flash_freq` | `40`, `80` | MHz |
| `board.partition_scheme` | See `references/partition-schemes.md` | **Ignored if `partitions.csv` is present in project root** |
| `upload.port` | `"auto"`, `"COM5"`, `"/dev/ttyUSB0"` | `auto` = Claude detects |
| `upload.upload_speed` | 115200, 230400, 460800, 921600 | Drop to 115200 if uploads are unreliable |
| `monitor.baudrate` | Usually 115200 | Must match `Serial.begin()` in code |
| `build.debug_level` | `none`, `error`, `warn`, `info`, `debug`, `verbose` | Core debug output |
| `build.extra_flags` | Array of `-D` defines | e.g. `["-DDEBUG_MODE=1"]` |

---

## Framework version enforcement — CRITICAL BEHAVIOR

Before every compile, Claude MUST verify the installed core version matches
`framework.core_version` from `project.json`.

### Never fabricate version numbers

Version numbers change over time. Claude must NEVER suggest, propose, or write a
core version from memory. Always obtain real version strings by running:

```bash
arduino-cli core update-index
arduino-cli core search esp32       # list all available versions
arduino-cli core list               # what's installed right now
```

The values like `<X.Y.Z>` shown throughout this document are placeholders only —
replace them with the actual output of these commands every time.

### Empty core_version field

If `framework.core_version` is empty or missing in an existing `project.json`:
1. This means the project has not been fully initialized
2. Run `arduino-cli core update-index` then `arduino-cli core search esp32`
3. Propose the latest stable version to the user
4. Once confirmed, write it to `project.json` and proceed

### Check sequence

```bash
arduino-cli core list
```

Parse the output, find the line for `esp32:esp32`, compare with `project.json`.

### Decision tree

**Installed version matches pinned version:**
- Proceed silently with compile

**Installed version is different:**
- STOP the compile
- Report clearly: `"Project pins esp32:esp32 v<pinned> but v<installed> is installed."`
- Offer three options:
  1. Install the pinned version: `arduino-cli core install esp32:esp32@<pinned>`
  2. Update `project.json` to match the installed version (if user confirms this is intentional)
  3. Cancel the compile

**Pinned version not installed at all:**
- Report: `"esp32:esp32 v<pinned> is not installed."`
- Propose: `arduino-cli core install esp32:esp32@<pinned>`
- Wait for confirmation before running

### NEVER auto-install or auto-update

Always require explicit user confirmation before changing the installed core version.
The pinned version is a contract — silently changing it would defeat the purpose.

---

## Compilation logic

### Step 1: Framework version check
See section above. Must pass before proceeding.

### Step 2: Check for custom partition file

**Before every compile**, check if `partitions.csv` exists in the project root:

```
project/
├── project-name.ino
├── project.json
└── partitions.csv          ← if present, overrides project.json partition_scheme
```

**If `partitions.csv` IS present:**
- Use it automatically via build property override
- Inform the user: "Using custom partitions.csv (overrides project.json partition_scheme)"

**If `partitions.csv` is NOT present:**
- Read `board.partition_scheme` from `project.json`
- **If the field is missing or empty**: ASK the user which scheme to use, then save the
  answer back into `project.json` so the question is asked only once per project

### Step 3: Build the compile command

**Standard compile (no custom partitions.csv):**

```bash
arduino-cli compile \
  --fqbn "esp32:esp32:esp32:PartitionScheme=default,CPUFreq=240,FlashSize=4MB,FlashMode=dio,FlashFreq=80,DebugLevel=none" \
  "<project_directory>"
```

**With custom partitions.csv:**

```bash
arduino-cli compile \
  --fqbn "esp32:esp32:esp32:CPUFreq=240,FlashSize=4MB,FlashMode=dio,FlashFreq=80,DebugLevel=none" \
  --build-property "build.custom_partitions=partitions" \
  --build-property "build.partitions=partitions" \
  "<project_directory>"
```

**With extra build flags:**

If `build.extra_flags` is non-empty, append:
```bash
  --build-property "compiler.cpp.extra_flags=-DDEBUG_MODE=1 -DFEATURE_X"
```

### Step 4: Report results

Always show:
- Sketch size and percentage of available program storage
- Global variables size and percentage of dynamic memory
- Warnings (if any)
- Path to generated `.bin` file

If compile fails, capture the error output and proceed to diagnostic mode (see
Troubleshooting section).

---

## Flash logic

### Port detection

If `upload.port` is `"auto"` or missing:

```bash
arduino-cli board list
```

Parse output. Scenarios:
- **One ESP32 detected** → use it silently, mention the port in confirmation
- **Multiple ESP32 detected** → ask user which one (offer tappable options)
- **No ESP32 detected** → report: "No ESP32 detected. Check USB cable and drivers (CP210x or CH340 typically required)."

### Upload command

```bash
arduino-cli upload \
  --fqbn "<fqbn>" \
  -p <port> \
  --input-dir <build_dir>
```

Add `--verify` if `upload.verify` is `true` in project.json.

### Upload failures

If upload fails with "Failed to connect to ESP32" or "Timed out waiting for packet header":

1. Tell user: **"Hold BOOT, tap EN (reset), release BOOT, then confirm to retry."**
2. Wait for confirmation before retrying
3. If still failing after retry, suggest lowering `upload.upload_speed` to `115200` in `project.json`

---

## Monitor logic

```bash
arduino-cli monitor -p <port> --config baudrate=<baudrate>
```

Notes:
- If a flash just happened, wait ~1 second before opening monitor (device needs to boot)
- Warn the user that monitor blocks the terminal until they stop it (Ctrl+C)
- If port from flash is known, reuse it without re-detecting

---

## Check framework updates (on-demand)

Trigger: user asks "check updates", "check framework updates", "any core updates",
or similar.

### Sequence

```bash
# 1. Refresh the package index
arduino-cli core update-index

# 2. Check what's outdated
arduino-cli core outdated

# 3. Show all available esp32:esp32 versions
arduino-cli core search esp32
```

### Report format

Present results clearly using the actual values returned by `arduino-cli`:

```
Currently installed: esp32:esp32 v<installed>
Latest available:    esp32:esp32 v<latest>

Recent versions available:
  <version>  (latest)
  <version>
  <version>
  <version>  ← currently installed (matches project.json pin)
  <version>
  ...
```

**Important:** never fabricate version numbers. Always run the commands and use
their real output — version numbers shown above are placeholders only.

Then offer next actions:
- **Install a newer version** (does not change existing projects, since they pin their own)
- **Update a specific project's pin** (requires editing `project.json` + testing)
- **Nothing, just informational**

NEVER auto-install or modify any project.json during a check.

---

## Creating a new project (interactive)

When the user asks to create a new ESP32 project or `project.json`, ask these
questions in order using tappable options when possible:

1. **Project name** (used for directory and `.ino` filename)
2. **Board model** — top picks: ESP32 Dev Module, ESP32-S3, ESP32-C3, ESP32-C6, NodeMCU-32S, ESP32-CAM, custom FQBN
3. **Flash size** — 4MB (default), 8MB, 16MB, 2MB
4. **Framework version** — determine the default by running `arduino-cli core update-index` then `arduino-cli core search esp32` to find the latest stable release of `esp32:esp32`. Propose it as default with the line "Dernière version stable: X.Y.Z — utiliser celle-là ?" and allow override. Once chosen, this value is frozen in `project.json` and never auto-changes.
5. **Partition scheme** — show options from `references/partition-schemes.md`, or "I'll add a custom partitions.csv later"
6. **Monitor baudrate** — default 115200, confirm
7. **Metadata** — description, client (optional but useful)

Generate:

```
new-project/
├── new-project.ino           ← skeleton with firmware version comment
├── project.json              ← populated from answers
├── .vscode/
│   └── tasks.json            ← copied from templates/.vscode/tasks.json
└── .gitignore                ← standard Arduino/build ignores
```

Do NOT create `partitions.csv` automatically — only mention that it can be added
later if the default scheme is insufficient.

---

## Troubleshooting (Claude parses errors and acts)

| Error substring | Likely cause | Action |
|---|---|---|
| `Failed to connect to ESP32` | Device not in download mode | Instruct BOOT+EN sequence, retry after user confirms |
| `Invalid head of packet` | Port busy or baudrate mismatch | Close monitors, lower upload_speed |
| `No board selected` | Core missing or wrong FQBN | Verify `arduino-cli core list`, check FQBN |
| `esp32:esp32 not installed` | Core not installed | Run `arduino-cli core install esp32:esp32@<version>` |
| `Library <X>.h: No such file` | Missing lib | Suggest `arduino-cli lib install "<name>"` — confirmation required |
| `region \`iram0_0_seg' overflowed` | Code too large for RAM | Review code size, reduce features |
| `region \`flash\`/\`app0\` overflowed` | Partition too small | Suggest `huge_app` scheme or custom partitions.csv |
| `Permission denied` on `/dev/ttyUSB*` | Linux dialout group missing | `sudo usermod -a -G dialout $USER` then re-login |
| `xtensa-esp32-elf-gcc: not found` | Toolchain corrupt | Reinstall core with `arduino-cli core install esp32:esp32@<version> --force` |

See `references/troubleshooting.md` for the detailed version.

---

## Global library management

Libraries are NOT declared in `project.json` (user preference). They are managed
globally via:

```bash
arduino-cli lib install "LibraryName"
arduino-cli lib list
arduino-cli lib update
```

When Claude sees a compile error about a missing library, it should propose the
install command but NEVER auto-run it without user confirmation.

---

## Prerequisite check (run once per session silently)

```bash
arduino-cli version        # must be >= 1.0
arduino-cli core list      # must include esp32:esp32
```

If `arduino-cli` is not installed at all, guide the user through installation
(see `references/installation.md`).

---

## File references

- `templates/project.json` — blank template with all fields
- `templates/partitions.csv.example` — example of a custom 4MB partition with OTA slots
- `templates/.vscode/tasks.json` — VS Code tasks that call arduino-cli directly (fallback)
- `templates/gitignore.txt` — standard .gitignore for ESP32 Arduino projects
- `references/fqbn-reference.md` — full list of ESP32 FQBN strings
- `references/partition-schemes.md` — all built-in partition schemes with sizes
- `references/troubleshooting.md` — detailed error diagnosis
- `references/installation.md` — arduino-cli installation per OS

---

## Style and tone

- If the conversation is in French, respond in French. If in English, respond in English
- Be concise and technical — avoid hand-holding
- Never auto-run destructive or state-changing commands without confirmation:
  - Compile: OK to auto-run once project.json is set up
  - Flash: OK to auto-run once project.json is set up
  - Monitor: OK to auto-run
  - `lib install`, `core install`, `core update`: **always require confirmation**
- When two configurations are plausible, ask rather than guess
- Never modify `project.json` without telling the user exactly what changed and why
