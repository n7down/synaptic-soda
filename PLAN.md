# Synaptic Soda — Build Plan

A voice-activated soda machine where you speak any word, mood, or idea and Claude
maps it to a custom drink using 24 flavor syrups dispensed into pre-carbonated water.

---

## How It Works

```
[You speak]
     |
     v
[INMP441 I2S Mic on ESP32]
     |  (WiFi HTTP)
     v
[OpenAI Whisper API] --> transcribed text
     |
     v
[Claude API]
  - system prompt lists all 24 flavors
  - maps your words to flavor ratios
  - returns JSON: drink name + ratios
     |
     v
[ESP32 processes JSON]
     |
     v
[PCF8574 I2C expanders -> 3x 8-channel relays -> 24 peristaltic pumps]
[Dedicated relay -> 12V diaphragm pump for soda water]
     |
     v
[Syrups + carbonated water dispense into your cup]
     |
     v
[OLED display shows drink name + description]
```

---

## 24 Flavors

| # | Flavor        | Category       |
|---|---------------|----------------|
| 1 | Lemon-Lime    | Bright/Citrus  |
| 2 | Orange        | Bright/Citrus  |
| 3 | Grapefruit    | Bright/Citrus  |
| 4 | Yuzu          | Bright/Citrus  |
| 5 | Watermelon    | Sweet/Fruity   |
| 6 | Strawberry    | Sweet/Fruity   |
| 7 | Peach         | Sweet/Fruity   |
| 8 | Passionfruit  | Sweet/Fruity   |
| 9 | Mango         | Tropical       |
| 10 | Coconut      | Tropical       |
| 11 | Pineapple    | Tropical       |
| 12 | Guava        | Tropical       |
| 13 | Raspberry    | Berry/Tart     |
| 14 | Blueberry    | Berry/Tart     |
| 15 | Black Cherry | Berry/Tart     |
| 16 | Pomegranate  | Berry/Tart     |
| 17 | Vanilla      | Smooth/Creamy  |
| 18 | Caramel      | Smooth/Creamy  |
| 19 | Brown Sugar  | Smooth/Creamy  |
| 20 | Ginger       | Spicy/Herbal   |
| 21 | Mint         | Spicy/Herbal   |
| 22 | Lavender     | Spicy/Herbal   |
| 23 | Rose         | Floral         |
| 24 | Hibiscus     | Floral         |

All sourced as water-soluble syrups (Torani / Monin).

---

## Combinations

With 24 flavors and variable ratios (summing to 20 units across 3-6 active flavors):

| Active Flavors | Combinations  |
|----------------|---------------|
| 4 of 24        | ~25 million   |
| 5 of 24        | ~200 million  |
| 6 of 24        | ~1.1 billion  |

---

## Hardware

Prices sourced from Adafruit (adafruit.com) and AliExpress, September 2026.
Two budget scenarios: **budget** (AliExpress pumps, 3-4 week shipping) and
**fast** (Adafruit/Amazon, ships immediately).

| Component                                          | Qty | Unit Price        | Budget Total | Fast Total  | Source                    |
|----------------------------------------------------|-----|-------------------|--------------|-------------|---------------------------|
| ESP32-S3-DevKitC-1 (32MB Flash, 16MB PSRAM)       | 1   | $19.95            | $19.95       | $19.95      | Adafruit #5364            |
| SPH0645LM4H I2S MEMS Microphone Breakout           | 1   | $6.95             | $6.95        | $6.95       | Adafruit #3421            |
| Peristaltic pumps 12V food-grade (syrups + spares) | 27  | $5 / $24.95       | $135.00      | $673.65     | AliExpress / Adafruit #1150 |
| 12V diaphragm pump (soda water)                    | 1   | ~$12-15           | $15.00       | $15.00      | AliExpress                |
| 3.5" ILI9488 TFT touchscreen + XPT2046 touch (SPI)| 1   | ~$15              | $15.00       | $15.00      | AliExpress                |
| SunFounder 5V 8-channel relay board                | 3   | $9.89             | $29.67       | $29.67      | Amazon B00DR9SE4A         |
| MCP23017 I2C GPIO Expander (STEMMA QT)             | 2   | $5.95             | $11.90       | $11.90      | Adafruit #5346            |
| 12V 5A switching power supply                      | 1   | $24.95            | $24.95       | $24.95      | Adafruit #352             |
| 5V USB power supply (ESP32 + logic)                | 1   | ~$8               | $8.00        | $8.00       | AliExpress                |
| Food-grade silicone tubing (1m rolls)              | 3   | $3.50             | $10.50       | $10.50      | Adafruit #3659            |
| Enclosure (wood/acrylic/3D print)                  | 1   | $30-80            | $30.00       | $80.00      | Local/hardware store      |
| Misc (wiring, connectors, hot glue)                | —   | —                 | $20.00       | $20.00      | Hardware store            |
| **Electronics subtotal**                           |     |                   | **~$318**    | **~$906**   |                           |
| Syrups (Torani/Monin 750ml x24, ~$10 each)         | 24  | ~$10              | $240.00      | $240.00     | Grocery/online            |
| **Grand total**                                    |     |                   | **~$558**    | **~$1,146** |                           |

### Power supply note
The Adafruit 12V 5A supply ($24.95) is sufficient because pumps dispense
**sequentially** (water first, then syrups one at a time) — peak draw stays
at 1-2 pumps simultaneously (~1-2A). If you ever want parallel dispensing,
upgrade to a 12V 10A supply (~$20-25 on AliExpress).

### Pump sourcing notes
- Syrups: search `12V peristaltic pump food grade silicone tube` on AliExpress (~$3-5 each)
- Water: search `12V diaphragm pump food grade self-priming`
- Brands: INTLLAB, Gikfun, or generic dosing pump listings
- **Order 3-4 spare syrup pumps** — they break, go out of stock, and are hard to replace mid-build
- **Test pump direction before installing** — small peristaltic pumps are often assembled in random directions
  at the factory; test each one, mark direction, and flip the rotor mechanically if backwards (just open
  the head and reverse the roller insert — no tools needed)

### Syrup pump identification (from reference build video)
The pump used in the reference video has these identifying features:
- **Clear/transparent pump head** with visible rollers inside
- **DC motor sits directly on top** of the head (inline, not offset)
- **Very small** — roughly fingertip-sized, ~28-32mm head diameter
- **3-wire connector** (red, blue, yellow) on the motor
- Cost up to $25 each on Amazon; available cheaper on AliExpress

Most likely model: **INTLLAB DS-100** or a close clone
Search terms:
- `micro peristaltic pump clear head DC`
- `mini peristaltic dosing pump transparent head`
- `INTLLAB DS-100 peristaltic pump`
- `Kamoer micro peristaltic pump`

Check the reference video description for a direct part link:
https://www.youtube.com/watch?v=kBb56968ixI

### Syrup sourcing notes
- Buy **syrup** form only (not extract, oil, or essence)
- Water + sugar as first ingredients = water-soluble = safe
- Coconut, Lavender, Rose: especially important to buy syrup not extract
- **Syrups have different concentrations** — the calibration tool must include a dilution normalization
  step so that 1 unit of every flavor has roughly equal taste intensity in the final drink

---

## Wiring Overview

```
ESP32-S3 SPI bus
     |-- ILI9488 TFT display  (MOSI/CLK/CS/DC/RST)
     '-- XPT2046 touch controller (MOSI/MISO/CLK/CS/IRQ)
         [tap screen to start recording — replaces physical button]

ESP32-S3 I2C (SDA/SCL via STEMMA QT daisy chain)
     |-- MCP23017 #1 (0x20) -> 16 pins -> relay boards 1+2 -> pumps 1-16
     '-- MCP23017 #2 (0x21) ->  8 pins -> relay board 3   -> pumps 17-24
                                (8 pins spare for future expansion to 32 pumps)

ESP32-S3 I2S -> SPH0645LM4H microphone (SCK/WS/SD) - Adafruit #3421
ESP32-S3 GPIO -> Relay -> 12V diaphragm pump (soda water)

12V PSU -> all pump motors (via relay NO contacts)
5V PSU  -> ESP32-S3, relay logic, MCP23017s, touchscreen

CRITICAL: Keep 12V pump rail and 5V logic rail on completely separate supplies.
A damaged or overloaded power supply feeding both rails simultaneously is the
most likely way to fry the ESP32-S3. Never share grounds carelessly between rails.
```

---

## Software Stack

| Layer              | Tech                                          |
|--------------------|-----------------------------------------------|
| ESP32-S3 firmware  | Arduino / C++                                 |
| Speech-to-text     | OpenAI Whisper API (HTTP over WiFi)           |
| AI flavor mapping  | Claude API (claude-sonnet-4-6)                |
| Pump control       | PCF8574 I2C + timed relay pulses              |
| Touchscreen        | ILI9488 (TFT_eSPI library) + XPT2046 touch   |
| UI states          | Idle / Listening / Thinking / Dispensing / Done |

---

## Firmware File Structure (to be built)

```
firmware/
  synaptic_soda.ino   - main sketch, touch loop, drink cycle
  config.h            - WiFi, API keys, pin definitions, drink params
  mic.h / mic.cpp     - I2S mic init + WAV capture
  whisper.h/.cpp      - Whisper API: WAV -> transcript
  claude.h/.cpp       - Claude API: transcript -> DrinkRecipe JSON
  pumps.h/.cpp        - PCF8574 + relay control + dispense sequence
  display.h/.cpp      - ILI9488 touchscreen driver + UI states

prompt/
  system_prompt.txt   - Claude system prompt (reference copy)

tools/
  calibrate.py        - Python script to calibrate ml/sec per pump
```

---

## Claude Prompt Strategy

System prompt tells Claude:
- The 24 flavor names (exact strings matching firmware)
- Return ONLY valid JSON: `{ name, description, flavor_ratios }`
- flavor_ratios values must sum to exactly 20
- Use 3-6 active flavors
- Be creative and poetic

Example mappings:
- "rainy Sunday" -> lavender, blueberry, vanilla, ginger
- "tropical beach" -> coconut, pineapple, mango, lemon-lime
- "angry" -> pomegranate, ginger, black-cherry, grapefruit

---

## Build Phases

### Phase 1 — AI Pipeline (no hardware needed)
- ESP32 WiFi connection
- I2S mic capture (button -> record 4 sec WAV)
- HTTP POST to Whisper API -> transcribed text
- HTTP POST to Claude API -> DrinkRecipe JSON
- Print recipe to Serial monitor (validate before touching pumps)

### Phase 2 — Pump Control
- Wire 1 pump + 1 relay + 1 PCF8574, confirm it runs
- **Test every pump direction before installing** — flip rotor mechanically if backwards
- Calibrate timing (ml/sec) for syrup pumps
- **Apply hot glue strain relief** to all pump wire connectors to prevent broken leads
- Expand to all 24 syrup pumps
- Wire and test diaphragm water pump

### Phase 3 — OLED Display
- SSD1306 init
- "Listening..." while recording
- "Thinking..." while waiting on Claude
- Drink name + description while dispensing

### Phase 4 — Calibration Tool
- Python script runs each pump for a fixed time
- User measures output, enters volume
- **Dilution normalization**: each syrup has a different concentration — script calculates
  a scaling factor per pump so 1 unit = equal flavor intensity across all 24 syrups
- Script outputs calibration + normalization values to paste into config.h

### Phase 5 — Integration + Enclosure
- Full end-to-end test
- Error handling (API failure, WiFi drop, empty bottle)
- README with wiring diagram
- Physical enclosure: mount bottles, route tubing, single mixing nozzle

---

## Lessons from Similar Builds

From a reference build (https://www.youtube.com/watch?v=kBb56968ixI) that used the same
core concept with ~20 flavors + Raspberry Pi + Gemini AI:

| Problem encountered | Our mitigation |
|---|---|
| Pumps arrive wired in random direction | Test + flip all pumps before installing |
| Two flavors not water-soluble (lavender, capsaicin extract) | Already caught — using syrup form only |
| Syrups had wildly different concentrations | Dilution normalization in calibration tool |
| Custom PCB design took weeks to debug | Using ESP32 + breadboard + PCF8574 — no custom PCB |
| Fried 2x Raspberry Pi from power issues | ESP32 is $8 if fried; strict power rail separation |
| Broken pump wire leads | Hot glue strain relief on all connectors |
| Ran out of replacement pumps mid-build | Order 3-4 spares upfront |

Note: the video creator could not identify the specific pump model used — described only
as "teeny tiny peristaltic pumps" up to $25 each on Amazon. Search
`12V mini peristaltic pump` on AliExpress for equivalents at $3-5 each.

---

## Open Decisions (to resolve before Phase 1)

- [ ] Arduino IDE or PlatformIO?
- [ ] API keys in `config.h` or separate `secrets.h` (gitignored)?
- [ ] Push-to-talk button or auto voice-activity detection?
- [ ] Cup detection sensor before dispensing, or just trust the user?
