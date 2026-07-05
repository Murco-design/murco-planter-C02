<p align="center">
 <img src="docs/image/hero.jpg">
</p>

# murCO Planter — a CO₂ monitor that lives in your living room

A local-only CO₂ / air quality monitor **and TTS notification speaker** for Home Assistant, hidden inside a geometric planter. No cloud. No subscription. No screen — just light and voice.

Built on ESPHome · Sensirion SCD40 (true NDIR) · M5Stack AtomS3 Lite (ESP32-S3) · **Firmware: MIT licensed**

> **The idea in one sentence:** your air quality device should tell you when to open a window — and look good enough to sit next to your books while doing it. Even kids get it: **red = open the window.**
>And even kids can build it: this design was tested by my sons (10 and 13) and our neighbor's family — every step they stumbled on, we made simpler.

---

## What's free in this repo

Everything you need to build a **fully working sensor**:

- ✅ `firmware/murco_v1.4.5.yaml` — complete, battle-tested ESPHome config (58 compilations later…), **MIT licensed**
- ✅ Bill of materials with sourcing notes (below)
- ✅ Wiring / pinout reference
- ✅ Quick-start flashing guide
- ✅ Home Assistant TTS automation example
- ✅ Lessons learned — the stuff that cost us evenings, free for you

**What's not here:** the 3D-printed geometric shell (STL), the illustrated 10-chapter build manual, and email support. Those live in the [murCO Starter Kit →](https://murco.design) — it's how a father-and-sons project funds itself. **The brain is free. The beauty is the product.**

<!-- [PHOTO — side by side: bare components vs. finished murCO] -->

---

## What it does

| Feature | How it works |
|---|---|
| 🟢🟠🔴 **CO₂ at a glance** | LED color by ppm (ASHRAE/REHVA thresholds). Standard 3-zone or sensitive 4-zone mode |
| 🗣 **It talks** | TTS speaker as a native HA `media_player` — announce CO₂ alerts, doorbells, anything |
| 🌡 **Climate report** | Long-press the button → temperature as color + optional voice report |
| 💧 **Plant watering tracker** | Blue pulse when your Tillandsia is thirsty (or disable it) |
| 🌙 **Day / night brightness** | Automatic, fully configurable hours and levels |
| 📡 **No display? No problem** | 4× click and murCO blinks out the last octet of its IP address in colors (green = hundreds, orange = tens, blue = units) |
| 📶 **Wi-Fi watchdog** | Magenta pulse when the connection drops — reconnects automatically |
| 📶 **BLE proxy** | Bonus: extends your HA Bluetooth range |
| 🏠 **100 % local** | ESPHome native API, encrypted, auto-discovered by HA. Web UI included — works without HA too |

---

## Bill of materials (~€35–40 total)

| Part | Note |
|---|---|
| M5Stack AtomS3 Lite | ESP32-S3 in a finished case — see below why |
| Sensirion SCD40 module | True NDIR CO₂ + temp + humidity. An SCD41 works too — same scd4x platform, no YAML changes needed. Planning to add the murCO shell later? Get the soldered 22×14 mm SCD40 breakout — the shell's internal mount is designed for that exact board size (larger boards like Adafruit/Qwiic modules won't fit). Verified sourcing links are included in the Starter Kit. |
| MAX98357A I2S amplifier | Optional — only if you want the TTS speaker |
| Small 4Ω speaker (~2–3 W) | Optional, pairs with the amp |
| Grove cable / dupont wires | I2C hookup |
| USB-C cable + 5V supply | Power |

All parts are commonly available (AliExpress, Mouser, local distributors).

### Why the AtomS3 Lite and not a bare ESP32 board?

Because murCO is built to be **a first project anyone can actually finish** — that's the whole philosophy, and the board choice follows it:

- **No soldering iron needed.** Grove connector for I2C, USB-C for power. If you can plug in cables, you can build this. (Our 10- and 13-year-old test builders confirm.)
- **The whole UX runs on stock hardware.** The button (1×/2×/4×/6× click functions) and the RGB LED indicator are already in the Atom — nothing to add, nothing to wire wrong.
- **Fixed form factor.** The murCO shell is engineered around the Atom's exact dimensions — airflow, LED diffusion, cable routing. Known hardware is what makes "works first time" a promise instead of a hope.

Hardcore DIY folks: the YAML ports to any ESP32-S3 with pin adjustments — everything is commented. Go wild.

### Wiring

| Signal | GPIO |
|---|---|
| I2C SDA (SCD40) | 1 |
| I2C SCL (SCD40) | 2 |
| I2S BCLK (audio) | 5 |
| I2S LRCLK (audio) | 6 |
| I2S DOUT (audio) | 7 |
| RGB LED (onboard) | 35 |
| Button (onboard) | 41 |

---

## Quick start

**Easy path — no ESPHome setup needed:**

1. Download `murco_v1.4.5_EN.factory.bin` from [Releases](../../releases)
2. Open [web.esphome.io](https://web.esphome.io) in Chrome/Edge, connect the Atom via a **data** USB cable
3. Click "Prepare for first use", then **Install** → select the downloaded .bin
4. Connect to the `murCO Planter Setup` Wi-Fi AP → `192.168.4.1` → enter your Wi-Fi
5. Home Assistant discovers it automatically — when asked for the encryption key, use:
   `lrzDKyTwxe+CjHKL8mYaP7FwV8zvfTGlpSesyjRAgKU=`
6. Done. Green light = breathe easy.

**Maker path — compile it yourself:**

1. Grab `firmware/murco_v1.4.5.yaml` and `firmware/partitions_murco_8mb.csv`
2. Put the CSV where the YAML's `board_build.partitions` path points — or adjust the path to your setup
3. Generate your own API encryption key (the YAML ships with a placeholder)
4. Compile in your ESPHome dashboard, flash, adopt in HA. Every section of the YAML is commented — bend it to your will.

Full web interface at `http://murcoplanter-xxxxx.local` — calibration, thresholds, brightness, all of it. Works standalone without HA.

---

## It talks 🗣

The speaker shows up in HA as a media player. From there, one small automation:

```yaml
alias: murCO CO2 Alert
triggers:
  - platform: numeric_state
    entity_id: sensor.murco_planter_XXXXXX_murco_co2
    above: 1500
    for:
      minutes: 5
actions:
  - action: tts.speak
    target:
      entity_id: tts.google_translate_en_com
    data:
      message: "CO2 is getting high. Please open a window."
      media_player_entity_id: media_player.murco_planter_XXXXXX_murco_speaker
mode: single
```

Doorbell announcements, laundry-done, "the plant is thirsty" — anything HA knows, murCO can say.

---

## Lessons learned (so you don't have to)

Things that cost us evenings across 58 firmware compilations:

- **The SCD40 self-heats inside an enclosure.** Sealed next to electronics, it reads 3–6 °C above ambient. The firmware ships with a configurable temperature offset (default 6.0 °C) — calibrate yours against a reference thermometer and adjust in 0.5 °C steps.
- **The temperature offset needs a restart to fully apply.** The SCD4x takes the new offset on the next boot — the firmware handles writing it, just restart after changing.
- **Never calibrate CO₂ indoors.** Forced recalibration sets the sensor to ~420 ppm reference — do it outside, after 10+ minutes of stabilization, or all your readings shift wrong.
- **LED logic + alerts + effects = recursion traps.** The LED state machine (CO₂ color, watering pulse, temperature blinks, critical alert loop) runs as a `mode: restart` script so overlapping triggers can't stack — if you extend it, keep it that way.
- **I2S audio on the AtomS3:** BCLK 5, LRCLK 6, DOUT 7 with an external DAC (`dac_type: external`, mono) — works reliably with the MAX98357A.

---

## The story

I wanted precise, local CO₂ data in Home Assistant — no cloud, no subscription. The prototype worked, but it was a mess of wires. My father, a mechanical engineer who spent his career designing industrial machinery, engineered the enclosure. My brother designed the polygonal shell. Then the build guide went through the toughest QA there is: my sons (10 and 13) and the neighbor's family built their own murCOs, and every step that confused them got rewritten. Months of walks and late evenings later, murCO is what it is: **three people, one device. Built by a family, for yours.**


---

## Make it beautiful

This repo gets you a working sensor on your desk. The [**murCO Starter Kit**](https://murco.design) gets you the thing on the hero photo:

- 🧊 All STL files — the geometric shell + internal mounts, designed around this exact hardware (airflow, LED diffusion, speaker cavity)
- 📖 Illustrated 10-chapter manual — refined by watching real families build it: my sons (10 and 13) and our neighbor's kids were the toughest QA team. From printed parts to a running device without googling anything.
- 🌿 Plant guide (yes, it's also a real planter — Tillandsia air plants, no soil)
- 💬 Email support from the person who wrote the firmware

**€23** at [murco.design](https://murco.design) · works out to ~€60 all-in including parts — compare that to $150+ consumer monitors that phone home.

---

## FAQ

**Can I use a different ESP32 board?** Yes, with pin adjustments — the YAML is commented. The AtomS3 Lite is just the neatest fit for the shell (see "Why the AtomS3" above).

**Does it work without Home Assistant?** Fully. Built-in web UI has every control. HA adds history graphs, automations and TTS.

**Manual CO₂ calibration?** Rarely needed (automatic self-calibration is on), but one button in the web UI — outdoors only.

**Why CO₂ only, no PM2.5?** Deliberate choice: for the daily "should I ventilate" question, CO₂ answers it, and it keeps the device small and affordable. PM is on the roadmap for v2.

**Fahrenheit?** Home Assistant converts automatically based on your unit settings. The device web UI and threshold sliders are °C for now.

**A Zigbee version?** Not planned for v1 — murCO is built around ESPHome's native API (and the TTS speaker needs the horsepower anyway). If you'd want one, open an issue and tell us — we're counting.

**License?** Firmware is **MIT** — use it, modify it, ship it, just keep the attribution. The murCO name and logo, the shell design (STL files) and the build manual are proprietary and **not** covered by the MIT license.

---

---

**If this saved you an evening, a ⭐ helps others find it.** Useful beyond that? You can [buy us a beer 🍺](https://buymeacoffee.com/murco.design) — or better, get the shell and give this sensor a home.

<p align="center">
  Made in Slovakia 🇸🇰 · <a href="https://murco.design">murco.design</a> · claim your spot on the <a href="https://murco.design/map">murCO world map</a>
</p>
