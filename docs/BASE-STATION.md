# Base Station

The base station is the farm's single fixed infrastructure point: it receives LoRa packets from every collar, runs the alarm rules, drives the siren, and serves the offline app.

---

## Hardware

| Part | Role |
|---|---|
| ESP32 DevKit (38-pin) | Controller: LoRa RX, alarm rules, siren, app server |
| Ai-Thinker Ra-02 (SX1278, 433 MHz) | LoRa receiver — **must match collar band** |
| 3–5 dBi antenna + 3–5 m mast | Coverage (height = range) |
| 12 V ≥ 110 dB siren + relay module | Audible alarm |
| 12 V 2A adapter + buck to 5V | Power |
| IP65+ junction box | Outdoor protection at the mast |

Radio settings and packet format: [FIRMWARE.md](FIRMWARE.md).

## Placement & Coverage

LoRa at SF7–SF9, 433 MHz, with a mast-mounted base antenna reaches **2–8 km** in open/rolling terrain, **300 m–2 km** through dense trees — enough for a typical grazing farm, but the site survey decides.

1. **Site survey first (before buying the mast):** carry a battery-packaged collar prototype (or T-Beam + power bank) around the farm boundary and paddock corners; walk with the base on a bench (or a second person at the base); log RSSI/SNR at each waypoint. A Saturday afternoon of walking beats a month of "why is collar 3 always missing."
2. **Mast:** 3–5 m above ground at the farmstead's highest practical point; antenna vertical, clear of metal roofing by ≥ 1 m.
3. **Fallback modes:** if a paddock is dead-zone, options: raise mast, higher SF for that collar, or a second base station (the protocol supports >1 base — collars just need one to hear them).
4. **Siren audibility:** the siren sounds at the base; place it where it's heard at the farmhouse — that's the point of the alarm.

## Power & Protection

- Powered from farmhouse mains via the 12 V adapter; add a **surge protector** (lightning-prone rural areas) and a small UPS/power bank on the 5 V rail so short brownouts don't lose state (ESP32 keeps history in LittleFS, so worst case is a reboot).
- Antenna surge/grounding: mast grounded; antenna disconnect during thunderstorms if practical (collars buffer-and-retry — an offline hour is fine).
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

## Siren Patterns

| Alarm | Pattern |
|---|---|
| Geofence | 3 short bursts, repeat every 2 min |
| Tamper | Continuous until ACKed (most urgent) |
| No movement | 10 s blast every 5 min |
| Multi-alarm | Tamper wins priority; patterns queue |

## App Connectivity (offline)

The base station runs a **local WiFi access point** (always on, no internet) plus optional USB serial:

- SSID `SmartCollar-Base` · pass `carabao####`
- WebSocket JSON stream of live positions/alarms + REST-ish endpoints for geofence and config (see [APP.md](APP.md))
- Desktop/laptop can also connect by **USB cable** (most reliable, no WiFi at all needed)
- App works fully offline: maps cached on the device (see [APP.md](APP.md)); the base is just a local server.

## Installation Checklist

- [ ] Site survey complete: RSSI/SNR map of farm boundary + paddocks
- [ ] Mast installed, antenna vertical, grounded
- [ ] Base box sealed, cable glands drip-looped
- [ ] Siren audible at farmhouse (test with someone inside, windows closed)
- [ ] Brownout test: pull power 10 s → base reboots, history intact, alarms re-evaluated
- [ ] Lightning-season checklist posted (antenna disconnect procedure)
