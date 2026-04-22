# Changelog

All notable changes to the `esp32-arduino-development` skill are documented in
this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Skill self-update mechanism with strict safety rules (silent session-start check,
  explicit update command, refuses pull on local changes, `git pull --ff-only`)
- `CHANGELOG.md` and `CONTRIBUTING.md`
- Explicit "Never fabricate version numbers" rule for core versioning
- Framework version detection at project creation (fetches latest stable from arduino-cli)
- On-demand `"check framework updates"` action
- `"custom"` as a valid value for `board.partition_scheme` — explicitly declares that
  `partitions.csv` is the authority for partition layout
- Auto-detection: when `partitions.csv` is present and `partition_scheme` ≠ `"custom"`,
  the skill prompts for confirmation before updating `project.json`
- `build.libraries_path` field — relative path to a local libraries folder, passed as
  `--libraries` to arduino-cli at compile time (cross-OS compatible)
- `build.output_dir` field — controls where the compiled `.bin` is written via
  `--output-dir`; use `"."` to output to the project root

### Changed
- Removed all hardcoded arduino-esp32 version numbers from templates and examples
- `project.json` template now has empty `framework.core_version` (filled at project creation)
- Compile command omits `PartitionScheme=` from FQBN when `partition_scheme` is
  `"custom"` — ensures arduino-cli reports sketch-size percentages against the actual
  partition layout, not a built-in scheme

### Fixed
- Misleading sketch-size percentages when `partitions.csv` was used alongside a
  generic `partition_scheme` value

## [1.0.0] - 2026-04-22

### Added
- Initial release
- Complete ESP32 firmware development workflow using Arduino CLI
- `project.json` schema (metadata, framework, board, upload, monitor, build sections)
- Core actions: compile, flash, monitor, create project, diagnose errors
- Automatic detection of custom `partitions.csv`
- Interactive project creation wizard
- VS Code tasks.json template as Claude-independent fallback
- Reference documentation: FQBN reference, partition schemes, troubleshooting, installation
- Example `partitions.csv` with OTA support
- Standard `.gitignore` for ESP32 Arduino projects
- MIT License

[Unreleased]: https://github.com/Erablitek/esp32-arduino-development/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Erablitek/esp32-arduino-development/releases/tag/v1.0.0
