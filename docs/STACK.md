# Stack

Everything SmartCollar is built on — offline-first, low-power, and free.

---

## Collar Firmware (LilyGO T-Beam v2.x)

| Layer | Choice | Notes |
|---|---|---|
| Board | **LilyGO T-Beam v2.x** — ESP32 + NEO-M8N GPS + SX1276 LoRa + AXP2101 PMU + 18650 holder | One board = the whole collar |
| IDE / toolchain | **Arduino IDE 2.x** + esp32 by Espressif (core v2.x+) | |
| LoRa driver | **RadioLib** | Better SX1276 power control than the stock LoRa.h |
| GPS parsing | **TinyGPSPlus** | NEO-M8N NMEA sentences |
| Motion / activity | **Adafruit MPU6050** | Motion-detect interrupt: MOT_THR / MOT_DUR + accel-only cycle mode for wake-on-motion |
| Power management | **AXP2101** library (LilyGO / x-lqi) | Rail shutoff for sub-2 mA deep sleep |
| Serialization | **ArduinoJson** | Packet + config messages |

## Base Station Firmware (ESP32 DevKit + SX1278)

| Layer | Choice | Notes |
|---|---|---|
| Board | **ESP32 DevKit 38-pin** | LoRa RX, alarm rules, siren, app server |
| LoRa RX module | **Ai-Thinker Ra-02 (SX1278, 433 MHz)** | Must match collar band |
| LoRa driver | **RadioLib** | Same library both ends |
| App server | **ESPAsyncWebServer + WebSocket** | Local WiFi AP `SmartCollar-Base`, JSON stream |
| History store | **LittleFS** (CSV/JSON per collar) | ~100 KB/collar/month at 10-min intervals |
| Reliability | **ESP32 task watchdog** | Unattended hardware must self-heal |

## App (offline-first PWA)

| Layer | Choice | Notes |
|---|---|---|
| Platform | **PWA served by the base station** (`http://smartcollar.local`) | One codebase → phone + desktop; installs to home screen |
| Maps | **Leaflet + pre-cached OSM tiles** (z14–z17, farm bounding box) | Tiles served from the base — the base *is* the internet |
| Alternative imagery | Mobile Atlas Creator → MBTiles → Leaflet | Optional satellite export during site survey (fence lines matter) |
| Live updates | **WebSocket** JSON stream | Positions + alarms push |
| Notifications | Service-worker push + persistent notification | Works with PWA backgrounded |
| Connectors | Base WiFi AP (default) · USB serial (Web Serial in Chrome) | Zero-internet by design |

## Radio Link

| Setting | Value | Why |
|---|---|---|
| Type | **Raw point-to-multipoint LoRa** (not LoRaWAN) | No gateway/network-server exists offline; we control both ends |
| Frequency | 433 MHz ISM (PH allocation; 915 variant must match base) | |
| Spreading factor | SF7–SF9 | SF7 fast/short; SF9 for far paddocks — per-site config |
| Sync word | private `0x2B` | Keeps foreign LoRa traffic out |
| MAC | interval TX + ±5% jitter · 3× emergency retry @ 30 s · daily timeslot-sync broadcast | Herd-sync collisions and battery drain avoided |
| Packet | binary, ≤ 23 bytes body | Position, battery, tamper flags, activity, SEQ |

## Hardware (context — see [BOM.md](BOM.md) & [HARDWARE.md](HARDWARE.md))

| Part | Role |
|---|---|
| T-Beam v2.x + 18650 | Collar core — compute, GPS, LoRa, power |
| Conductive tamper loop + reed switch | Supervised resistance loop on the adjustable strap |
| MPU6050 accelerometer | 24-hour no-movement rule + wake-on-motion |
| IP67 enclosure + glands + sealant | Wallow/submersion-proof housing |
| Flexible whip antenna | LoRa uplink, bulkhead-mounted |
| ESP32 DevKit + Ra-02 + mast antenna | Base receiver |
| 12 V ≥110 dB siren + relay | Audible alarm at the farmstead |
| 5 V solar panel + Li charge module (Phase 2) | Lid-swap upgrade, no redesign |

---

## Why this stack

- **Zero internet, zero subscriptions** — GPS satellites + LoRa radio + a local WiFi AP are the only links; nothing leaves the farm.
- **One radio library both ends** — RadioLib on collar and base keeps packet handling identical and debuggable.
- **PWA instead of native** — one codebase covers "offline mobile/desktop app" with no store, no build chain per platform.
- **ESP32 everywhere** — collar, base, and app server share one toolchain, one firmware skill set.
- **Power-first choices** — RadioLib power control, AXP2101 rail shutoff, MPU6050 accel-only cycle mode (GY-521 LED/LDO stripped — see POWER.md): every library pick was made for the ~2-week battery target.
- **All free & open-source** — software cost ₱0; budget lives in hardware.
