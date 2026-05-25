# iX Charging Intelligence

A single-file, zero-install dashboard for analyzing BMW iX charging history. Drop in the Excel file you export from the My BMW app, and get an interactive view of cost, performance, and behavior — entirely client-side, with your data never leaving the browser.

Built for the 2024 BMW iX but works with any BMW EV that exports charging data in the same format.

---

## Features

Four tabs, each focused on a different question about your charging:

**01 · Cost** — what you're actually paying
- Cost per kWh and cost per mile, trended over time
- Cumulative spend with a gasoline-equivalent comparison line (assumes a configurable gas price and MPG)
- Cost composition stacked by base charge, idle fees, sales tax, and discounts
- Cost by location, ranked

**02 · Performance** — how the car and chargers are doing
- Efficiency (miles per kWh) over time
- Average vs. max charging rate per session
- Start-of-charge and end-of-charge SOC distribution
- Range gained vs. energy delivered (a quick sanity check on the charger and the EPA estimate)

**03 · Behavior** — where, when, and how you charge
- Map of every public charging stop (geocoded automatically from the address — see [Geocoding](#geocoding))
- Calendar heatmap of daily kWh delivered
- Miles between charges
- Idle and grace-period minutes per session

**04 · Sessions** — the raw table
- Every session, sortable by date, location, cost, energy, rate, or duration
- Filter chips for All / DC fast charging / Home

Plus:
- Top-line KPIs that update with the active filter
- Light theme by default with a one-click dark toggle (preference persists)
- Drag-and-drop Excel upload with cached parse in `localStorage`

---

## Quick start

1. Download `index.html` (and optionally `bmw.png` for the favicon).
2. Open it in any modern browser. That's it.
3. Click the upload button in the header and select your charging export from the My BMW app.

The app ships with a couple of weeks of sample data so you can poke around immediately without uploading anything.

---

## Exporting your data from BMW

In the **My BMW** app, navigate to **Charging → History**, then export to Excel. The resulting file should have one row per session and these columns (the parser is tolerant of extra columns and whitespace):

| Column | Example | Notes |
|---|---|---|
| `Date` | `2026-05-14T16:34:15` | ISO timestamp or any format `Date()` understands |
| `Location` | `Walmart 2177 (San Diego, CA)` | Used for grouping; blank or matching `/home/i` → home |
| `Location Address` | `3382 Murphy Canyon Rd., San Diego, CA` | Used for map geocoding |
| `ChargerID` | `USEAME20014403` | Network is inferred (`USEAME*` → Electrify America) |
| `TotalCost`, `IdleCost`, `BaseChargingCost`, `Discount` | `$9.16` | `$` and `,` stripped automatically |
| `SalesTaxCost` | `$0.42` | |
| `TotalEnergyDelivered` | `19.1354 kWh` | Unit suffix optional |
| `MaxChargingRate` | `179.0 kW` | Used to classify DCFC (≥20 kW) |
| `TotalDuration`, `ChargeDuration`, `GracePeriodDuration`, `IdleDuration` | `11 min 37 sec` | Hours/min/sec format |
| `Mileage` | `23,807` | Thousands separator OK |
| `Start SOC`, `End SOC` | `0.71` or `71%` | |
| `Start Range`, `End Range` | `217` | In miles |

Any column missing or unparseable resolves to `null` and the dashboard adapts.

---

## Customization

All of these live near the top of the script block — search the file for the constant name.

**`HOME_COORDS`** — your home latitude/longitude. Used to plot home charging on the map and (later) for distance calculations:
```js
const HOME_COORDS = { lat: 32.717278, lng: -117.163176 };
```

**`STORAGE_KEY` / `GEO_CACHE_KEY` / `THEME_KEY`** — bump the version suffix if you want to invalidate cached data after a schema change.

**Theme** — light is the default. Toggle in the header; preference is remembered in `localStorage`.

---

## Geocoding

Public-charger sessions need lat/lng to appear on the Behavior map. Since BMW's export doesn't include coordinates, the app geocodes the `Location Address` column using [Nominatim (OpenStreetMap)](https://nominatim.org/) — free, no API key, no signup.

How it behaves:
- Runs in the **background** after the file loads, so the dashboard renders immediately.
- **Deduplicates** addresses before requesting — your usual Walmart is one lookup, not ten.
- **Caches** results permanently in `localStorage` under `ix-charging-geocache-v1`. First load of a new export takes about a second per unique address (Nominatim rate-limits to 1 request/second); subsequent loads are instant.
- Failed lookups are cached as `null` so the app doesn't keep retrying obviously-bad addresses. To force a re-lookup after fixing an address:
  ```js
  localStorage.removeItem('ix-charging-geocache-v1');
  ```

To swap providers (Google, Mapbox, HERE), replace the body of `geocodeAddressOnce()`. The rest of the geocoding pipeline is provider-agnostic.

---

## Privacy

Everything runs in your browser:

- **Charging data** never leaves your device. It's parsed in-memory and cached only in `localStorage`.
- **No analytics**, no trackers, no telemetry.
- **External requests** are limited to: Google Fonts (typography), Google Charts (visualization library), SheetJS CDN (XLSX parser), and Nominatim (only when geocoding a new address — never on every page load).

If you want a fully offline build, swap the three CDN-loaded scripts for local copies.

---

## Tech stack

- **Vanilla JS** — no framework, no build step, no dependencies to install
- **Google Charts** — line, area, bar, calendar heatmap, geo
- **SheetJS** (`xlsx.full.min.js`) — Excel parsing
- **Nominatim** — address → lat/lng
- **Fonts**: Fraunces (display) and IBM Plex Sans/Mono (text and numbers)

Everything is in a single `index.html` file. Total weight on disk is a few hundred KB, mostly seed data.

---

## Known limitations

- Designed and tested against the My BMW app's CSV/Excel export. Other automaker formats won't parse without changes to `parseRow()`.
- Nominatim's 1-request-per-second policy makes the first geocoding pass slow for large historical exports. The cache makes it a one-time cost.
- Charting library is Google Charts, which means an external script load and no native dark-mode for the charts themselves (the app theming handles that with CSS).
- No CI, no tests — this is a personal tool, not a production product.

---

## License

MIT — do whatever you'd like with it. Attribution appreciated but not required.

---

## Acknowledgments

- [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors for Nominatim geocoding
- BMW for shipping a vehicle that exports its own data in a halfway-reasonable format
