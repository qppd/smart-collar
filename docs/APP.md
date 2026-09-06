# Offline App — Dashboard & Geofence Editor

The app is the farmer's window into the system: live locations, alarm feed, per-collar status, and the geofence editor. It runs **with zero internet** — connecting straight to the base station over local WiFi or USB.

---

## Platform Choice

| Option | Offline maps | Cost/effort | Verdict |
|---|---|---|---|
| **PWA (web app) served by the base station** (browser on phone/laptop) | Offline tiles cached via service worker | Low — one codebase, no store | **Chosen for Phase 1** |
| Native Android app (Kotlin/Flutter) | MBTiles in assets | Medium-high | Phase 2 if PWA limits show |
| Desktop-only (Electron/Tauri) | Easiest tiles | Medium | Fallback — farms want phone-in-pocket |

Phase 1: the base station serves the PWA; the farmer opens `http://smartcollar.local` (or the AP gateway IP) in any phone browser and it **installs to home screen**. One app for mobile *and* desktop — matches the "Offline Mobile/Desktop App" requirement with a single build.

## Connectivity

| Mode | How | When |
|---|---|---|
| Base WiFi AP | Phone connects to `SmartCollar-Base` | Default in the field |
| USB | Laptop/phone ↔ base ESP32 (Web Serial in Chrome) | Most reliable; zero WiFi |

## Core Screens

1. **Live Map** — all collars, color by status (green OK, amber warning, red alarm); last-seen timestamp; tap collar → history trail.
2. **Alarm Feed** — reverse-chronological events: type, collar, position, time, ACK button (ACK silences the siren at the base).
3. **Geofence Editor** — draw/edit polygon(s) on the map (max 16 vertices per paddock zone); save → base pushes to collars at next wake window.
4. **Collar Detail** — battery %, loop resistance trend, reporting interval, activity graph (24 h), tamper latch status, service-mode controls.
5. **Maintenance** — battery swap log, lid-open events, collar assignment (ID ↔ animal name/photo).

## Offline Maps (the hard part)

No internet means no Google Maps tiles. Options:

| Approach | Notes |
|---|---|
| **Pre-cached OSM tiles** (Leaflet + cached tile set for the farm's bounding box, z14–z17) | Simple; farm area is small (~few km² → tens of MB); served from the base station itself |
| Custom satellite imagery export (Mobile Atlas Creator → MBTiles → Leaflet) | Better for visual fence lines; slightly more setup |

**Chosen: pre-cached OSM tiles stored on the base station** and served to the PWA over local WiFi — the base is the "internet" for the app. Add a satellite-imagery export during site survey (the fence lines and paddocks matter more than street names).

## Alarm UX

- Siren pattern + push notification + in-app banner (service-worker push while PWA in background — tested; keep a persistent notification).
- ACK requires acknowledging **the event, not the collar** (each alarm event is ACKed individually).
- Low-battery/collar-silence alerts are app-only (they'd cause siren fatigue).
- All alarms also logged to history — "did I miss anything yesterday?" is a first-class screen.

## Data & Sync

- Base station keeps history (LittleFS); the app pulls delta-sync on connect (`since=` cursor).
- Export: CSV download of positions/events per date range (for the capstone evaluation logs).
- Geofence config + per-collar settings live on the base; app is a view/controller, so any phone can manage the farm with no pairing.

## Stretch (post-capstone)

- Multi-base support display (if farm adds a second base)
- Offline routing: "walk to last known position" compass arrow (phone GPS vs collar position)
- Herd analytics: grazing heat map, activity anomaly detection ("carabao 5 is 40% less active this week")

## Acceptance Checklist

- [ ] Fresh phone (no internet, no prior install) connects to base AP and loads the app
- [ ] Home-screen install works; app opens offline from cache
- [ ] Live position updates < 10 s of packet arrival at base
- [ ] Geofence drawn on phone visible on desktop and vice versa
- [ ] ACK on phone silences the siren within 2 s
- [ ] 7 days of history browsable; CSV export downloads correctly
