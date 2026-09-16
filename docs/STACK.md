# Stack

Everything SmartCollar is built on — offline-first, low-power, and free.

---

## Collar Firmware (LilyGO T-Beam v2.x)

| Layer | Choice | Notes |
|---|---|---|
| Board | **LilyGO T-Beam v2.x** — ESP32 + NEO-M8N GPS + SX1276 LoRa + AXP2101 PMU + 18650 holder | One board = the whole collar |
| IDE / toolchain | **Arduino IDE 2.x** + esp32 by Espressif (core v2.x+) | |
| LoRa driver | **RadioLib** | Better SX127x power control than the stock LoRa.h |
| GPS parsing | **TinyGPSPlus** | NEO-M8N NMEA sentences |
| Motion / activity | **Adafruit MPU6050** | Motion-detect interrupt (MOT_THR / MOT_DUR, accel-only cycle mode) feeds **activity counting only** — motion never triggers a GPS/TX cycle (power guard, see POWER.md) |
| Power management | **AXP2101** library (LilyGO / x-lqi) | Rail shutoff for sub-2 mA deep sleep |
| Serialization | **ArduinoJson** | Packet + config messages |

## Base Station Firmware (ESP32 DevKit + SX1276)

| Layer | Choice | Notes |
|---|---|---|
| Board | **ESP32 DevKit 38-pin** | LoRa RX, alarm rules, siren, app server |
| LoRa RX module | **Ai-Thinker Ra-02 (SX1276, 915 MHz)** | Must match collar band |
| LoRa driver | **RadioLib** | Same library both ends |
| App server | **ESP32 BLE stack (NimBLE-Arduino) + USB serial** | GATT server `SmartCollar-Base`; JSON over notify/write characteristics, MTU-chunked |
| History store | **LittleFS** (CSV/JSON per collar) | ~100 KB/collar/month at 10-min intervals |
| Clock | **Phone-set via BLE** (`TIME_SET` on app connect) | No RTC hardware; `unsynced` flag after an offline reboot — no alarm rule depends on wall time |
| Reliability | **ESP32 task watchdog** | Unattended hardware must self-heal |

## App (offline-first PWA)

| Layer | Choice | Notes |
|---|---|---|
| Platform | **PWA installed on the phone at provisioning** | One codebase → phone + desktop; installs to home screen; opens fully offline. Copied to the phone once (Web Bluetooth to the base afterwards) |
| Maps | **MapLibre GL JS + offline OpenStreetMap data** (farm bbox + LoRa coverage margin) | One-time provisioning download into the phone — the map renders offline from the phone, never from the internet; markers stay visible past the geofence |
| Alternative imagery | Mobile Atlas Creator → MBTiles → MapLibre raster/style | Optional satellite export during site survey (fence lines matter) |
| Live updates | **BLE notify** JSON stream (MTU-chunked) · USB serial (bench) | Positions + alarms pushed from the base on packet reception |
| Notifications | Service-worker local notification + persistent banner | On-device — no push service, works with PWA backgrounded |
| Connectors | Bluetooth LE (default) · USB serial (Web Serial in Chrome) | Zero-internet by design |

## Radio Link

| Setting | Value | Why |
|---|---|---|
| Type | **Raw point-to-multipoint LoRa** (not LoRaWAN) | No gateway/network-server exists offline; we control both ends. CRC16 on every frame, 3 s RX window after every TX (downlink spec: [FIRMWARE.md](FIRMWARE.md)) |
| Frequency | 915 MHz ISM (PH allocation; 433 variant must match base) | |
| Spreading factor | SF7–SF9 | SF7 fast/short; SF9 for far paddocks — per-site config |
| Sync word | private `0x2B` | Keeps foreign LoRa traffic out |
| MAC | interval TX + ±5% jitter · 3× emergency retry @ 30 s · daily timeslot-sync broadcast | Herd-sync collisions and battery drain avoided |
| Packet | binary, ≤ 23 bytes body | Position, battery, tamper flags, activity, SEQ |

## Hardware (context — see [BOM.md](BOM.md) & [HARDWARE.md](HARDWARE.md))

| Part | Role |
|---|---|
| T-Beam v2.x + 18650 | Collar core — compute, GPS, LoRa, power |
| Conductive tamper loop with buckle break-away link | Supervised resistance loop on the adjustable strap — bands relative to each collar's recorded baseline R₀ |
| MPU6050 accelerometer | 24-hour no-movement rule + activity counting |
| IP67 enclosure + glands + sealant | Wallow/submersion-proof housing |
| Flexible whip antenna | LoRa uplink, bulkhead-mounted |
| ESP32 DevKit + Ra-02 + mast antenna | Base receiver |
| 12 V ≥110 dB siren + relay | Audible alarm at the farmstead |
| 5 V solar panel + Li charge module (Phase 2) | Lid-swap upgrade, no redesign |

---

## Why this stack

- **Offline end to end** — GPS satellites + LoRa radio + Bluetooth LE are the only links the system needs; the app's map (MapLibre + OpenStreetMap data) is downloaded once during provisioning and never again. No cloud, no accounts, no subscriptions.
- **One radio library both ends** — RadioLib on collar and base keeps packet handling identical and debuggable.
- **PWA instead of native** — one codebase covers "offline mobile/desktop app" with no store, no build chain per platform.
- **ESP32 everywhere** — collar, base, and app server share one toolchain, one firmware skill set.
- **Power-first choices** — RadioLib power control, AXP2101 rail shutoff, motion flag polled (never an interrupt wake), 3 s post-TX RX windows, MPU6050 accel-only cycle mode (GY-521 LED/LDO stripped — see POWER.md): every pick was made for the ~2-week battery target.
- **All free & open-source** — software cost ₱0; budget lives in hardware.
