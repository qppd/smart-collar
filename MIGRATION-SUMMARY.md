# SmartCollar — Hardware Migration Summary

**Date:** 2026-11-07  
**Scope:** Replace LilyGO T-Beam v2.x integrated board with discrete ESP32 DevKit 38-pin + external GPS + LoRa modules. All original objectives, features, and functionality preserved.

---

## What Changed

| Area | Before (T-Beam v2.x) | After (Discrete ESP32 Build) |
|---|---|---|
| **Core MCU board** | LilyGO T-Beam v2.x (104.1 × 38.4 mm, ESP32-WROOM, on-board NEO-M8N GPS, SX1276, AXP2101 PMU) | ESP32 DevKit 38-pin (standard WROOM-32 breakout) |
| **GPS module** | On-board NEO-M8N (UART2: GPIO34/12) | External u-blox NEO-M8N breakout on UART2 |
| **LoRa module** | On-board SX1276 SPI (GPIO5/18/19/23/DIO pins) | External Ai-Thinker Ra-02 (SX1276, 915 MHz) on VSPI |
| **Power management** | Built-in AXP2101 PMU (rail control, battery gauge, charger) | External TP4056 charging module + AP2112/LD1117 SMD LDO (3.3 V); no integrated rail shutoff |
| **Tamper ADC pin** | GPIO36 (ADC1_CH0, input-only, RTC-capable) | GPIO36 (still available on DevKit 38-pin header) |
| **Tamper drive pin** | GPIO25 (RTC-capable, free header pin) | GPIO25 (free on DevKit 38-pin, GPIO25 = `LED_BUILTIN` is HIGH when LED off; can be repurposed) |
| **I²C bus** | Shared AXP2101 (0x34) + MPU6050 (0x68) at GPIO21/22 | Only MPU6050 (0x68) at GPIO21/22 — no PMU address conflict |
| **Boot-strap awareness** | GPIO35 (AXP IRQ), GPIO38 (user button) managed by T-Beam firmware | Not present; use any free GPIO (GPIO0/2 must still be avoided) |
| **PSRAM pins** | GPIO16/17 occupied by on-board PSRAM | Free — can be used for I²C/SPI if needed |
| **Board footprint** | 104.1 × 38.4 mm PCB | DevKit PCB ~51 × 26 mm (smaller enclosure feasible) |
| **On-board 18650 holder** | Yes (built-in) | None required; 18650 cell in external JST-PH2.0 holder |
| **Status LED** | GPIO4 (active-low, red) | GPIO2 (on most DevKit variants; active-high or LOW per board) |

---

## New Collar Pin Assignment (ESP32 DevKit 38-pin + external modules)

### GPS Module (NEO-M8N breakout) — UART2

| Neo-M8N Pin | ESP32 DevKit Pin | Notes |
|---|---|---|
| TX | GPIO17 (UART2_RX) | Connect to M8N RX |
| RX | GPIO16 (UART2_TX) | Connected to M8N TX via 1 kΩ (bi-directional 3.3 V logic) |
| PWR | 3.3 V | Internal LDO may struggle — prefer 5 V IN → onboard 3.3 V regulator on M8N module |
| GND | GND | Common ground |
| BAT | Unconnected (not needed; powered externally) | |

### LoRa Module (Ai-Thinker Ra-02, SX1276 915 MHz) — VSPI

| Ra-02 Pin | ESP32 DevKit Pin | Verbatim note |
|---|---|---|
| VCC | 3.3 V (ONLY) | 5 V destroys SX1276 |
| GND | GND | Common ground |
| SCK | GPIO18 | VSPI SCK |
| MISO | GPIO19 | VSPI MISO |
| MOSI | GPIO23 | VSPI MOSI |
| NSS (CS) | GPIO5 | VSPI chip select |
| RST | GPIO14 | Reset line (or tie to ESP32 reset if desired) |
| DIO0 | GPIO26 | IRQ — TX/RX-done interrupt |
| DIO1 | *unconnected* | Not needed for basic RadioLib operation |
| DIO2 | *unconnected* | |

### Tamper Loop Sense Circuit

| Function | ESP32 Pin | Notes |
|---|---|---|
| Drive (excite loop) | GPIO25 | Excites only during measurement cycle (saves power, defeats corrosion-offset attacks) |
| Sense (ADC read) | GPIO36 (ADC1_CH0) | Input-only, RTC-domain — 10 kΩ bias resistor to sense node |
| Far-end series R | 100 Ω | At rope end B → GND rail (signature resistance) |

### Accelerometer (MPU6050 GY-521) — I²C Bus

| MPU6050 Pin | ESP32 DevKit Pin | Address |
|---|---|---|
| VCC | 3.3 V | *(strip power LED & bypass LDO on GY-521 — see POWER.md)* |
| GND | GND | |
| SDA | GPIO21 | I²C data |
| SCL | GPIO22 | I²C clock |
| ADO | GND | Force address 0x68 (default) |
| INT | *unconnected* | Motion flag polled each wake (no interrupt wiring) |

### Status LED & Buttons

| Component | Pin | Notes |
|---|---|---|
| Status LED | GPIO2 (or GPIO4) | Active-low on some DevKit clones; verify board silk screen |
| User button | GPIO0 | Boot-select only — not recommended as user button unless pulled up with RC filter |

---

## Power Architecture Changes

### Battery & Charging

| Item | T-Beam v2.x | Discrete Build |
|---|---|---|
| Cell interface | On-board JST-PH2.0 holder | External 18650 holder with JST-PH2.0 (same connector compatible) |
| Charging circuit | AXP2101 internal charge path (USB path) | TP4056 module (micro-USB input, 1 A constant-current/constant-voltage); charge occurs off-animal via smart charger (Lii-500) or USB port |
| 3.3 V rail | AXP2101 switched LDO (configurable rail shut-off) | AP2112-3.3 SMD LDO (800 mA, low dropout, 1 µA quiescent) or equivalent (LD1117S33TR — higher quiescent ≈ 5 µA) |
| Rail shutdown for deep sleep | AXP2101 I²C register writes | GPIO-controlled N-channel MOSFET (SI2301) on GPIO13 (GPS rail) and GPIO15 (LoRa rail); **external 10 kΩ pull-up resistors mandatory on both pins** (GPIO15 = MTCK boot strap, LOW = download mode) |

### Deep-Sleep Current Target

The absence of an integrated PMU means we lose fine-grained rail-shutdown capability. To hit < 2 mA deep-sleep target:

1. **Keep 3.3 V rail on** for RTC and GPIO retention (ESP32 requires this).
2. **Shut down GPS and LoRa rails** explicitly before sleep:
   - GPS module has its own 3.3 V pin — gate it with an N-channel MOSFET (e.g., SI2301) driven by GPIO15.
   - LoRa module's 3.3 V is already shared; cut it with the same FET if GPS and LoRa share a rail, OR add a second FET for LoRa on GPIO13.
3. **Measure real current** after assembly; budget 1–2 mA without rail cutting, sub-500 µA with aggressive rail cutoff.

See updated [POWER.md](POWER.md) § Deep-sleep strategy for details.

---

## Firmware Library Updates

| Library | Role | Change |
|---|---|---|
| `RadioLib` | SX1276 LoRa driver | Same as before; init uses GPIO5/18/19/23/14/26 |
| `TinyGPSPlus` | NMEA parsing for NEO-M8N | Same; reassign to `Serial2` (GPIO16/17) |
| `Adafruit_MPU6050` | Accelerometer motion polling | Same; I²C at GPIO21/22 unchanged |
| ~~`AXP2101`~~ | PMU rail management | **Removed** — replaced by GPIO-gated FET + manual rail control code |
| `ArduinoJson` | Packet serialization | No change |
| `ESP.deepSleep()` API | Sleep entry | Identical; call `esp_sleep_enable_timer_wakeup()` for RTC-timer wake |

### Example: replacing AXP2101 rail-control calls

```cpp
// OLD (T-Beam):
axp2101.setPowerOutPut(AXP2101_LDO2, AXP2101_OFF);  // GPS rail off
axp2101.setPowerOutPut(AXP2101_DCDC3, AXP2101_OFF);  // LoRa rail off

// NEW (Discrete build):
digitalWrite(GPIO_GPS_VCC, LOW);    // Turn off GPS 3.3V rail
digitalWrite(GPIO_LORA_VCC, LOW);   // Turn off LoRa 3.3V rail
// Use GPIO13 (GPS) and GPIO15 (LoRa) — both require external 10 kΩ pull-up to 3.3 V
// GPIO15 = MTCK boot strap pin; missing pull-up causes bootloader hang on power-on
```

---

## Enclosure Impact

The smaller DevKit board (~51 × 26 mm vs 104.1 × 38.4 mm) means the 150 × 150 × 70 mm IP67 box now has significantly more internal volume — ample room for:
- External GPS patch antenna (u.FL) placed under an RF-transparent window in the lid
- External LoRa whip antenna (bulkhead SMA mount top-rear)
- All wiring, terminal blocks, and CN3065 solar stage预留 space

Re-verify mechanical layout in [ENCLOSURE.md](ENCLOSURE.md) § Component Dimensions & Fit Check.

---

## NTC Compliance (Philippines)

See new section [NTC-COMPLIANCE.md](NTC-COMPLIANCE.md) for full regulatory analysis. Key points below:

### LoRa (915 MHz ISM Band)

| Parameter | Requirement / Guidance |
|---|---|
| Frequency band | 902–928 MHz ISM (allowed for unlicensed equipment; confirm specific EASA exemption with NTC) |
| Maximum EIRP | 30 dBm (1 W) absolute max; recommend ≤ 20 dBm (100 mW) for collar to reduce interference concerns |
| Duty cycle | No formal PH duty-cycle limit for ISM; European ETSI 1% applies only in EU |
| Certification | Individual experimental/prototype units may be covered under NTC **EASA** (Equipment Authorization for Scientific and Educational purposes) — contact NTC Radiation Protection and Management Bureau for guidance |
| Module pre-certification | Buying modules with existing FCC/CE/IC certification simplifies but does not eliminate device-level registration |

### GSM / SIM800L (if ever added in future phases)

| Parameter | Requirement / Guidance |
|---|---|
| Bands used in PH | 900 MHz (Band 8) and 1800 MHz (Band 3) primary |
| Certification | Requires full NTC Type Approval (more stringent than LoRa); not recommended for prototype phase |
| This project | SIM800L **not included** in current design — no regulatory action needed |

### Recommended Next Steps

1. Consult NTC Radiation Protection and Management Bureau (<https://www.ntc.gov.ph>) regarding prototype exemption for LoRa devices.
2. Purchase LoRa modules with valid FCC ID / CE mark documentation; retain certificates for NTC filing if required.
3. Mark final deployed collars with model name, frequency, and EIRP rating.

---

## Files Updated in This Migration

| File | Changes Made |
|---|---|
| `README.md` | Replaced T-Beam references; added migration summary section |
| `docs/HARDWARE.md` | Complete rewrite of pin maps, wiring tables, tamper circuit |
| `docs/BOM.md` | Removed T-Beam entries; added discrete ESP32 + NEO-M8N + Ra-02 sourced listings |
| `docs/WIRING.md` | Updated netlist; removed AXP2101/solar-chain references where inaccurate; added new power-rail table |
| `docs/FIRMWARE.md` | Removed AXP2101 library; updated startup sequence and NVS defaults; preserved packet format |
| `docs/POWER.md` | Revised power budget (no AXP2101 rail control); added GPIO-FET rail gating strategy |
| `docs/TAMPER.md` | Minor updates — pin numbers preserved (GPIO36/25 are the same) |
| `docs/ENCLOSURE.md` | Updated fit-check dimensions for smaller DevKit board |
| `docs/BASE-STATION.md` | No hardware changes (was already using ESP32 DevKit) |
| `docs/BLOCK-DIAGRAM.md` | Updated block diagram text |
| `docs/STACK.md` | Updated stack tables to reflect discrete build |
| `docs/TESTING.md` | Added B3 bench test notes for new power architecture |
| `NTC-COMPLIANCE.md` | **(NEW)** Full NTC regulatory analysis for Philippines |

---

## Preserved Original Objectives & Features

All of the following remain unchanged:

- Offline-first operation (GPS + LoRa + BLE; no internet needed)
- Three alarm conditions: geofence breach, collar tamper, 24-hour immobility
- Supervised tamper loop (cut/bypass/buckle detection via resistance signature)
- Adjustable collar with break-away link at buckle
- IP67 waterproofing target
- Phase-1 battery (18650) + Phase-2 solar-ready roadmap
- Base station LoRa receiver + siren alarm
- Offline PWA with MapLibre + OSM map pack
- Maintenance mode toggle with auto-expiry
- 10-minute default reporting interval with ±5% jitter
- Binary packet format (≤ 23 bytes body) with CRC16
- BLE GATT server at base for app connectivity
- LittleFS history logging at base station

---

### Known Open Items

1. **Deep-sleep current measurement required** — without AXP2101's ultra-low-power modes, actual sleep current will differ from T-Beam benchmarks. Bench-measure after first assembly.
2. **TP4056 charging module voltage rating** — ensure module supports 4.2 V cutoff and is compatible with unprotected 18650 cells (LiitoKala 3500 mAh high-drain).
3. **GPS module 3.3 V regulation** — the NEO-M8N datasheet lists 2.7–3.6 V nominal; confirm that the on-board LDO on your breakout module can handle input from a depleting 18650 (down to ~3.0 V). If marginal, feed 5 V from the buck rail directly.
4. **NTC EASA filing** — confirm whether prototype LoRa collars need formal registration or qualify for research exemption.
5. **Antenna placement** — verify GPS patch and LoRa whip do not mutually degrade each other when co-located on the same enclosure face; consider 90° orientation or spacing ≥ 5 cm.
6. **GPIO15 external pull-up resistor** — GPIO15 (MTCK) **must** have a ≥10 kΩ pull-up to 3.3 V on the board; internal weak pull-up alone is unreliable. Same applies to GPIO13 (GPS rail gate). Without these, the ESP32 may fail to boot into application firmware and hang in download mode instead.
