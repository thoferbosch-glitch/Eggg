# Free APIs for the Egg Cooking Calculator

Ranked by ease of integration and practical usefulness. Last researched: May 2026.

---

## Summary Table

| API | Category | Auth | Free Limit | Ease | Usefulness |
|-----|----------|------|------------|------|------------|
| Open-Meteo | Weather + Pressure + Humidity | None | 10k/day | ★★★★★ | ★★★★★ |
| Browser Geolocation API | Location | None (browser) | Unlimited | ★★★★★ | ★★★★★ |
| Open Topo Data | Elevation | None | 1k/day | ★★★★★ | ★★★★★ |
| IP-API.com | IP Geolocation + Timezone | None | 45 req/min | ★★★★☆ | ★★★★☆ |
| BigDataCloud Reverse Geocode | Reverse Geocode + Locality | None (client-side) | Fair use | ★★★★☆ | ★★★☆☆ |
| ipapi.co | IP Geolocation + Timezone | None | 1k/day | ★★★★☆ | ★★★☆☆ |
| Open-Elevation | Elevation | None | ~1k/month | ★★★☆☆ | ★★★☆☆ |
| OpenWeatherMap | Weather + Humidity + Pressure | API key required | 1k/day | ★★★☆☆ | ★★★★☆ |
| WeatherAPI.com | Weather + Humidity + Pressure | API key required | ~3.3k/day | ★★★☆☆ | ★★★★☆ |
| Nominatim (OSM) | Reverse Geocode | None | ~1 req/sec | ★★★☆☆ | ★★☆☆☆ |

---

## Tier 1 — Recommended Stack (No Auth, High Limits)

### 1. Open-Meteo — Weather, Pressure & Humidity

**The single most valuable API for this project.** Provides temperature, barometric pressure, humidity, and altitude (as "surface elevation") — all from one call, with zero authentication.

- **Endpoint:** `https://api.open-meteo.com/v1/forecast`
- **Auth:** None (non-commercial use)
- **Free limits:** 10,000 calls/day · 300,000/month · 600 req/min burst
- **License:** CC BY 4.0 (attribution required)
- **Docs:** https://open-meteo.com/en/docs

**Sample request (all egg-relevant variables):**
```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=47.37
  &longitude=8.54
  &current=temperature_2m,relative_humidity_2m,surface_pressure,pressure_msl
  &timezone=auto
```

**Sample response:**
```json
{
  "latitude": 47.36,
  "longitude": 8.54,
  "elevation": 408.0,
  "timezone": "Europe/Zurich",
  "current": {
    "time": "2026-05-14T10:00",
    "temperature_2m": 18.3,
    "relative_humidity_2m": 62,
    "surface_pressure": 960.4,
    "pressure_msl": 1013.2
  }
}
```

**Variables of interest:**
| Variable | Description |
|---|---|
| `temperature_2m` | Air temperature at 2 m above ground (°C) |
| `relative_humidity_2m` | Relative humidity at 2 m (%) |
| `surface_pressure` | Atmospheric pressure at surface elevation (hPa) — best for altitude-adjusted boiling point |
| `pressure_msl` | Pressure normalized to mean sea level (hPa) |

**Pros:**
- Single call covers temperature + humidity + pressure + elevation
- No API key, no CORS issues when called from browser
- `surface_pressure` directly reflects altitude — ideal for boiling point calculation
- `elevation` field included in response = no separate elevation call needed
- Very high free limits; covers a public web app easily

**Cons:**
- Non-commercial only on free tier (paid plans start at ~$10/month for commercial)
- CC BY 4.0 requires attribution in the UI
- Response is forecast-based (nearest hour); not raw sensor data

---

### 2. Browser Geolocation API — Coordinates & Altitude

Built into every modern browser. No external call, no API key, no rate limit.

- **API:** `navigator.geolocation.getCurrentPosition()`
- **Auth:** User permission prompt only
- **Free limits:** Unlimited
- **Docs:** https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API

**Usage:**
```javascript
navigator.geolocation.getCurrentPosition(
  (pos) => {
    const { latitude, longitude, altitude, accuracy } = pos.coords;
    // altitude: meters above WGS84 ellipsoid, or null if unavailable
  },
  (err) => console.warn(err),
  { enableHighAccuracy: true, timeout: 8000 }
);
```

**Pros:**
- Zero network calls for lat/lon
- Returns `altitude` on GPS-enabled mobile devices (often null on desktop)
- `accuracy` field helps filter low-confidence fixes

**Cons:**
- Requires HTTPS
- User must grant permission (can be denied)
- `altitude` unreliable or null on desktops and many laptops — use Open Topo Data as fallback
- IP-based fallback (when GPS unavailable) can be inaccurate by tens of km

**Recommended pattern:** Use browser geolocation for lat/lon, then pass coordinates to Open Topo Data for a reliable altitude value.

---

### 3. Open Topo Data — Elevation

The most reliable free elevation API. No key required, 1000 calls/day.

- **Endpoint:** `https://api.opentopodata.org/v1/{dataset}?locations={lat},{lng}`
- **Auth:** None
- **Free limits:** 1,000 calls/day, max 1 call/second, max 100 locations/call
- **Recommended dataset:** `srtm90m` (global, 90 m resolution) or `aster30m` (30 m resolution)
- **Docs:** https://www.opentopodata.org/api/

**Sample request:**
```
GET https://api.opentopodata.org/v1/srtm90m?locations=47.37,8.54
```

**Sample response:**
```json
{
  "status": "OK",
  "results": [
    {
      "dataset": "srtm90m",
      "elevation": 408,
      "location": { "lat": 47.37, "lng": 8.54 }
    }
  ]
}
```

**Pros:**
- No auth, no setup
- SRTM90m covers latitudes −60° to +60° (most inhabited locations)
- `aster30m` provides finer 30 m resolution globally
- Batch up to 100 coordinates per request

**Cons:**
- 1,000 calls/day limit is sufficient for a typical public app but not a high-traffic one
- Data derived from SRTM/ASTER surveys — accuracy ±10–30 m, acceptable for boiling point
- If Open-Meteo `elevation` field is used, this call may be redundant

**Note:** Open-Elevation (open-elevation.com) is an alternative but its public instance is rate-limited to ~1,000/month on free plan — Open Topo Data is strictly better.

---

## Tier 2 — IP Geolocation & Timezone Fallback

When the user denies browser geolocation, IP-based location provides a rough fallback for pre-filling coordinates.

### 4. IP-API.com — IP Geolocation + Timezone (No Key)

- **Endpoint:** `http://ip-api.com/json/?fields=lat,lon,timezone,city,country`
- **Auth:** None
- **Free limits:** 45 requests/minute (per IP)
- **Note:** HTTP only on free tier (not HTTPS) — use a proxy or server-side call to avoid mixed-content warnings
- **Docs:** https://ip-api.com/docs/api:json

**Sample response:**
```json
{
  "lat": 47.3769,
  "lon": 8.5417,
  "timezone": "Europe/Zurich",
  "city": "Zurich",
  "country": "Switzerland"
}
```

**Pros:**
- No API key, extremely simple
- Returns `timezone` string (IANA format) — useful for `timezone=auto` in Open-Meteo
- 45 req/min is more than enough for a web app

**Cons:**
- Free tier is HTTP only (HTTPS requires paid plan) — must proxy from your server
- Accuracy: city-level (~10–50 km), not precise enough for altitude but fine for weather
- No pressure or altitude data

---

### 5. ipapi.co — IP Geolocation + Timezone (HTTPS)

- **Endpoint:** `https://ipapi.co/json/`
- **Auth:** None (up to limits)
- **Free limits:** 1,000 calls/day, 30,000/month
- **Docs:** https://ipapi.co/

**Sample response:**
```json
{
  "latitude": 47.3769,
  "longitude": 8.5417,
  "timezone": "Europe/Zurich",
  "city": "Zurich",
  "country_name": "Switzerland",
  "currency": "CHF"
}
```

**Pros:**
- HTTPS natively (no proxy needed)
- Clean field names, consistent JSON
- 1k/day is fine for most apps

**Cons:**
- Lower limit than IP-API.com at per-minute resolution
- Described as a "trial" tier; may require sign-up if limits change

**Recommendation:** Use ipapi.co as the default IP fallback (HTTPS, no proxy), fall back to IP-API.com if ipapi.co is unavailable.

---

### 6. BigDataCloud Reverse Geocode (Client-Side, No Key)

Converts lat/lon from `navigator.geolocation` into a human-readable city name. Useful for displaying "Zurich, Switzerland" in the UI.

- **Endpoint:** `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude={lat}&longitude={lng}&localityLanguage=en`
- **Auth:** None (client-side only, Fair Use Policy)
- **Free limits:** Unspecified; "Fair Use" — caller must be the actual client browser
- **Docs:** https://www.bigdatacloud.com/free-api/free-reverse-geocode-to-city-api

**Pros:**
- No key, direct browser call, no CORS issues
- Returns structured address (city, state, country, postcode)
- Falls back to IP geolocation if no coords provided

**Cons:**
- No timezone, no elevation
- Fair-use only — not suitable for server-side batch calls
- No guaranteed rate limit published

---

## Tier 3 — Auth-Required Weather APIs

These are viable alternatives if Open-Meteo becomes unavailable or you need commercial licensing, but require a free API key registration.

### 7. OpenWeatherMap

- **Endpoint:** `https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={key}&units=metric`
- **Auth:** API key (free registration)
- **Free limits:** 1,000 calls/day · 60 calls/minute
- **Docs:** https://openweathermap.org/api

**Returns:** temperature, humidity, pressure (sea-level and ground-level), wind, clouds

**Sample response (condensed):**
```json
{
  "main": {
    "temp": 18.3,
    "humidity": 62,
    "pressure": 1013,
    "grnd_level": 960
  }
}
```

**Pros:**
- Mature, widely used, excellent uptime SLA on paid tiers
- `grnd_level` pressure is the surface pressure (altitude-adjusted)
- HTTPS on free tier

**Cons:**
- Requires API key (registration, email confirmation)
- Harder to embed safely in a pure frontend app (key exposure)
- 1k/day is tighter than Open-Meteo's 10k/day

---

### 8. WeatherAPI.com

- **Endpoint:** `https://api.weatherapi.com/v1/current.json?key={key}&q={lat},{lon}&aqi=no`
- **Auth:** API key (free registration)
- **Free limits:** 100,000 calls/month (~3,333/day)
- **Docs:** https://www.weatherapi.com/docs/

**Returns:** temperature, humidity, pressure (mb), cloud cover, UV index

**Pros:**
- Generous monthly free limit (100k/month)
- Clean, consistent JSON structure
- HTTPS free tier

**Cons:**
- API key required (same exposure risk as OWM in pure frontend)
- No `surface_pressure` — only sea-level `pressure_mb`

---

## Tier 4 — Nominatim (Low Limit, Niche Use)

### 9. Nominatim (OpenStreetMap)

Only relevant if you need city/address lookup from coordinates and want to avoid BigDataCloud.

- **Endpoint:** `https://nominatim.openstreetmap.org/reverse?lat={lat}&lon={lon}&format=jsonv2`
- **Auth:** None (must include `User-Agent` header and optionally `email`)
- **Free limits:** Max 1 request/second; no bulk use; no commercial without self-hosting
- **Docs:** https://nominatim.org/

**Pros:**
- Open data (OSM), globally comprehensive address data
- No key

**Cons:**
- Hard 1 req/sec rate limit — risky for a web app with burst traffic
- OSM policy requires a valid `User-Agent` and prohibits systematic scraping
- No weather, no elevation, no timezone

---

## Recommended Integration Strategy

For the egg calculator, this minimal stack covers everything:

```
1. navigator.geolocation.getCurrentPosition()
       → lat, lon, (maybe altitude)

2. One Open-Meteo call (lat + lon from step 1)
       → temperature_2m, relative_humidity_2m,
          surface_pressure, pressure_msl, elevation
       → timezone (via timezone=auto)

3. Open Topo Data (optional fallback if Open-Meteo elevation seems wrong)
       → precise elevation in meters

4. ipapi.co (if user denies geolocation)
       → approximate lat/lon + timezone for pre-fill
```

This stack requires **zero API keys**, has a combined free limit well above 10,000 requests/day, and all calls are HTTPS-safe from the browser.

### Boiling Point Formula Reference

With `surface_pressure` (hPa) from Open-Meteo:

```
T_boil ≈ 100 - (1013.25 - P_surface) × 0.037   [°C]
```

Or from elevation (m) alone:
```
T_boil ≈ 100 - elevation × 0.003337              [°C]
```

Humidity affects evaporation but has a negligible effect on boiling point (<0.1°C at typical ranges) — it's most useful for user-facing "conditions" display.

---

## Sources

- [Open-Meteo Free Weather API](https://open-meteo.com/)
- [Open-Meteo Pricing & Limits](https://open-meteo.com/en/pricing)
- [Open-Meteo API Docs](https://open-meteo.com/en/docs)
- [Open Topo Data](https://www.opentopodata.org/)
- [Open Topo Data API Docs](https://www.opentopodata.org/api/)
- [Open-Elevation API](https://open-elevation.com/)
- [IP-API.com Docs](https://ip-api.com/docs/api:json)
- [ipapi.co Free Tier](https://ipapi.co/free/)
- [BigDataCloud Free Reverse Geocode API](https://www.bigdatacloud.com/free-api/free-reverse-geocode-to-city-api)
- [Nominatim Reverse Geocoding Docs](https://nominatim.org/release-docs/latest/api/Reverse/)
- [OpenWeatherMap Pricing](https://openweathermap.org/price)
- [WeatherAPI.com Pricing](https://www.weatherapi.com/pricing.aspx)
- [Top Weather APIs 2026 — WeatherAPI Blog](https://blog.weatherapi.com/top-5-weather-apis-developers-2026-complete-comparison/)
