# ESP32 FQBN Reference

FQBN = Fully Qualified Board Name. Used in `project.json` as `board.fqbn`.

All ESP32 boards use the `esp32:esp32:` prefix from the arduino-esp32 core.

## Primary chip families

| Chip     | FQBN                           | Architecture | Notes                              |
|----------|--------------------------------|--------------|------------------------------------|
| ESP32    | `esp32:esp32:esp32`            | Xtensa LX6   | Original, dual-core, WiFi+BT       |
| ESP32-S2 | `esp32:esp32:esp32s2`          | Xtensa LX7   | Single-core, WiFi only, USB OTG    |
| ESP32-S3 | `esp32:esp32:esp32s3`          | Xtensa LX7   | Dual-core, WiFi+BLE, USB OTG, AI   |
| ESP32-C3 | `esp32:esp32:esp32c3`          | RISC-V       | Single-core, WiFi+BLE 5, low-power |
| ESP32-C6 | `esp32:esp32:esp32c6`          | RISC-V       | WiFi 6, BLE 5, Thread, Zigbee      |
| ESP32-H2 | `esp32:esp32:esp32h2`          | RISC-V       | BLE 5, Thread, Zigbee (no WiFi)    |
| ESP32-P4 | `esp32:esp32:esp32p4`          | RISC-V       | High-performance, no wireless      |

## Common dev board variants

| Board                          | FQBN                              |
|--------------------------------|-----------------------------------|
| ESP32 Dev Module               | `esp32:esp32:esp32`               |
| NodeMCU-32S                    | `esp32:esp32:nodemcu-32s`         |
| ESP32-WROOM-DA Module          | `esp32:esp32:esp32da`             |
| ESP32 PICO-D4                  | `esp32:esp32:pico32`              |
| ESP32-CAM                      | `esp32:esp32:esp32cam`            |
| ESP32-S3 Dev Module            | `esp32:esp32:esp32s3`             |
| ESP32-S3-BOX                   | `esp32:esp32:esp32s3box`          |
| ESP32-S2 Dev Module            | `esp32:esp32:esp32s2`             |
| ESP32-C3 Dev Module            | `esp32:esp32:esp32c3`             |
| ESP32-C6 Dev Module            | `esp32:esp32:esp32c6`             |
| LOLIN D32                      | `esp32:esp32:d32`                 |
| LOLIN S2 Mini                  | `esp32:esp32:lolin_s2_mini`       |
| TTGO T-Display                 | `esp32:esp32:ttgo-t1`             |
| Heltec WiFi Kit 32             | `esp32:esp32:heltec_wifi_kit_32`  |
| M5Stack Core                   | `esp32:esp32:m5stack_core_esp32`  |
| M5Stack ATOM                   | `esp32:esp32:m5stack_atom`        |
| Adafruit Feather ESP32         | `esp32:esp32:featheresp32`        |
| Adafruit Feather ESP32-S3      | `esp32:esp32:adafruit_feather_esp32s3` |
| SparkFun ESP32 Thing           | `esp32:esp32:esp32thing`          |

## How to list all available boards

```bash
arduino-cli board listall esp32
```

## How to discover FQBN of a connected board

```bash
arduino-cli board list
```

Output includes the FQBN in the `FQBN` column for auto-identified boards.

## FQBN with options

Options are appended to the base FQBN with `:key=value,key=value`:

```
esp32:esp32:esp32:PartitionScheme=huge_app,CPUFreq=240,FlashSize=4MB
```

To see all options for a specific board:

```bash
arduino-cli board details --fqbn esp32:esp32:esp32
```
