# Wiring — Circuit Diagram & Netlist

The complete electrical design lives in one diagram, built in Cirkit Designer:

| Source | What it is |
|---|---|
| ![SmartCollar wiring diagram](../wiring/SmartCollar.png) | Full wiring diagram — collar circuit (solar chain) and base-station circuit (power + siren) |
| [`wiring/SmartCollar.ckt`](../wiring/SmartCollar.ckt) | Editable Cirkit Designer project |
| [Cirkit Designer — SmartCollar project](https://app.cirkitdesigner.com/project/fdca6758-069a-49d6-b99e-c560568bd131) | Online source of truth (open, then edit/export) |

The netlist below is extracted programmatically from the `.ckt` project file — it is the authoritative connection list. Two sub-circuits share the sheet: **Collar** (one of three identical collars) and **Base Station** (single unit).

## 1. Collar sub-circuit

Components: LilyGo T-Beam v2.x, 3× 5 V solar panels, 3× SS14 Schottky diodes, CN3065 solar charge controller, 18650 cell in holder, MPU6050.

### Solar charge chain (Phase-2 solar, per [POWER.md](POWER.md))

| From | To | Purpose |
|---|---|---|
| Solar panel #1 (+) | Net 1 (+ solar bus) | Panel output — 3 panels in parallel |
| Solar panel #2 (+) | Net 1 (+ solar bus) | Panel output |
| Solar panel #3 (+) | Net 1 (+ solar bus) | Panel output |
| SS14 diode (anode) | Net 1 (+ solar bus) | Per-panel blocking diode — keeps shaded panels from sinking current |
| SS14 diode (cathode) | Net 6 (+ solar in) | Orchestrated bus → charge controller |
| Solar panel #1 (−) | Net 0 (GND / solar −) | Common negative |
| Solar panel #2 (−) | Net 0 (GND / solar −) | Common negative |
| Solar panel #3 (−) | Net 0 (GND / solar −) | Common negative |
| CN3065 (− Solar In) | Net 0 (GND / solar −) | Charge controller solar input |
| CN3065 (+ Solar In) | Net 6 (+ solar in) | Fed via the SS14 diodes |
| CN3065 (+ Batt In) | Net 2 (18650 VCC) | Charge output → cell positive |
| CN3065 (− Batt In) | Net 3 (18650 GND) | Charge output → cell negative |

**Connection summary:** panels parallel (+ each through its own SS14) → CN3065 Solar In; CN3065 Batt In/Out → 18650 (+/−). The T-Beam draws from the same cell — the CN3065 charges it directly, bypassing the AXP2101 USB path (USB charger stays for bench charging).

### Sensors / I²C

| From | To | Purpose |
|---|---|---|
| MPU6050 (SDA) | T-Beam (GPIO21 / SDA) | I²C data |
| MPU6050 (SCL) | T-Beam (GPIO22 / SCL) | I²C clock |
| MPU6050 (INT) | T-Beam (GPIO39) | Motion-detect interrupt — activity counting only (never a wake cycle; see [POWER.md](POWER.md)) |
| MPU6050 (VCC) | T-Beam (3.3V) | Sensor supply |
| MPU6050 (GND) | T-Beam (GND) | Common ground |
| MPU6050 (ADO, XDA, XCL) | — (unconnected in diagram) | Address/aux pins — tie ADO to GND for address 0x68 on the real build |

> **Note:** the tamper loop and service-mode reed are drawn in [TAMPER.md](TAMPER.md) / [HARDWARE.md](HARDWARE.md) (GPIO25 loop drive, GPIO36 loop ADC, GPIO13 service reed) — add them on the strap harness during collar assembly, not on this sheet.

## 2. Base-station sub-circuit

Components: AC plug, 12 V 5 A switching PSU (S-60-12), 24/12 V → 5 V buck (Circuitrocks 36V→5V 5A), ESP32 DevKit 38-pin, 1-ch relay, 12 V siren ("pia horn" in the diagram).

| From | To | Purpose |
|---|---|---|
| AC plug (Line) | PSU (220 V + / AC L) | Mains input |
| AC plug (Neutral) | PSU (220 V − / AC N) | Mains input |
| PSU (12 V DC +) | Net 12 (+12 V rail) | Main 12 V rail |
| PSU (GND / DC −) | Net 11 (GND rail) | Main ground |
| Net 12 (+12 V rail) | Relay (COM) | Switched feed for the siren |
| Net 12 (+12 V rail) | Buck (VIN+) | Buck input |
| Net 11 (GND rail) | Buck (VIN−) | Buck return |
| Buck (5V out) | ESP32 (VIN 5V) | Logic supply |
| Buck (GND out) | ESP32 (GND) | Common ground |
| ESP32 (GPIO23) | Relay (IN1) | Siren drive — patterned per alarm type (see [FIRMWARE.md](FIRMWARE.md)) |
| ESP32 (GND) | Relay (GND) | Relay logic ground |
| Relay (NO) | Siren (+) | Siren energizes when GPIO23 drives the relay |
| Siren (−) | Net 11 (GND rail) | Siren return |

**Connection summary:** AC → PSU (12 V 5 A) → two branches: (a) 12 V → relay COM → siren (NO), (b) 12 V → buck → 5 V → ESP32. GPIO23 → relay IN1 switches the siren. NC contact unused (siren off by default).

## 3. Deltas vs. the reference design (HARDWARE.md / BASE-STATION.md)

| Point | Diagram | Reference design | Why it differs / what to do |
|---|---|---|---|
| Base LoRa (Ra-02 SX1278) | Not yet drawn | SPI + RST/DIO pins per HARDWARE.md | Draw next revision; wire per [HARDWARE.md](HARDWARE.md) before bench test |
| DS3231 RTC | Not yet drawn | I²C (SDA 21 / SCL 22 shared bus) on base | Add to the I²C bus; set clock at provisioning ([BASE-STATION.md](BASE-STATION.md)) |
| Tamper loop + service reed | Not on this sheet | GPIO25 drive / GPIO36 ADC / GPIO13 reed | Strap harness — see [TAMPER.md](TAMPER.md) |
| Relay board | IN1 driven directly | Opto-isolated boards also have a JD-VCC jumper | Keep jumper closed for 5 V logic drive; see BOM relay note |
| MPU6050 | GY-521 module | Strip power LED + LDO for sleep budget | Do the LED/LDO strip per [POWER.md](POWER.md) when building |
| Solar chain | Present | Phase-2 upgrade | Build it now on the Phase-1 collar or leave panels off — the wiring is identical |

## 4. Power rails at a glance

| Rail | Source | Feeds | Budget |
|---|---|---|---|
| 12 V | S-60-12 PSU (mains) | Siren (via relay), buck input | 5 A — siren draws > 1 A when sounding ([BASE-STATION.md](BASE-STATION.md)) |
| 5 V | Circuitrocks buck (12 V → 5 V) | ESP32 DevKit VIN, relay module VCC | 5 A module, < 1 A actual |
| 3.3 V | Onboard regulators | Sensors, radio | Per-module |
| Battery | 18650 (3.0–4.2 V) via CN3065 / AXP2101 | Collar T-Beam | 3,500 mAh — duty-cycled per [POWER.md](POWER.md) |
| Solar | 3× 5 V 0.3 W panels → SS14 → CN3065 | 18650 trickle charge | ~0.9 W STC array |

## 5. Build-order notes

1. Wire the **base** first: it is bench-testable immediately (power up, flash, `B2` LoRa link once the Ra-02 is added).
2. Wire the **collar** power chain (18650 + T-Beam) before the solar chain; validate `B3` sleep current with panels disconnected.
3. Add the solar chain last — it must not change the sleep budget (verify `B3` again with panels connected).
4. Commons: every sub-circuit shares its own GND — collar GND is floating (never tied to base), base GND is PSU-referenced.
