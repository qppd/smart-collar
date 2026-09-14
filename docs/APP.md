# Offline App — Dashboard & Geofence Editor

The app is the farmer's window into the system: live locations, alarm feed, per-collar status, and the geofence editor. It runs **fully offline** — the connection to the base station is **Bluetooth LE** (Web Bluetooth in the browser), and the map renders from data pre-downloaded onto the phone. The internet is needed **once, during provisioning** — to download MapLibre and the OpenStreetMap map data — never again afterwards.

---

## Platform Choice

| Option | Offline map | Cost/effort | Verdict |
|---|---|---|---|
| **PWA (web app) installed on the phone** (Web Bluetooth to the base) | MapLibre + offline OSM data bundled in the app's cache | Low — one codebase, no store | **Chosen for Phase 1** |
| Native Android app (Kotlin/Flutter) | MBTiles in assets | Medium-high | Phase 2 if PWA limits show |
| Desktop-only (Electron/Tauri) | Easiest bundling | Medium | Fallback — farms want phone-in-pocket |

Phase 1: the built PWA bundle is copied onto the phone **once** during provisioning (e.g., served briefly from the laptop over USB) and **installs to home screen**. After that it opens offline every time. One app for mobile *and* desktop — matches the "Offline Mobile/Desktop App" requirement with a single build.

> **Browser support note:** Web Bluetooth works in Chrome/Edge (Android + desktop). iPhones (Safari) do not support it — any Android phone or laptop is the farmer's device. This is an accepted Phase-1 constraint.

## Connectivity — Bluetooth LE to the base

| Mode | How | When |
|---|---|---|
| **Bluetooth LE** (default) | Phone pairs to the base's BLE GATT server (`SmartCollar-Base`), read/notify + write characteristics | Everyday use — no WiFi anywhere in the system |
| USB | Laptop ↔ base ESP32 serial | Bench work, flashing, diagnostics |

- Range: BLE indoor class ≈ **10–30 m** — walk to the farmhouse/near the base to sync. Designed that way: the siren and the dashboard live at the farmstead.
- Live data: the base pushes position/alarm updates as BLE notify packets (JSON, compacted to fit MTU chunks).
- Commands (geofence save, ACK, interval change, maintenance toggle) are BLE writes, applied and confirmed by the base.

## One-Time Provisioning (the only internet the project ever needs)

Done once before deployment — at home/school with any WiFi:

1. **Download MapLibre GL JS** (bundle + glyphs + base style JSON) into the app's cache — full step-by-step build (bbox + margin, PMTiles extract, PWA bundling): [MAP-PACK.md](MAP-PACK.md).
2. **Download OpenStreetMap map data** for the farm bounding box + **LoRa coverage margin** (the site-survey radius from [TESTING.md](TESTING.md) R1/R2 — typically 2–5 km beyond the outermost fence). Vector tiles or a region extract (e.g., Protomaps/OSM PBF → offline tiles) stored in the app. **This margin is why a breached animal stays visible on the offline map even beyond the geofence** — as far as the collar can still report a position, the map has data there.
3. **Service-worker precache** of the whole PWA (app code, MapLibre, map data, icons).
4. **Pair over BLE** — the app sets the base clock automatically (`TIME_SET` from the phone's date/time on every connect; no RTC hardware on the base).
5. Done — from this point the system never touches the internet again. Re-provisioning is only needed to expand coverage or update the app.

## Map Data & Layers

The map screen is built from these layers (all offline):

```
Map
├── Offline basemap          ← OSM data (MapLibre), farm bbox + LoRa margin
├── Carabao #1               ← live marker (green/amber/red by status)
├── Carabao #2
├── Carabao #3
├── Base Station             ← fixed farmhouse marker
├── Geofence boundary        ← editable polygon overlay (≤ 16 vertices/paddock)
├── Last known location      ← marker + STALE-age label when no fresh fix
└── Movement/history trail   ← breadcrumb polyline from history
```

- **No clipping to the geofence:** the basemap extends past the fence by the coverage margin. An animal outside the polygon is still rendered — marker, last-known label, and trail — because theft/breach is exactly when the farmer needs to see *where it went*.
- **Last known location** is always shown with its age ("last seen 12 min ago"); STALE fixes keep the marker but flag it amber.
- Trails are pulled from base history over BLE (per-collar, time-ranged).

## Core Screens

1. **Live Map** — the layered map above; all 3 collars color by status; tap collar → detail + trail.
2. **Alarm Feed** — reverse-chronological events: type, collar, position, time, ACK button (ACK silences the siren at the base).
3. **Geofence Editor** — draw/edit polygon(s) on the offline map (max 16 vertices per paddock zone); save → BLE write → base pushes to collars at next wake window.
4. **Collar Detail** — battery %, loop resistance trend, reporting interval, activity graph (24 h), tamper latch status, service-mode controls.
5. **Maintenance** — battery swap log, lid-open events, collar assignment (ID ↔ animal name/photo), R₀ re-commission.
6. **Maintenance Mode toggle** — see below.

## Maintenance Mode Toggle (alarm suppression switch)

A prominent **toggle** in the app ("Maintenance mode") for deliberate work on collars:

| Aspect | Behavior |
|---|---|
| Purpose | Removing a collar, battery swap, strap adjustment, re-commissioning R₀ — **without triggering the siren or alarm feed** |
| When ON | The base **suppresses alarm triggering** (geofence, tamper, no-movement) and siren patterns. Events still detected while suppressed are **logged to history as `suppressed` entries**, never silently dropped |
| Still active | Battery/low-battery app alerts, RX-watchdog collar-silence alerts, and all LoRa packet reception — only the alarm/siren path is gated |
| Auto-expiry | Toggle carries a duration (default **30 min**, max 2 h, configurable). It **never latches silently** — the base re-enables alarms when it expires and logs that too, so a forgotten toggle can't mute the farm |
| How it works | BLE write `MAINT_ON {minutes}` / `MAINT_OFF` → base sets/clears its maintenance flag in the rules engine ([FIRMWARE.md](FIRMWARE.md)) |
| Audit | Every entry/exit/expiry logged with a timestamp — "why was the siren quiet yesterday?" is answerable from history |

## Alarm UX

- Siren pattern + in-app banner + persistent notification while the PWA is backgrounded (on-device local notification — no push service involved).
- ACK requires acknowledging **the event, not the collar** (each alarm event is ACKed individually).
- Low-battery/collar-silence alerts are app-only (they'd cause siren fatigue) — and they are **not** suppressed by Maintenance mode.
- All alarms also logged to history — "did I miss anything yesterday?" is a first-class screen.

## Data & Sync

- Base station keeps history (LittleFS); the app pulls delta-sync over BLE on connect (`since=` cursor; long transfers chunked across MTUs — a day of history for 3 collars ≈ tens of KB, seconds over BLE).
- Export: CSV download of positions/events per date range (for the capstone evaluation logs).
- Geofence config + per-collar settings live on the base; the app is a view/controller, so any provisioned phone can manage the farm.

## Acceptance Checklist

- [ ] Provisioned phone (airplane mode on) pairs over BLE and loads the app fully offline
- [ ] Map renders with all layers; **marker + trail remain visible well beyond the geofence polygon**
- [ ] Live position update arrives via BLE notify < 2 s of packet reception at base (phone in range)
- [ ] Geofence drawn on the phone reaches the collars within 24 h; visible in the editor after reconnect
- [ ] ACK silences the siren within 2 s
- [ ] **Maintenance toggle ON → cut/adjust a collar: no siren, event logged as `suppressed`; toggle OFF (or expiry) → next tamper event alarms normally**
- [ ] Maintenance auto-expiry re-enables alarms and is logged
- [ ] 7 days of history browsable; CSV export downloads correctly
