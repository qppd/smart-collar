# Block Diagram

Hardware blocks and the signals that connect them — the physical view of SmartCollar. Diagrams are **Mermaid** and render in any Mermaid-compatible viewer.

---

## 1. Overall System Block Diagram

```mermaid
flowchart TB
    GPS["GPS satellites"]

    subgraph COW["CARABAO — wears it 24/7"]
        subgraph COLLAR["SMART COLLAR (IP67)"]
            TBEAM["LilyGO T-Beam v2.x<br/>ESP32 · NEO-M8N GPS · SX1278 LoRa · AXP2101 PMU"]
            LOOP["Supervised tamper loop<br/>wire rope + buckle reed + far-end 100 Ω<br/>+ 10 kΩ bias → GPIO36 ADC · GPIO25 drive"]
            ACCEL["MPU6050 accelerometer (GY-521)<br/>INT → GPIO39 motion-detect (activity count)"]
            HALL["Service-mode hall sensor<br/>GPIO13 (maintenance magnet)"]
            CELL["18650 Li-ion cell<br/>3,500 mAh (Phase 2: solar lid)"]
            WHIP["Whip antenna<br/>bulkhead + strain relief"]
        end
    end

    subgraph BASE["BASE STATION — farmhouse, powered"]
        DEVKIT["ESP32 DevKit 38-pin<br/>LoRa RX · alarm rules engine · siren driver"]
        RA02["Ai-Thinker Ra-02<br/>SX1278 433 MHz<br/>(must match collar band)"]
        MAST["3–5 dBi antenna<br/>3–5 m mast, grounded"]
        SIREN["12 V ≥ 110 dB siren<br/>via relay module"]
        STORE["LittleFS history<br/>CSV/JSON per collar"]
        APSRV["App server<br/>BLE GATT: notify + write<br/>+ USB serial console"]
    end

    subgraph FARM["FARMER'S DEVICE (offline)"]
        PWA["PWA — Web Bluetooth to base<br/>MapLibre + offline OpenStreetMap data<br/>dashboard · geofence editor · maintenance toggle"]
    end

    GPS -- "NEO-M8N fix" --> TBEAM
    LOOP -- "ADC band check vs R₀" --> TBEAM
    ACCEL -- "motion INT" --> TBEAM
    HALL -- "10 s magnet hold" --> TBEAM
    CELL -.-> TBEAM
    TBEAM -- "SX1278 TX" --> WHIP
    WHIP <-. "LoRa 433 MHz · SF7–SF9 · ≤23 B packet<br/>2–8 km open / 300 m–2 km trees" .-> MAST
    MAST -- "SX1278 RX" --> RA02
    RA02 -- "SPI" --> DEVKIT
    DEVKIT -- "relay" --> SIREN
    DEVKIT --- STORE
    DEVKIT --- APSRV
    APSRV <-. "Bluetooth LE (or USB serial)" .-> PWA
```

**Flows:** collar→base is LoRa uplink (offline radio) · base→collar is downlink (geofence push, interval change, ACKs, clear-tamper-latch) · base→farmer is **Bluetooth LE** or USB — nothing touches the internet; the map data was downloaded once during provisioning.

---

## 2. Tamper Loop Sense Circuit

```mermaid
flowchart LR
    G25["GPIO25 drive<br/>(excites only during measure)"] -- "10 kΩ bias" --> NODE["sense node"]
    NODE -- "GPIO36 ADC<br/>band check vs R₀ ± 30%" --> ESP["ESP32 band check<br/>8-sample average, 100 ms debounce"]
    NODE -- "wire rope + buckle reed<br/>+ far-end 100 Ω" --> FAR["far end"]
    FAR -- "GND" --> GND["GND"]
    ESP -- "outside window →" --> LATCH["latched alarm flag<br/>immediate high-priority packet"]
```

| Attack | ADC sees | Result |
|---|---|---|
| Intact collar | designed resistance (~R₀ baseline) | normal |
| **Cut** anywhere (incl. adjustment tail) | full pull-up | alarm |
| **Bypass** (jumper near body) | skips far-end resistor → too low | alarm |
| Splice-in wire to fake resistance | momentary open during splice | latched alarm |

---

## 3. Signal Chain Summary

| Signal | Source → Destination | Conditioning | Meaning |
|---|---|---|---|
| GPS fix | NEO-M8N → ESP32 UART (TinyGPSPlus) | warm-start ephemeris in RTC RAM | position; no fix → last fix flagged STALE |
| Tamper ADC | loop → GPIO36 via 10 kΩ bias | 8-sample avg, 100 ms debounce, bands vs baseline R₀ (R₀ ± 30% OK · R₀+220 Ω buckle · < 40 Ω short · > 2 kΩ open) | resistance signature vs. cut/bypass |
| Loop drive | GPIO25 → loop | excited only during measurement | power save + defeats corrosion-offset attacks |
| Motion INT | MPU6050 INT → GPIO39 | motion-detect interrupt (accel-only cycle mode) | activity counter for 24 h rule + wake |
| Service mode | hall sensor → GPIO13 | magnet held 10 s | maintenance window (no alarms) |
| Battery telemetry | AXP2101 → ESP32 | BAT_PERCENT in packet | low-battery app alert (< 20%) |
| LoRa uplink | collar SX1278 → base Ra-02 (SX1278) | SF7–SF9, sync 0x2B, ≤23 B, ±5% jitter | position + flags + battery + activity |
| LoRa downlink | base → collar | at wake windows / daily sync | geofence polygon, interval, ACK, latch clear |
| Maintenance mode | app → BLE → rules engine | MAINT_ON {min} / MAINT_OFF, auto-expires | gates siren/alarm path; events logged as `suppressed` |
| Siren drive | GPIO27 → relay IN1 → 12 V siren | patterned per alarm type | geofence 3-short/2-min · tamper continuous · no-move 10 s/5 min |
| Base LoRa SPI | Ra-02 ↔ ESP32 (VSPI 18/19/23/5 · RST 14 · DIO0 26) | 3.3 V only — decoupled supply | SX1278 RX/TX control ([WIRING.md](WIRING.md)) |

---

Electrical details: [HARDWARE.md](HARDWARE.md) · loop design: [TAMPER.md](TAMPER.md) · Logic: [FLOWCHART.md](FLOWCHART.md) · Layers: [SYSTEM-ARCHITECTURE.md](SYSTEM-ARCHITECTURE.md)
