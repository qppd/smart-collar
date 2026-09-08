# Flowchart

Runtime behavior, decision by decision — the logic view of SmartCollar, matching the actual firmware state machines in FIRMWARE.md. Diagrams are **Mermaid** and render natively on GitHub.

---

## 1. Collar Main State Machine

One duty cycle, every `REPORT_INTERVAL` (default 10 min) or motion interrupt:

```mermaid
flowchart TB
    SLEEP["DEEP SLEEP<br/>GPS rail off · SX1276 sleep · AXP rails down<br/>LIS3DH wake-on-motion armed · tamper ADC on RTC timer<br/>target &lt; 2 mA whole-system (measure!)"]
    SLEEP -- "every 10 min<br/>or motion INT" --> WAKE["Wake"]
    WAKE --> TAMPER["Measure tamper loop<br/>GPIO4 excite → GPIO36 ADC<br/>8-sample avg · 100 ms debounce<br/>window 40–260 Ω?"]
    TAMPER --> GFIX{"GPS fix<br/>≤ 90 s?<br/>(warm-start RTC RAM)"}
    GFIX -- "no fix" --> STALE["reuse last position<br/>flag STALE"]
    GFIX -- "fix" --> POS["fresh position"]
    STALE --> PKT
    POS --> PKT["Build LoRa packet ≤ 23 B<br/>position · battery · tamper flags · activity · SEQ"]
    PKT --> TX{"TX OK?<br/>retries 3×"}
    TX -- "no ACK" --> RETRY["retry @ 30 s spacing<br/>(emergency packets)"]
    RETRY --> LATCHQ
    TX -- "ACKed / sent" --> LATCHQ{"Latched<br/>tamper alarm?"}
    LATCHQ -- "yes" --> NOW["send high-priority packet NOW<br/>then every 1 min until base ACKs<br/>(a collar under attack must not go quiet)"]
    NOW --> BACK
    LATCHQ -- "no" --> BACK["Deep sleep"]
    RETRY -- "still failing" --> BACK
    TAMPER -- "outside window →" --> LATCHEV["latch alarm flag<br/>(any brief open, even re-closed)"]
    LATCHEV --> GFIX
```

## 2. Base Station Alarm Pipeline

Every received packet walks the rules engine:

```mermaid
flowchart TB
    RX["LoRa RX interrupt"] --> VAL{"validate MAGIC<br/>+ SEQ dedupe?"}
    VAL -- "invalid / replay" --> DROP["drop (no duplicate alarms)"]
    VAL -- "valid" --> RING["store in ring buffer<br/>(last N positions per collar)"]
    RING --> RULES["evaluate alarm rules"]
    RULES --> GEOF{"GEOFENCE:<br/>point-in-polygon on<br/>latest position?"}
    RULES --> TAMP{"TAMPER:<br/>FLAGS bits from collar?"}
    RULES --> NOMOVE{"NO_MOVE:<br/>ACTIVITY below threshold<br/>continuously 24 h?"}
    RULES --> BATT{"BAT_PERCENT<br/>&lt; 20%?"}
    BATT -- "yes" --> APPLERT["app alert only<br/>(no siren)"]
    GEOF -- "breach" --> ALARM
    TAMP -- "yes" --> ALARM
    NOMOVE -- "yes" --> ALARM
    ALARM["NEW alarm → siren ON<br/>pattern per type · app push queued"]
    ALARM --> ACKQ{"farmer ACK in app?"}
    ACKQ -- "yes" --> SIRENOFF["siren OFF · event archived<br/>re-arm when condition clears<br/>(e.g. 2 consecutive packets back inside fence)"]
    GEOF -- "inside" --> HIST
    TAMP -- "no" --> HIST
    NOMOVE -- "no" --> HIST
    HIST["append history<br/>LittleFS CSV/JSON per collar<br/>~100 KB / collar / month"]
    HIST --> PUSH["WebSocket push → app<br/>(live positions + alarms)"]
    PUSH --> WDOGB["task watchdog feeds<br/>(LoRa RX + app server loops)"]
    SIRENOFF --> WDOGB
    APPLERT --> HIST
```

**Siren patterns:** geofence = 3 short / 2-min pause repeat · tamper = continuous · no-movement = 10 s every 5 min. Auto re-arm.

## 3. App Session (farmer)

```mermaid
flowchart LR
    CONN["connect<br/>WiFi AP SmartCollar-Base<br/>(or USB Web Serial)"] --> OPEN["open PWA<br/>smartcollar.local<br/>(served from base, cached offline)"]
    OPEN --> MAP["map view — Leaflet<br/>cached OSM tiles z14–z17<br/>live positions + history trail"]
    MAP --> ALARMS["alarm feed + banner<br/>service-worker push (persistent)"]
    ALARMS --> ACK["ACK an alarm<br/>→ siren silences, event archived"]
    MAP --> EDIT["geofence editor<br/>draw/edit polygon, ≤ 16 vertices/paddock"]
    EDIT --> PUSH["save → base pushes polygon<br/>to collars at next wake window"]
    PUSH --> MAP
```

## 4. Geofence Push-Down Path

```mermaid
flowchart LR
    DRAW["farmer draws polygon<br/>in app"] --> SAVE["save over WebSocket<br/>to base station"]
    SAVE --> DAILY["base broadcasts<br/>daily timeslot sync"]
    DAILY --> WINDOW["collars align wake<br/>windows to the sync"]
    WINDOW --> DL["downlink at wake:<br/>geofence polygon · interval change ·<br/>ACK · clear-tamper-latch (service mode)"]
    DL --> EARLY["collar evaluates polygon on-collar<br/>for early warning (geofence early flag)"]
```

---

State machine source: [FIRMWARE.md](FIRMWARE.md) · alarm rules & hysteresis: [BASE-STATION.md](BASE-STATION.md) · Blocks & signals: [BLOCK-DIAGRAM.md](BLOCK-DIAGRAM.md) · Layers: [SYSTEM-ARCHITECTURE.md](SYSTEM-ARCHITECTURE.md)
