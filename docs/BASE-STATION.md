# Base Station

The base station is the farm's single fixed infrastructure point: it receives LoRa packets from every collar, runs the alarm rules, drives the siren, and serves the offline app.

---

## Hardware

| Part | Role |
|---|---|
| ESP32 DevKit (38-pin) | Controller: LoRa RX, alarm rules, siren, app server |
| Ai-Thinker Ra-02 (**SX1278**, **433 MHz**) | LoRa receiver — **must match collar band** |
| 3–5 dBi antenna + 3–5 m mast | Coverage (height = range) |
| 12 V ≥ 110 dB siren + relay module | Audible alarm — relay IN1 on ESP32 GPIO17 (wiring: [WIRING.md](WIRING.md)) |
| Switching PSU **S-60-12 (12 V 5 A 60 W)** + Circuitrocks 36V→5V 5A buck | Power — 12 V rail drives the siren directly; buck makes the 5 V rail |
| IP65+ junction box | Outdoor protection at the mast |

Radio settings and packet format: [FIRMWARE.md](FIRMWARE.md).

## Placement & Coverage

**LoRa at SF7–SF9, 433 MHz**, with a mast-mounted base antenna reaches **2–8 km** in open/rolling terrain, **300 m–2 km** through dense trees — enough for a typical grazing farm, but the site survey decides.

1. **Site survey first (before buying the mast):** carry a battery-packaged collar prototype (or ESP32 DevKit + power bank) around the farm boundary and paddock corners; walk with the base on a bench (or a second person at the base); log RSSI/SNR at each waypoint. A Saturday afternoon of walking beats a month of "why is collar 3 always missing."
2. **Mast:** 3–5 m above ground at the farmstead's highest practical point; antenna vertical, clear of metal roofing by ≥ 1 m.
3. **Fallback modes:** if a paddock is dead-zone, options: raise mast, higher SF for that collar, or a second base station (the protocol supports >1 base — collars just need one to hear them).
4. **Siren audibility:** the siren sounds at the base; place it where it's heard at the farmhouse — that's the point of the alarm.

## Power & Protection

- Powered from farmhouse mains via an enclosed **12 V 5 A switching PSU (S-60-12)**: the 12 V rail feeds the siren directly (a 110 dB siren can pull over 1 A — well within the 5 A budget), and the buck drops 12 V → 5 V for the ESP32 + LoRa. Mount the PSU inside the IP65 box with strain-relieved mains cable and a drip loop — **no UPS and no surge protector by decision**: LittleFS history survives reboots, the watchdog self-heals, and collars buffer-and-retry through any outage.
- Lightning handling relies on **passive protection + human procedure only**: mast grounded; antenna disconnect during thunderstorms if practical — collars buffer-and-retry, an offline hour is fine.
- IP65 box shaded if the sun hits it directly (interior < 60 °C).

## Alarm Rules (server-side)

| Rule | Input | Fires when |
|---|---|---|
| **Geofence breach** | Latest position per collar | Point outside configured polygon (with hysteresis: must be out for 2 consecutive packets to avoid GPS jitter false alarms) |
| **Tamper** | Collar FLAGS | Any tamper bit set (cut / buckle / bypass subtype preserved) |
| **No movement 24 h** | ACTIVITY score | Below threshold continuously 24 h |
| Low battery | BAT_PERCENT | < 20% (app alert only, no siren) |
| Collar silence | RX watchdog | No packet from a collar for 3× its reporting interval → app alert (could be dead battery, dead zone, or an unusual event — check) |

Hysteresis, debounce, and re-arm logic: an alarm ACKed in the app silences the siren but the event stays until the condition clears AND a clear event is seen (e.g., collar back inside fence 2 consecutive packets).

### Maintenance Mode (alarm suppression)

Toggled from the app (BLE `MAINT_ON {minutes}` / `MAINT_OFF`; default 30 min, max 2 h, **auto-expires**):

- **ON:** geofence, tamper, and no-movement rules do **not** fire the siren or raise alarms — collar removal, battery swaps, and strap work proceed silently.
- Events detected while suppressed are **logged as `suppressed` entries** in history — never silently dropped.
- Still active: low-battery and collar-silence app alerts, all LoRa reception, history recording.
- **Every ON/OFF/expiry is logged** with a timestamp — a forgotten toggle can never mute the farm quietly.

## Siren Patterns

| Alarm | Pattern |
|---|---|
| Geofence | 3 short bursts, repeat every 2 min |
| Tamper | Continuous until ACKed (most urgent) |
| No movement | 10 s blast every 5 min |
| Multi-alarm | Tamper wins priority; patterns queue |

## App Connectivity (Bluetooth LE — offline)

The base station runs a **Bluetooth LE GATT server** (always on) plus a USB serial console — **no WiFi, no internet**:

- BLE device name `SmartCollar-Base`; the app pairs via Web Bluetooth (see [APP.md](APP.md))
- GATT layout: one JSON **notify** characteristic (live positions/alarms, MTU-chunked) + one **write** characteristic (commands: geofence save, ACK, interval, `TIME_SET`, `MAINT_ON`/`MAINT_OFF`) + one **read** characteristic (base status: uptime, clock state, collar count)
- Desktop/laptop can also connect by **USB cable** (bench work, flashing, clock setting)
- App works fully offline: MapLibre + OpenStreetMap data live in each phone's app cache (see [APP.md](APP.md)); the base is a local data source over BLE.

### One-time provisioning

Before deployment, provision each phone once with WiFi — see **[APP.md](APP.md) §One-Time Provisioning** for the full step-by-step (MapLibre + OpenStreetMap download, PWA install, BLE pairing). The base station sets its clock from the phone on every BLE connect (`TIME_SET`) — no RTC hardware; after a reboot with the phone away, the clock is flagged `unsynced` until the next connect. No alarm rule depends on wall time.

## Installation Checklist

- [ ] Site survey complete: RSSI/SNR map of farm boundary + paddocks
- [ ] Mast installed, antenna vertical, grounded
- [ ] Base box sealed, cable glands drip-looped
- [ ] Siren audible at farmhouse (test with someone inside, windows closed)
- [ ] Brownout test: pull power 10 s → base reboots, history intact, alarms re-evaluated; clock shows `unsynced` until the phone reconnects
- [ ] Lightning-season checklist posted (antenna disconnect procedure)
