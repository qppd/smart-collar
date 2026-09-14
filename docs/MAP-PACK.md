# Map Pack — Provisioning Guide

The Map Pack is the bundle of map assets that makes the app render **fully offline**: MapLibre GL JS, the map style, and the OpenStreetMap data for the farm area. It is built **once** on a laptop with internet, copied into the PWA, and installed on the farmer's phone. After that the farm never needs the internet again (see [APP.md](APP.md)).

At a glance, the whole build is two commands plus some asset copying:

```bash
pmtiles extract <protomaps daily build> farm.pmtiles --bbox=<W>,<S>,<E>,<N> --maxzoom=15
# then copy farm.pmtiles + MapLibre dist + style + glyphs into the PWA's precache
```

## 0. What you are building

| Artifact | What it is | Typical size | Ends up at |
|---|---|---:|---|
| `farm.pmtiles` | Vector tiles (OpenStreetMap data) for the farm bbox + LoRa margin, single-file PMTiles | 10–30 MB | `assets/map/farm.pmtiles` |
| `maplibre-gl.js` + `maplibre-gl.css` | MapLibre GL JS v5 library (the rendering engine) | ~800 KB | `assets/map/` |
| `style.json` | Base style (Protomaps "light" theme), URLs rewritten to local paths | ~30 KB | `assets/map/style.json` |
| glyphs | Font PBFs for every label layer (e.g., Noto Sans Regular + Bold, ranges 0–655) | 1–3 MB | `assets/map/fonts/` |
| sprite | Icon atlas for the style | ~100 KB | `assets/map/sprite.png` + `.json` |

Total added to the PWA cache: **≈ 15–40 MB** per phone. Fine for any Android phone.

## 1. Define the bounding box (farm + margin)

1. Open [bboxfinder.com](http://bboxfinder.com), draw a box around the **outermost fence segments**, and read off **West, South, East, North** in decimal degrees (WGS84 — the same datum the collar's GPS outputs).
2. **Pad the box by the LoRa coverage margin** — the site-survey radius from [TESTING.md](TESTING.md) R1/R2, typically **2–5 km beyond the fence**. This margin is the requirement: an animal outside the geofence must still render on the offline map (marker, last-known label, trail) as far as the collar can still report (see [APP.md](APP.md), Map Data & Layers).
3. Margin math: 1° latitude ≈ 111 km; near Philippine latitudes 1° longitude ≈ 107 km. So **add ≈ 0.01° of margin per km of coverage radius** (use 0.05° for a 5 km radius, 0.03° for 3 km).

Worked example — fence box 120.98–121.05 E / 14.75–14.81 N with a 3 km site-survey radius:

```
W = 120.98 − 0.03 = 120.95   S = 14.75 − 0.03 = 14.72
E = 121.05 + 0.03 = 121.08   N = 14.81 + 0.03 = 14.84
```

## 2. Build `farm.pmtiles` from OpenStreetMap data

Install the `pmtiles` CLI (single binary — the go-pmtiles releases page on the Protomaps project), then pick one path.

### Path A — extract from the Protomaps daily build (recommended)

Grab the newest daily build name from the builds page (`maps.protomaps.com/builds`, files like `20260901.pmtiles`), then:

```bash
pmtiles extract https://build.protomaps.com/20260901.pmtiles farm.pmtiles --bbox=120.95,14.72,121.08,14.84 --maxzoom=15
```

- The `--bbox` order is **West, South, East, North** (longitude first).
- `--maxzoom=15` keeps z0–15; MapLibre overzooms beyond that for closer views. z15 shows fences, buildings, and farm tracks — plenty for herd positioning. Drop to 14 if you want the file smaller.

### Path B — build it yourself with Planetiler (no dependence on Protomaps builds)

Download the country extract from [download.geofabrik.de](https://download.geofabrik.de/asia/philippines.html) (`philippines-latest.osm.pbf`, ≈ 1 GB), then:

```bash
java -Xmx6g -jar planetiler.jar --area=philippines --download --output=ph.pmtiles
pmtiles extract ph.pmtiles farm.pmtiles --bbox=120.95,14.72,121.08,14.84 --maxzoom=15
```

Give Java as much heap as the machine allows; the country build needs several GB of RAM and disk. Use Path A unless you specifically want to control the build.

### Verify the extract

```bash
pmtiles show farm.pmtiles
```

Check: the header bbox matches your padded box, and the file size is in the 10–30 MB range. A quick visual check: open it in the PMTiles viewer web tool and confirm the farm, the boundary barangay roads, and land beyond the fence all render.

## 3. Bundle the MapLibre offline assets

1. **MapLibre GL JS**: from the maplibre-gl npm package or CDN, save `dist/maplibre-gl.js` and `dist/maplibre-gl.css` into `assets/map/`.
2. **Style**: take the Protomaps basemaps **light** theme JSON and rewrite its three URL classes to local relative paths:

```jsonc
{
  "sources": {
    "protomaps": {
      "type": "vector",
      "tiles": ["pmtiles://assets/map/farm.pmtiles"],
      "attribution": "(c) OpenStreetMap contributors"
    }
  },
  "glyphs": "assets/map/fonts/{fontstack}/{range}.pbf",
  "sprite": "assets/map/sprite"
}
```

3. **Glyphs**: copy the font stacks the style references (Noto Sans Regular / Bold) into `assets/map/fonts/`. All text layers — place names, road labels — need these; missing glyphs render as blank text.
4. **Sprite**: copy the style's icon atlas (`sprite.png`, `sprite.json`, plus `@2x` variants) into `assets/map/`.

## 4. Wire it into the PWA

Final layout inside the app:

```
app/
├── index.html
├── sw.js
└── assets/map/
    ├── farm.pmtiles        ← your extract (Path A or B)
    ├── style.json          ← rewritten to relative paths
    ├── maplibre-gl.js
    ├── maplibre-gl.css
    ├── sprite.png / sprite.json (+ @2x)
    └── fonts/NotoSansRegular/0-255.pbf ... (all ranges the style uses)
```

Add the Map Pack to the service-worker precache so the first install caches everything:

```js
// sw.js — extend the install-time precache
const MAP_PACK = [
  "./assets/map/style.json",
  "./assets/map/farm.pmtiles",
  "./assets/map/maplibre-gl.js",
  "./assets/map/maplibre-gl.css",
  "./assets/map/sprite.png",
  "./assets/map/sprite.json",
  // glyphs are deterministic paths: precache the ranges the style uses,
  // or cache-on-first-render with a cache-first strategy
];
```

In the app, register the PMTiles protocol with MapLibre (the `pmtiles` JS package — follow its README for the current API) before creating the map:

```js
import maplibregl from "maplibre-gl";
import { Protocol } from "pmtiles";

const protocol = new Protocol();
maplibregl.addProtocol("pmtiles", (params, abort) => protocol.tile(params, abort));

const map = new maplibregl.Map({
  container: "map",
  style: "./assets/map/style.json",
  center: [121.0, 14.78],   // farm center
  zoom: 13,
});
```

**Offline read pitfall (important):** MapLibre reads PMTiles with HTTP range requests. On an installed offline PWA there is no server, so either

- load the whole `farm.pmtiles` from Cache Storage as a Blob at startup and hand it to the pmtiles library's in-memory/Blob source (simplest and fine — the file is ≤ ~40 MB), **or**
- serve range requests from the service worker by slicing the cached full response into `206` responses.

Pick one and test it in airplane mode; a blank map after install almost always means range requests are falling through the service worker unhandled.

## 5. Install on the farmer's phone

1. Serve the built PWA from the laptop (`python -m http.server 8080` in the app folder).
2. Make the origin a **secure context** for Web Bluetooth — either `adb reverse tcp:8080 tcp:8080` (the phone then opens `http://localhost:8080`, which Chrome treats as secure) or serve over HTTPS with a self-signed cert. Without this, Chrome blocks Web Bluetooth on a plain LAN address.
3. Open the page in Chrome on the phone, then **Add to Home Screen**.
4. **Airplane mode** and verify (next section), then pair to `SmartCollar-Base` and set the DS3231 clock (see [BASE-STATION.md](BASE-STATION.md), one-time provisioning).

Repeat per phone. Re-provisioning is only needed when: the farm area or site-survey radius grows, the app is updated, or a new phone is added.

## 6. Verification checklist (airplane mode on)

- [ ] PWA opens from home screen with **no network at all**
- [ ] Basemap renders across the entire farm **and the full margin** — pan to every fence corner and 5 km past it; no blank tiles
- [ ] Labels render (glyphs OK) and style icons render (sprite OK)
- [ ] Zoom to z15 at the geofence polygon — fence lines, buildings, tracks visible
- [ ] Pair over BLE while still offline — a live collar marker lands on the offline basemap
- [ ] DevTools → Application → Cache Storage shows the expected ~15–40 MB
- [ ] Marker + trail remain drawn when a test position is placed beyond the geofence (acceptance item in [APP.md](APP.md))

## Maintenance

| Trigger | Action |
|---|---|
| Farm expands / site-survey radius grows | Re-run the extract with the new bbox; rebuild the PWA; re-provision phones |
| App update | Rebuild + re-provision (Map Pack steps 4–5 only, if assets unchanged) |
| OSM data improvements nearby matter (new roads) | Optional rebuild with a newer daily build — cosmetic, not required for operation |
