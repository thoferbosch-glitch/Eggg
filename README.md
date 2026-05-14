# Egg Timer - Scientific Cooking Calculator

A single-file, self-contained web app that calculates precise egg boiling times using a physics-based heat transfer model. Adjusts for altitude, atmospheric pressure, water starting temperature, egg size, and fridge vs room-temperature eggs. Includes pixel art egg visualizations for each doneness level.

---

## How to Use

1. Open `index.html` in any modern browser (no server required).
2. Fill in your cooking conditions:
   - **Water start temp** — your tap water temperature (°C). Click "Auto" to use your GPS location and optionally pull live ambient temperature via OpenWeatherMap.
   - **Pressure** — atmospheric pressure in atm. Sea level is 1.00 atm. Useful if you have a barometer or are cooking in a pressure cooker (up to 1.5 atm).
   - **Altitude** — drag the slider to your elevation above sea level (0–5000 m).
   - **Egg size** — from Small (~42 g) to Jumbo (~85 g).
   - **Egg start temp** — fridge (4 °C) or room temperature (20 °C).
   - **Desired doneness** — Soft-boiled, Jammy, Medium, or Hard-boiled.
3. Click **Calculate Cooking Time**.
4. The result shows cook time from the moment the water reaches a rolling boil, plus a pixel art cross-section of your chosen doneness.
5. Use **Copy link** to share your exact settings via URL.

---

## The Physics Behind the Formula

The calculator uses a simplified heat transfer model based on protein denaturation kinetics.

### Boiling Point Correction

Water boils at lower temperatures at altitude because atmospheric pressure is reduced:

```
T_boil(altitude) = 100 - (altitude_m / 100) × 0.34   [°C]
```

An additional pressure correction applies if you specify a non-standard atmospheric pressure:

```
T_boil_effective = T_boil(altitude) + (pressure_atm - 1) × 28.5
```

### Base Cook Times

Starting values (at sea level, 20 °C water, large egg from fridge):

| Doneness   | Base time |
|------------|-----------|
| Soft       | 6 min 00 s |
| Jammy      | 7 min 00 s |
| Medium     | 8 min 30 s |
| Hard       | 11 min 00 s |

### Size Scaling

Eggs are approximated as spheres. Heat penetration time scales with the 2/3 power of mass (surface-area-to-volume ratio):

```
size_factor = (egg_mass_g / 63) ^ (2/3)
```

### Egg Starting Temperature

A cold egg from the fridge (4 °C) takes longer than one at room temperature. The correction adds approximately 4 seconds per °C below the reference fridge temperature:

```
time += (4 - egg_start_temp) × 4
```

### Boiling Point Factor

The driving force for heat transfer is proportional to how far above the protein denaturation threshold (~62 °C) the boiling water sits. At altitude, this driving force shrinks, requiring longer cooking:

```
bp_factor = (100 - 62) / (T_boil_effective - 62)
time *= bp_factor
```

### Water Starting Temperature

Cold tap water slightly extends the time until the egg is uniformly heated:

```
time += ((20 - water_temp) / 5) × 1   [seconds]
```

The final time is floored at 60 seconds to avoid physically implausible results.

---

## Customizing for Different Altitudes / Pressures

- **High altitude cities** (e.g. Denver ~1600 m, La Paz ~3600 m, Lhasa ~3650 m): drag the altitude slider to your elevation. The boiling point correction and bp_factor will automatically extend cook times significantly.
- **Pressure cookers**: set pressure above 1.0 atm (e.g. 1.2–1.5 atm for standard pressure cookers). This raises the effective boiling point and reduces cook times dramatically.
- **OpenWeatherMap API key**: to enable automatic water temperature auto-fill from live weather data, open `index.html`, find the line `const OWMKEY = '';` in the `<script>` block, and paste your free API key from [openweathermap.org](https://openweathermap.org/api) between the quotes. The free tier (1000 calls/day) is more than sufficient.

---

## Deployment Instructions (Netlify Drag-and-Drop)

This project is a single self-contained HTML file — no build step, no dependencies.

1. Go to [netlify.com](https://netlify.com) and sign in (or create a free account).
2. On the dashboard, scroll to the **"Want to deploy a new site without connecting to Git?"** section.
3. Drag the `egg-calculator` folder (or just `index.html`) onto the drop zone.
4. Netlify will deploy instantly and give you a URL like `https://quirky-name-12345.netlify.app`.
5. Optional: rename the site under Site Settings > General > Site name.

Alternatively, deploy via Netlify CLI:

```bash
npm install -g netlify-cli
netlify deploy --prod --dir=. --open
```

---

## Project Structure

```
egg-calculator/
  index.html    # Complete self-contained app (CSS + JS + SVG all inline)
  .gitignore    # Standard web project ignores
  README.md     # This file
```

---

## Formula Reference Card

| Parameter | Symbol | Effect |
|-----------|--------|--------|
| Altitude (m) | `h` | Lowers boiling point ~0.34 °C per 100 m |
| Pressure (atm) | `P` | Shifts boiling point ±28.5 °C per atm above/below 1.0 |
| Egg mass (g) | `m` | Scales time by `(m/63)^(2/3)` |
| Egg start temp (°C) | `T_egg` | +4 s per °C below 4 °C |
| Water start temp (°C) | `T_water` | +1 s per 5 °C below 20 °C |
| Protein denaturation temp | `T_d` | Fixed at 62 °C |
