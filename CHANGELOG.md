# Changelog

All notable changes to the `esp32-arduino-development` skill are documented in
this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.0] - 2026-04-22

### Added
- Skill self-update mechanism with strict safety rules
  - Silent session-start check via `git fetch` (hybrid mode)
  - Short notification shown at top of first response when updates are available
  - `"update the skill"` / `"pull skill updates"` commands trigger explicit update
  - Refuses to pull if there are any uncommitted local changes
  - Uses `git pull --ff-only` to avoid merge commit pollution
  - Never runs destructive Git commands automatically
- `CHANGELOG.md` in Keep a Changelog format
- Semantic versioning rules and contribution guidelines (see CONTRIBUTING.md)

## [1.1.0] - 2026-04-22

### Added
- Explicit "Never fabricate version numbers" rule for core versioning
- Framework version detection at project creation (fetches latest stable from arduino-cli)
- On-demand `"check framework updates"` action to scan for arduino-esp32 core updates

### Changed
- Removed all hardcoded arduino-esp32 version numbers from templates and examples
- Replaced example versions with placeholders (`<X.Y.Z>`, `<pinned>`, `<installed>`, `<latest>`)
- `project.json` template now has empty `framework.core_version` field (to be filled at project creation)

### Fixed
- Documentation would have become outdated with time due to baked-in version numbers

## [1.0.0] - 2026-04-22

### Added
- Initial release
- Complete ESP32 firmware development workflow using Arduino CLI
- `project.json` schema (metadata, framework, board, upload, monitor, build sections)
- Core actions: compile, flash, monitor, create project, diagnose errors
- Automatic detection of custom `partitions.csv` (overrides project.json scheme)
- Interactive project creation wizard
- VS Code tasks.json template as Claude-independent fallback
- Reference documentation:
  - FQBN reference for all ESP32 variants (ESP32, S2, S3, C3, C6, H2, P4)
  - Built-in partition schemes with sizes
  - Troubleshooting guide for common errors (upload, compile, runtime)
  - arduino-cli installation per OS (Windows, macOS, Linux)
- Example `partitions.csv` with OTA support
- Standard `.gitignore` for ESP32 Arduino projects
- MIT License

[Unreleased]: https://github.com/Erablitek/esp32-arduino-development/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/Erablitek/esp32-arduino-development/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/Erablitek/esp32-arduino-development/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/Erablitek/esp32-arduino-development/releases/tag/v1.0.0
