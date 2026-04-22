# ESP32 Partition Schemes

Built-in partition schemes available via `board.partition_scheme` in `project.json`.
These are the schemes bundled with arduino-esp32 core.

## Schemes for 4MB flash (most common)

| Scheme name         | App size   | SPIFFS/FS | OTA | Use case                              |
|---------------------|-----------|-----------|-----|---------------------------------------|
| `default`           | 1.2 MB    | 1.5 MB    | Yes | General-purpose (default)             |
| `default_ffat`      | 1.2 MB    | 1.5 MB (FAT) | Yes | Use FAT filesystem instead of SPIFFS |
| `minimal`           | 1.9 MB    | 190 KB    | No  | No OTA, minimal filesystem            |
| `no_ota`            | 2 MB      | 1.9 MB    | No  | Bigger app, no OTA                    |
| `huge_app`          | 3 MB      | 900 KB    | No  | Large firmware, no OTA                |
| `min_spiffs`        | 1.9 MB    | 190 KB    | Yes | OTA + tiny SPIFFS                     |
| `max_app_4MB`       | ~2 MB     | 0         | Yes | Max app size with OTA                 |

## Schemes for 8MB flash

| Scheme name              | App size | SPIFFS/FS  | OTA |
|--------------------------|----------|-----------|-----|
| `default_8MB`            | 3.3 MB   | 1.4 MB    | Yes |
| `large_spiffs_8MB`       | 1.9 MB   | 4 MB      | Yes |

## Schemes for 16MB flash

| Scheme name              | App size | SPIFFS/FS  | OTA |
|--------------------------|----------|-----------|-----|
| `default_16MB`           | 6.5 MB   | 2.8 MB    | Yes |
| `large_spiffs_16MB`      | 1.9 MB   | 12 MB     | Yes |
| `app3M_fat9M_16MB`       | 3 MB     | 9 MB (FAT)| Yes |

## ESP32-S3 / C6 specific

| Scheme name              | Flash size | App size  | PSRAM support |
|--------------------------|-----------|----------|---------------|
| `default_8MB`            | 8 MB      | 3.3 MB   | Compatible    |
| `rainmaker`              | 4 MB      | 1.9 MB   | Yes (ESP RainMaker) |

## How to list all schemes for a board

```bash
arduino-cli board details --fqbn esp32:esp32:esp32 --show-properties
```

Then grep for `menu.PartitionScheme`.

Or browse the source:
- Linux/macOS: `~/.arduino15/packages/esp32/hardware/esp32/<version>/tools/partitions/`
- Windows: `%LOCALAPPDATA%\Arduino15\packages\esp32\hardware\esp32\<version>\tools\partitions\`

## When to use a custom partitions.csv instead

Use a custom `partitions.csv` in the project root when:
- You need a specific layout not covered above
- You want custom NVS partitions (e.g. separate encrypted keys partition)
- You need multiple SPIFFS/FAT partitions
- You're doing secure boot with custom signed partitions
- You need specific partition offsets for factory programming

When `partitions.csv` is present, the skill ignores `board.partition_scheme`.

## Partition overflow errors

If you see:
```
region `app0' overflowed by N bytes
```

You've got three options:
1. Reduce firmware size (remove features, optimize)
2. Switch to a scheme with a bigger app partition (e.g. `huge_app`, `no_ota`)
3. Create a custom `partitions.csv` with a larger app slot
