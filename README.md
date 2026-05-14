# Egg Timer — Scientific Cooking Calculator

**Live demo:** https://egg-calculator.netlify.app/

A single-file, self-contained web app that calculates precise egg boiling times using a physics-based heat transfer model. Adjusts for altitude, atmospheric pressure, water starting temperature, egg size, and fridge vs room-temperature eggs.

Features a smooth doneness slider (0–100), animated pixel-art cross-section egg visualization with five states (Raw → Soft-boiled → Jammy → Hard-boiled → Overcooked), and one-click auto-detection of water temperature, elevation, and barometric pressure from free public APIs — no API key required.

---

## What's New (v2)

- **Doneness slider** — replaces the old radio-button pills. Drag 0–100; the label badge and egg animation update in real time. Calculation re-runs instantly when the result panel is open.
- **Animated pixel-art egg** — five higher-resolution states (50×56 viewBox, ~200 px display) cross-fade smoothly via CSS opacity transitions as you move the slider.
- **Auto-detect button** — one click uses the browser Geolocation API + Open-Meteo to populate altitude, ambient temperature, and barometric pressure. "Auto-detected" badges appear beside each filled field.
- **"Overcooked" state** — slider range now extends to fully overcooked with its own grey-green rubbery-yolk egg art.

---

## How to Use

1. Open `index.html` in any modern browser (no server or build step required).
2. Fill in your cooking conditions:
   - **Water start temp** — your tap water temperature (°C). Click **Auto** to auto-detect from Open-Meteo via GPS.
   - **Pressure** — atmospheric pressure in atm. Sea level is 1.00 atm. Auto-detected from Open-Meteo surface pressure.
   - **Altitude** — drag the slider to your elevation (0–5000 m). Auto-detected from Open-Meteo elevation data.
   - **Egg size** — from Small (~42 g) to Jumbo (~85 g).
   - **Egg start temp** — fridge (4 °C) or room temperature (20 °C).
   - **Desired doneness** — drag the slider from Raw (0) to Overcooked (100). Labels: Raw / Soft-boiled / Jammy / Hard-boiled / Overcooked.
3. Click **Calculate Cooking Time**.
4. The result shows cook time from rolling boil, a pixel-art cross-section matching your doneness, and key physics details.
5. Move the doneness slider while the result is showing — time updates live.
6. Click **Copy link** to share your exact settings via URL.

---

## Doneness Slider Ranges

| Slider range | State        | Algorithm key | Egg art                          |
|:------------:|:-------------|:-------------:|:---------------------------------|
| 0–19         | Raw          | soft (floor)  | Translucent bluish white, runny yolk |
| 20–39        | Soft-boiled  | soft          | Set white, bright liquid yolk    |
| 40–59        | Jammy        | jammy         | Golden-orange thick jammy yolk   |
| 60–79        | Hard-boiled  | hard          | Green sulphur ring, chalky yolk  |
| 80–100       | Overcooked   | hard          | Grey-green rubbery yolk, cracks  |

---

## Auto-Detection APIs

### Open-Meteo (primary — free, no API key required)

- **URL:** https://open-meteo.com
- **Endpoint used:** `https://api.open-meteo.com/v1/forecast?latitude=…&longitude=…&current=temperature_2m,surface_pressure&elevation=true&forecast_days=1`
- **Data provided:** current temperature (°C), surface pressure (hPa → converted to atm), and terrain elevation (m)
- **Rate limit:** unlimited for personal/non-commercial use
- **Privacy:** coordinates sent to Open-Meteo servers; no account or key needed

### OpenWeatherMap (optional fallback)

- **URL:** https://openweathermap.org/api
- **Free tier:** 1,000 API calls/day, current weather endpoint
- **Setup:** open `index.html`, find `const OWMKEY = '';` in the `<script>` block, paste your free key between the quotes
- **Used as fallback** when Open-Meteo is unreachable; provides temperature + pressure + city name

### Browser Geolocation API (built-in, free)

- Used to obtain GPS latitude/longitude (and sometimes altitude) to pass to the weather APIs
- Requires user permission prompt in the browser
- Altitude from GPS is used as a fallback if Open-Meteo elevation is unavailable

---

## The Physics Behind the Formula

The calculator uses a simplified heat transfer model based on protein denaturation kinetics.

### Boiling Point Correction

Water boils at lower temperatures at altitude:

```
T_boil(altitude) = 100 - (altitude_m / 100) × 0.34   [°C]
```

Pressure correction:

```
T_boil_effective = T_boil(altitude) + (pressure_atm - 1) × 28.5
```

### Base Cook Times

Starting values (sea level, 20 °C water, large egg, fridge):

| Doneness   | Base time   |
|:-----------|:-----------|
| Soft       | 6 min 00 s |
| Jammy      | 7 min 00 s |
| Medium     | 8 min 30 s |
| Hard       | 11 min 00 s |

### Size Scaling

```
size_factor = (egg_mass_g / 63) ^ (2/3)
```

### Egg Starting Temperature

```
time += (4 - egg_start_temp) × 4   [seconds]
```

### Boiling Point Factor

```
bp_factor = (100 - 62) / (T_boil_effective - 62)
time *= bp_factor
```

### Water Starting Temperature

```
time += ((20 - water_temp) / 5) × 1   [seconds]
```

Final time is floored at 60 seconds.

---

## Customising for Altitude / Pressure

- **High-altitude cities** (Denver ~1600 m, La Paz ~3600 m, Lhasa ~3650 m): click Auto or drag the altitude slider. Cook times extend significantly.
- **Pressure cookers**: set pressure above 1.0 atm (1.2–1.5 atm). This raises effective boiling point and reduces cook times.

---

## Deployment (Netlify)

This project is a single self-contained HTML file — no build step, no dependencies.

1. Go to [netlify.com](https://netlify.com) and sign in (or create a free account).
2. Drag the `egg-calculator` folder onto the Netlify dashboard drop zone.
3. Netlify deploys instantly. Rename under Site Settings > General > Site name.

Via CLI:

```bash
npm install -g netlify-cli
netlify deploy --prod --dir=. --open
```

Live site: **https://egg-calculator.netlify.app/**

---

## Project Structure

```
egg-calculator/
  index.html    # Complete self-contained app (CSS + JS + SVG inline)
  .gitignore    # Standard web project ignores
  README.md     # This file
```

---

## Formula Reference Card

| Parameter           | Symbol   | Effect                                          |
|:--------------------|:--------:|:------------------------------------------------|
| Altitude (m)        | `h`      | Lowers boiling point ~0.34 °C per 100 m        |
| Pressure (atm)      | `P`      | Shifts boiling point ±28.5 °C per atm delta    |
| Egg mass (g)        | `m`      | Scales time by `(m/63)^(2/3)`                  |
| Egg start temp (°C) | `T_egg`  | +4 s per °C below 4 °C                         |
| Water start temp    | `T_w`    | +1 s per 5 °C below 20 °C                      |
| Denaturation temp   | `T_d`    | Fixed at 62 °C                                  |

---

## Credits

- Physics model adapted from egg heat-transfer literature (Pöhlmann et al.)
- Pixel art egg SVGs — original work, hand-coded in SVG `<rect>` elements
- Weather/elevation data: [Open-Meteo](https://open-meteo.com) (CC BY 4.0)
- Optional weather data: [OpenWeatherMap](https://openweathermap.org) free tier
