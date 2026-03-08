# ESP32-audioI2S — v2.0.6 patched for GCC 14 / no PSRAM

This is a fork of the excellent [ESP32-audioI2S library by schreibfaul1](https://github.com/schreibfaul1/ESP32-audioI2S), maintained here for ESP32 boards without PSRAM.

## Why this fork ?

Starting with v3.0.0, the original library requires PSRAM and allocates a 704 KB audio buffer at boot. On ESP32 dev boards without PSRAM, this causes an immediate out-of-memory crash. v2.0.6 is the last version that supports no-PSRAM boards, but it does not compile with the ESP32 board package v3.x, which ships with the xtensa-esp32-elf GCC 14 toolchain. GCC 14 enforces stricter type checking that exposes several incompatibilities in the original source.

The goal of this fork is to allow makers who own ESP32 boards without PSRAM to keep using them for simple audio projects — internet radio, audio notifications, basic streaming — without having to downgrade their Arduino IDE or their ESP32 board package. These boards are perfectly capable hardware for many practical use cases, and there is no good reason to abandon them just because the upstream library moved on.

This fork applies 4 targeted patches to make v2.0.6 compile and run correctly with ESP32 board package v3.x and GCC 14, with no functional changes to the library behaviour. Compatibility has been validated with board package v3.3.7 and Arduino IDE 2.3.8. We will do our best to maintain this compatibility as new versions of the board package are released, but we cannot guarantee it indefinitely — if a future board package version introduces breaking changes that cannot be easily patched, this will be documented here.

## Patches applied

Patch 1 — Audio.cpp : fixed a min() type mismatch between uint32_t and size_t that GCC 14 rejects as an ambiguous overload.

Patch 2 — aac_decoder.cpp : changed int val to int32_t val in 5 functions (UnpackQuads, UnpackPairsNoEsc, UnpackPairsEsc, DecodeOneScaleFactor, DecodeOneSymbol) to match the int32_t* pointer type expected by DecodeHuffmanScalar.

Patch 3 — aac_decoder.cpp : fixed a cast incompatibility — (uint32_t*)last replaced by (unsigned int*)last to resolve a C++ name mangling conflict under GCC 14.

Patch 4 — aac_decoder.cpp : corrected the DecodeHuffmanScalar function definition signature — uint32_t bitBuf replaced by unsigned int bitBuf to align declaration and definition and eliminate the linker error.

## Header rename

To allow coexistence with other versions of ESP32-audioI2S in the same Arduino libraries folder, Audio.h and Audio.cpp have been renamed to Audio_nopsram.h and Audio_nopsram.cpp. All internal #include references have been updated accordingly. In your sketch, use #include "Audio_nopsram.h" instead of #include "Audio.h". This makes it possible to install this fork alongside the original library or its v3.x versions without any conflict.

## Installation

Download or clone this repository and place the folder in your Arduino libraries directory. In Arduino IDE, open your sketch and replace #include "Audio.h" with #include "Audio_nopsram.h". Select your ESP32 dev board with PSRAM disabled and partition scheme set to Huge APP (3MB No OTA / 1MB SPIFFS). Set CPU frequency to 240 MHz for reliable AAC decoding.

## Tested environment

Arduino IDE : 2.3.8. ESP32 board package (Espressif) : 3.3.7. Compiler : xtensa-esp32-elf-g++ (esp-x32 toolchain, build 2511). esptool : 5.1.0. All system libraries (WiFi, SPI, NetworkClientSecure, FS, SPIFFS, FFat, SD, SD_MMC) are included in the board package v3.3.7 and require no separate installation.

## Arduino IDE board settings

Board : ESP32 Dev Module. PSRAM : Disabled. Partition Scheme : Huge APP (3MB No OTA / 1MB SPIFFS). CPU Frequency : 240 MHz.

## Validated examples

WiFi_Radio_ESP32dev_noPSRAM : a complete internet radio sketch for ESP32 dev board with MAX98357A I2S amplifier. Streams AAC and MP3 stations over WiFi. Includes a TCP probe mechanism to ensure the lwIP stack is fully ready before connecting to the stream, a WiFi watchdog for automatic reconnection, and a pre-fill buffer phase to prevent dropouts on startup. Tested and validated on ESP32 dev board without PSRAM, with ESP32 board package v3.3.7 and GCC 14.

Additional examples for no-PSRAM ESP32 boards will be added progressively as they are tested and validated on hardware. Only verified examples are published in this repository.

## Hardware tested

MCU : ESP32 dev board (no PSRAM), 240 MHz. Amplifier : MAX98357A I2S Class-D module.

## No-PSRAM behaviour

Unlike v3.x, ESP32-audioI2S v2.0.6 supports boards without PSRAM. When no PSRAM is detected it automatically falls back to a smaller internal SRAM buffer (approximately 6 KB input buffer) instead of the 704 KB PSRAM allocation that caused out-of-memory crashes on v3.x. To get reliable audio streaming with this small buffer, it is important to disable WiFi modem sleep (WiFi.setSleep(false)), set CPU frequency to 240 MHz for AAC decoding headroom, and allow a pre-fill phase after connecttohost() before the main loop takes over.

## Credits

Original library by [schreibfaul1](https://github.com/schreibfaul1/ESP32-audioI2S) — all audio decoding and streaming logic is his work. This fork only adds GCC 14 compatibility patches for no-PSRAM boards and contributes validated examples for this hardware configuration.
