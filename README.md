# Retro Christmas Lights — vintage C9 bulb flicker for any FastLED strip

Personal project · 2025-12-15 · Solo: Max Dokukin · Status: Completed (archived)

![IMG_2009](https://github.com/user-attachments/assets/edbc5720-7bf1-480b-9ed8-3618fcad2b74)

## Overview

This Arduino sketch creates a realistic, vintage C9-style flicker effect on any addressable RGB LED strip or string
supported by the [FastLED](https://fastled.io/) library. It simulates classic incandescent C9 Christmas bulbs: every
pixel takes a colour from a five-colour C9 palette (red, orange, green, blue, warm white) and its brightness is driven by
its own channel of FastLED's `inoise8` Perlin noise, so each "bulb" flickers smoothly and independently. The example is
configured for 120 WS2812 / WS2812B (NeoPixel) LEDs on pin 3, but it relies only on the FastLED API, so it can be adapted
to any chipset FastLED supports (SK6812, APA102, WS2801, and more) and to any board that runs FastLED (Arduino Uno, Nano,
Mega, ESP32, ESP8266, Teensy, STM32, etc.).

## Highlights

- Realistic flicker using FastLED's `inoise8` (Perlin noise) for smooth brightness variation — no random jumps.
- Per-pixel noise offsets (`random16()` seed per LED) so every bulb flickers independently.
- Classic C9 colour palette: red `0xB80400`, orange `0x902C02`, green `0x046002`, blue `0x070758`, warm white `(86, 94, 22)`.
- Designed for 120 LEDs by default, configurable for any strip length with one `#define`.
- Platform-agnostic: one 47-line sketch with no board-specific code; runs on any board that can compile and run FastLED.

## How it works

```
setup: register WS2812 strip (GRB, TypicalLEDStrip correction) → seed one 16-bit noise offset per LED
loop (~60 FPS): for each LED i → colour = palette[i % 5] → brightness = inoise8(offset[i], z) → scale colour
               → FastLED.show() → z += FLICKR_SPEED → FastLED.delay(1000 / 60)
```

- **Palette** — `colors_map[5]` holds the five C9 colours; LEDs cycle through them in order (`i % 5`), like a
  traditional C9 string.
- **Noise field** — `inoise8(noiseOffset[i], gZ)` samples 2-D Perlin noise: the per-LED offset picks a unique track and
  the shared time axis `gZ` moves along it, returning 0–255.
- **Brightness scaling** — `nscale8_video(noise)` dims the base colour by the noise value (255 = full brightness, lower =
  dimmer).
- **Timing** — `gZ` advances by `FLICKR_SPEED` (3) per frame; smaller values give a slower, calmer flicker.

### Compatibility

**Supported LED types** — any addressable chipset supported by FastLED, including but not limited to:

- WS2812 / WS2812B (NeoPixel)
- SK6812
- APA102 (DotStar)
- WS2801
- Many other addressable RGB strips and pixel strings (and RGBW, where your FastLED version supports it)

To adapt the sketch to a different LED type, change the `FastLED.addLeds<…>` controller line to match your chipset and
colour order. As long as your LEDs are supported by FastLED, this effect will work.

**Supported boards / platforms** — any microcontroller that (1) is supported by the Arduino IDE or a compatible
environment, (2) can install and run the FastLED library, and (3) can drive your chosen addressable LEDs. Common boards:

- Arduino Uno, Nano, Pro Mini, Mega
- Arduino Leonardo, Micro
- ESP8266 (NodeMCU, Wemos D1 mini, etc.)
- ESP32 development boards
- Teensy boards
- Many other AVR and ARM-based boards supported by the Arduino ecosystem and FastLED

If your board can run FastLED examples, it can run this C9 flicker effect.

### Customization

- **Flicker speed** — change `FLICKR_SPEED` and/or the frame delay (`FastLED.delay(1000 / 60)`) to make the flicker
  slower or faster.
- **Colour palette** — edit the `colors_map` array to change or extend the colours (update the `5` in `i % 5` if you
  change its size).
- **Brightness** — `FastLED.setBrightness()` (255 by default) caps maximum brightness globally, which is useful for
  power-limited installations.

These changes do not affect cross-platform compatibility; the effect remains portable across all boards and LED types
supported by FastLED.

## Results

| Metric | Value | Baseline / note |
|---|---|---|
| Default strip length | 120 LEDs | `NUM_LEDS` |
| Palette | 5 C9 colours | `colors_map[5]` |
| Frame rate | ~60 FPS | `FastLED.delay(1000 / 60)` = 16 ms per frame |
| LED state in RAM | 600 bytes | 120 × 3-byte `CRGB` + 120 × 2-byte noise offsets |
| Code size | 47 lines, one file | `xewe-led-retro-christmas-lights.ino` |
| Arduino Uno build | 5,624 B flash (17%), 960 B RAM (46%) | `arduino-cli compile --fqbn arduino:avr:uno`, FastLED 3.10.5, AVR core 1.8.8 |
| ESP32 build | 393,779 B flash (30%), 27,916 B RAM (8%) | `arduino-cli compile --fqbn esp32:esp32:esp32`, FastLED 3.10.5, ESP32 core 3.3.11 |

The project is a lighting effect rather than a measured experiment, so the table lists its configuration and footprint;
the result is the look of the lights themselves (photo above). The two build rows were measured on 2026-09-27 while
documenting the project (compile only, not flashed to hardware).

## Getting started

### Hardware setup

1. **Connect your LED strip or string**
   - LED data input → microcontroller digital pin (default: pin 3).
   - LED 5 V → stable 5 V power supply (sized for your LED count).
   - LED GND → shared ground with the power supply ground and the microcontroller GND.
2. **Set the LED count** — set `NUM_LEDS` to the actual number of LEDs in your strip or string.
3. **Optional: change the data pin** — adjust `DATA_PIN` if you use a different digital pin.
4. **Recommended protection components**
   - 330–470 Ω resistor in series with the data line.
   - 1000 µF capacitor across 5 V and GND at the LED strip input.
   - These protect both the LEDs and the microcontroller and improve stability.

### Arduino IDE: quick setup and upload

1. **Install the Arduino IDE** — download the latest version for your operating system from the official Arduino
   website and launch it.
2. **Install the FastLED library** — **Sketch → Include Library → Manage Libraries…**, search for `FastLED`, and install
   `FastLED` by Daniel Garcia and Mark Kriegsman.
3. **Open or create the sketch** — clone this repository and open `xewe-led-retro-christmas-lights.ino` (the folder name
   already matches the sketch name), or create a new sketch (**File → New**), paste the code from this repository, and
   save it under a name such as `FastLED_Flickering_C9`.
4. **Configure** — `NUM_LEDS` (strip length), `DATA_PIN` (wiring), the FastLED LED type and colour order in
   `FastLED.addLeds<WS2812, DATA_PIN, GRB>` (to match your LEDs), and optionally `FastLED.setBrightness(...)`.
5. **Select board and port** — connect the board over USB; choose it under **Tools → Board** (e.g. "Arduino Uno",
   "ESP32 Dev Module") and its serial port under **Tools → Port**.
6. **Verify and upload** — click **Verify** (checkmark), then **Upload** (right arrow). After the upload finishes, the
   LEDs show the independent C9-style flicker.

```bash
# Alternative with arduino-cli, run inside the repository folder (example: Arduino Uno on /dev/ttyACM0;
# for an ESP32 use --fqbn esp32:esp32:esp32)
arduino-cli lib install FastLED
arduino-cli compile --fqbn arduino:avr:uno .
arduino-cli upload  --fqbn arduino:avr:uno -p /dev/ttyACM0 .
```

Requirements: Arduino IDE (or arduino-cli), the FastLED library, an addressable LED strip and a 5 V supply sized for it.
There are no tests; the check is watching the strip.

## Documents

- [Sketch source](xewe-led-retro-christmas-lights.ino)
- Project page: [maxdokukin.com/projects/xewe-led-retro-christmas-lights](https://maxdokukin.com/projects/xewe-led-retro-christmas-lights)
- Related: [XeWe LED OS](https://maxdokukin.com/projects/xewe-led-os) ([github.com/xewe-labs/xewe-led-os](https://github.com/xewe-labs/xewe-led-os)) —
  ESP32 firmware for addressable LEDs that includes a "Christmas Lights" mode built on the same palette-plus-Perlin-noise
  flicker.
- [FastLED library](https://fastled.io/)

### Keywords

This project is suitable if you are looking for:

- Arduino FastLED Christmas light effects
- Arduino WS2812B C9 flicker animation
- NeoPixel vintage C9 incandescent-style LEDs
- ESP32 / ESP8266 addressable LED Christmas lights
- FastLED Perlin noise flickering effect
- Cross-platform FastLED holiday lighting sketch
- Addressable LED C9 string light effect

Because the code relies strictly on the FastLED API and does not use any board-specific features, it is highly
compatible and portable. As long as your platform supports FastLED and your LEDs are in the list of supported chipsets,
this flickering C9 effect will work without major changes.
