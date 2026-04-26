# EthioRoute

> Landmark-first navigation & real-time transit for Addis Ababa.

---

## 🚀 Quick Start

No build step, no bundler. Requires a local HTTP server (Service Worker and Fetch API do not work over `file://`):

```powershell
npx serve .
```

Then open `http://localhost:3000`.

**One-time setup checklist:**
1. Add your **Supabase credentials** to `config.js` (URL + anon key)
2. Create the required Supabase tables — SQL in `NEEDS_MORE_INFO.md` Step 2
3. Download the **Noto Sans Ethiopic** fonts into `fonts/` — instructions in `NEEDS_MORE_INFO.md` Step 1

---

## 🗂 Project Structure

```
EthioRoute/
├── index.html                  # App shell — open this
├── manifest.json               # PWA manifest (installable)
├── service-worker.js           # Offline tile + asset caching
├── config.js                   # Supabase URL & anon key ← YOUR CONFIG
│
├── css/
│   ├── main.css                # Global design system, tokens, splash, FABs
│   ├── map.css                 # Map container, locate button, popups, user marker
│   └── panels.css              # Left drawer, route steps, tariff table, toggles
│
├── js/
│   ├── app.js                  # Bootstrapper & all UI event wiring
│   ├── map.js                  # Leaflet 1.9.4 initialisation & tile switching
│   ├── i18n.js                 # Amharic ↔ English string table & toggle
│   ├── gtfs-data.js            # Route registry, Haversine, nearest-stop finder
│   ├── transfer-graph.js       # Multi-leg transfer adjacency graph (BFS)
│   ├── landmarks.js            # Weighted POI search, favourites, suggestions
│   ├── search.js               # Search UI, per-input autocomplete, suggest modal
│   ├── transit.js              # LRT / Anbessa / Minibus / Sheger layer renderer
│   ├── heatmap.js              # Supabase real-time demand heatmap
│   ├── crowdsource.js          # Anonymous crowdsourced bus GPS
│   ├── routing.js              # Multi-modal route engine + OSRM walk directions
│   ├── commute-tracker.js      # Arrival-time tracker for active journeys
│   └── lowbandwidth.js         # Low-data mode toggle
│
├── data/
│   ├── landmarks.json          # 30+ real Addis Ababa POIs
│   ├── lrt-routes.json         # LRT Line 1 & 2 GeoJSON polylines (~18 MB)
│   ├── lrt-stations.json       # LRT station Point features with metadata
│   ├── anbessa-routes.json     # Anbessa Bus GeoJSON corridors (~7.4 MB)
│   ├── minibus-routes.json     # 6 blue/white minibus corridors
│   ├── sheger-routes.json      # 3 Sheger Bus corridors (~3.2 MB)
│   ├── tariffs.json            # Fare comparison data by transit type
│   └── terminals.json          # Transfer hubs (Megenagna, Mexico, etc.)
│
├── fonts/                      # NotoSansEthiopic-Regular/Bold.woff2 & .ttf
├── icons/                      # SVG icons: logo, landmark categories, live bus
└── scripts/
    └── sync-gtfs.js            # ETag-aware weekly GTFS data refresh
```

---

## 🌍 Transit Data

**LRT — Addis Ababa Light Rail Transit (ERC)**
- **Line 1 (East–West):** Ayat → Megenagna → Stadium → Meskel Square → Mexico → Menelik II → Tor Hailoch
- **Line 2 (North–South):** Menelik II → Mexico → Meskel Square → Gofa → Akaki → Kaliti

**Minibus Corridors (6)**
- Bole ↔ Megenagna
- Merkato ↔ Piassa
- CMC ↔ Megenagna
- Saris ↔ Mexico Square
- Piassa ↔ Megenagna
- Meskel Square ↔ Asko

**Anbessa City Bus (4 corridors)**
- Route 47: Piassa ↔ Bole
- Route 3: Merkato ↔ Sidist Kilo
- Route 12: Saris ↔ Megenagna
- Route 28: CMC ↔ Merkato

**Sheger Bus (3 corridors)**
- Sheger Bus 01, 02, 03

---

## ✨ Features

### Navigation & Search
| Feature | Detail |
|---|---|
| **Landmark-First Search** | Exact match → alias → partial, with proximity boost when GPS is available. Top 7 results. |
| **Amharic / English** | Full Ge'ez script support. All landmarks have `name` + `nameAm`. Toggle in the header. |
| **Route Planner** | Type or select From/To — autocomplete appears inline next to each input. |
| **Map Long-Press** | Hold on any map spot → "Set as From / Set as To" menu pins that coordinate. |
| **Saved Places** | ♡ Save button in any popup → stored in `localStorage`, accessible across sessions. |
| **Suggest a Landmark** | Drop a pin on the map + fill in name/category → saved locally and pushed to Supabase. |
| **URL Route Sharing** | Every planned route encodes `#route/lat,lng/lat,lng` in the URL hash for deep-linking. |

### Routing Engine
| Feature | Detail |
|---|---|
| **Walk-Only** | Trips under 1.5 km go straight to OSRM foot routing. |
| **Single-Route Transit** | Finds the best minibus or Anbessa route where both origin and destination lie on the same polyline. Checks both forward and reversed direction. |
| **LRT Routing** | Used when both points are within 500 m of different stations. Draws the actual rail polyline from `lrt-routes.json`. |
| **Multi-Leg (Transfer)** | BFS over `terminals.json` finds 2-leg journeys: e.g. Minibus → walk at Megenagna → LRT. |
| **Turn-by-Turn Walking** | Walk legs hit OSRM's public foot router. Collapsible step list with distance and direction per turn. |
| **Fare Comparison** | After routing, a collapsible table shows fares for all transit types on that corridor from `tariffs.json`. |
| **Transit Toggles** | LRT / Anbessa / Minibus toggles unlock after a route is planned. Turning one off re-runs the engine without that mode. |

### Real-Time & Crowdsourcing
| Feature | Detail |
|---|---|
| **Waiting for Taxi** | "Waiting for Taxi" FAB → sends GPS + route ID to `commuter_waiting` in Supabase. |
| **Going Back** | "↩ Going Back" FAB sends an `outbound` direction signal — drivers see return demand separately. |
| **Driver Heatmap** | Driver mode → pick a route → live demand heatmap fetched from Supabase with real-time updates. |
| **Return Demand Toggle** | Drivers can toggle to see outbound (return) commuter locations to avoid empty trips. |
| **Live Bus GPS** | "I'm on this Bus" FAB → pings GPS every 30 s to `live_vehicles` → shown as 🟢 marker to others on the route. |
| **Commute Tracker** | Once a route is confirmed, tracks GPS against the planned polyline and estimates arrival. |

### Map & Display
| Feature | Detail |
|---|---|
| **Tile Layers** | Default: CartoDB Voyager (coloured roads). Settings → Light Map (also Voyager) / Satellite (Esri). OS dark mode → auto-switches to CartoDB Dark. |
| **Transit Layers** | LRT, Anbessa, Minibus, Sheger Bus — each independently toggleable. Hidden by default; shown on demand. |
| **LRT Station Markers** | Dot markers from `lrt-stations.json`; hidden below zoom 14, visible above. |
| **User Location** | Animated GPS dot with compass arrow. Gold in driving mode (speed > ~14 km/h). |
| **Low-Data Mode** | Switches to CartoDB dark (no-labels) tile. Reduces unnecessary layer rendering. |
| **PWA / Offline** | Service Worker caches all JS/CSS/data/icons/fonts + Addis Ababa tiles at zoom 12–15. |

---

## 🏗 Architecture

```
Browser
 └── index.html (app shell)
     ├── Leaflet 1.9.4         — map, polylines, markers
     ├── leaflet.heat 0.2.0    — demand heatmap
     ├── leaflet-polylinedecorator 1.6.0  — direction arrows on routes
     ├── Supabase JS v2        — real-time heatmap + live vehicles
     └── App JS (vanilla, no framework, load-order dependent)
             config.js          → Supabase credentials, feature flags
             i18n.js            → string table, currentLang
             map.js             → Leaflet init, tile switching, flyTo
             gtfs-data.js       → GTFS_ROUTES registry, Haversine, nearest-stop, tariff helpers
             transfer-graph.js  → terminal adjacency graph, BFS multi-leg finder
             landmarks.js       → landmarksData, weighted search, favourites
             search.js          → autocomplete (per-input), suggest modal
             transit.js         → renderLRTLayer / renderAnbessaLayer / renderMinibusLayer / renderShegerLayer
             heatmap.js         → Supabase init, toggleWaiting, startDriverHeatmap
             crowdsource.js     → toggleOnBus, live vehicle markers
             routing.js         → planRoute, renderWalkOnly/Minibus/LRT/MultiLeg, OSRM walk legs
             commute-tracker.js → startCommuteTracking, GPS journey progress
             lowbandwidth.js    → toggleLowBandwidth
             app.js             → DOMContentLoaded bootstrap, wireUI, populateRouteDropdowns

Service Worker
  ├── App-shell cache  (JS, CSS, data, icons, fonts — cache-first)
  ├── Tile cache       (CartoDB + Esri — cache-first, Addis Ababa zoom 12–15 pre-cached)
  └── Supabase bypass  (always network-first — real-time data must be fresh)
```

---

## 🔄 GTFS Sync

```powershell
# Run manually to refresh route GeoJSON from DT4A
node scripts/sync-gtfs.js
```

Uses `ETag` / `Last-Modified` headers — only downloads if data has changed. State tracked in `data/.sync-state.json`.

---

## 📍 Key Coordinates

| Location | Lat | Lng |
|---|---|---|
| Meskel Square | 9.0109 | 38.7612 |
| Mexico Square | 9.0192 | 38.7573 |
| Piassa | 9.0332 | 38.7477 |
| Merkato | 9.0370 | 38.7284 |
| Megenagna | 9.0247 | 38.8006 |
| Bole Airport | 8.9779 | 38.7989 |
| CMC | 9.0543 | 38.8182 |
| Saris | 8.9838 | 38.7363 |

---

## Preview

- See [EthioRoute](./ethioroute.png)

---

Map data © [OpenStreetMap](https://openstreetmap.org) contributors · Tiles © [CARTO](https://carto.com) · Transit data © [Digital Transport for Africa](https://digitaltransport4africa.org)
