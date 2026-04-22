# ESP32 Arduino Development Skill

A Claude skill for ESP32 firmware development using Arduino CLI, replacing the Arduino IDE entirely.

## What this skill does

When installed, Claude can:

- **Compile** ESP32 projects using settings from a `project.json` file
- **Flash** firmware to connected ESP32 boards
- **Monitor** serial output
- **Create** new projects interactively
- **Check** for arduino-esp32 core updates on demand
- **Diagnose** compile and upload errors automatically

All driven through `arduino-cli` directly — no custom build scripts required.

## Project structure

Each ESP32 project uses this layout:

```
my-project/
├── my-project.ino          # Main firmware file
├── project.json            # Board, port, framework version, build options
├── partitions.csv          # OPTIONAL — custom partition table
├── src/                    # OPTIONAL — additional .cpp/.h source files
└── .vscode/
    └── tasks.json          # OPTIONAL — standalone VS Code tasks (Claude-independent)
```

The `project.json` is the **single source of truth** for build settings. Libraries
are managed globally via `arduino-cli lib install`, not declared per project.

## Installation

1. Copy the `esp32-arduino-development/` directory into your Claude skills folder
2. Ensure `arduino-cli` is installed (see `references/installation.md`)
3. Start a conversation mentioning ESP32 — Claude will detect the skill

## Usage examples

Just talk to Claude naturally:

- *"Crée un nouveau projet ESP32 pour un capteur de vacuum"*
- *"Compile ce projet"*
- *"Flash sur COM5"*
- *"Ouvre le moniteur série"*
- *"Check for framework updates"*
- *"L'upload fait un timeout, aide-moi"*

Claude reads `project.json`, respects the pinned framework version, checks for
a custom `partitions.csv`, and handles everything.

## Framework version pinning

When you create a new project, the skill fetches the **latest stable** version of
`arduino-esp32` from the Arduino package index and pins it into `project.json`.
From that point on, the version is frozen — before every compile, Claude verifies
the installed core matches the pin. Mismatches halt the build with a clear prompt
to install the correct version or explicitly update the pin.

This ensures **firmware built in 2026 can be rebuilt identically in 2030**, while
new projects always start fresh on the latest stable release.

## Custom partitions

If a `partitions.csv` file exists in the project root, Claude automatically uses
it instead of the built-in partition scheme from `project.json`. See
`templates/partitions.csv.example` for a 4MB example with OTA slots.

## Supported boards

All ESP32 variants: classic ESP32, S2, S3, C3, C6, H2, P4, and their dev board
derivatives. See `references/fqbn-reference.md` for the full list.

## Files in this skill

| File | Purpose |
|---|---|
| `SKILL.md` | Main skill definition — Claude reads this |
| `CHANGELOG.md` | Version history (Keep a Changelog format) |
| `CONTRIBUTING.md` | Versioning rules (SemVer) and release process |
| `LICENSE` | MIT license |
| `templates/project.json` | Blank config template for new projects |
| `templates/partitions.csv.example` | Example custom partition table |
| `templates/.vscode/tasks.json` | Standalone VS Code build tasks |
| `templates/gitignore.txt` | Standard .gitignore for ESP32 projects |
| `references/fqbn-reference.md` | FQBN strings for all ESP32 boards |
| `references/partition-schemes.md` | Built-in partition schemes documented |
| `references/troubleshooting.md` | Detailed error diagnosis |
| `references/installation.md` | arduino-cli install per OS |

## Credits

- Based in part on the workflow originally published by EricSun787
  (github.com/EricSun787/esp32-arduino-development)
- Rewritten, translated to English, and restructured as a Claude skill

## License

MIT — use freely.
