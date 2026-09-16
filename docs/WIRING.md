# Wiring — Circuit Diagram & Netlist

The complete electrical design lives in one diagram, built in Cirkit Designer:

| Source | What it is |
|---|---|
| ![SmartCollar wiring diagram](../wiring/SmartCollar.png) | Full wiring diagram — collar circuit (solar chain + tamper loop) and base-station circuit (power + siren) |
| [`wiring/SmartCollar.ckt`](../wiring/SmartCollar.ckt) | Editable Cirkit Designer project |
| [Cirkit Designer — SmartCollar project](https://app.cirkitdesigner.com/project/fdca6758-069a-49d6-b99e-c560568bd131) | Online source of truth (open, then edit/export) |

The netlist below is extracted programmatically from the `.ckt` project file — it is the authoritative connection list. Two sub-circuits share the sheet: **Collar** (one of three identical collars) and **Base Station** (single unit).

## 1. Collar sub-circuit

Components: LilyGo T-Beam v2.x, 3× 5 V solar panels, 3× SS14 Schottky diodes, CN3065 solar charge controller, 18650 cell in holder, MPU6050, stainless wire rope + series resistor (tamper loop, simplified).

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
| MPU6050 (VCC) | T-Beam (3.3V) | Sensor supply |
| MPU6050 (GND) | T-Beam (GND) | Common ground |
| MPU6050 (ADO, XDA, XCL, INT) | — (unconnected) | INT not used — the motion flag is polled over I²C each wake cycle (see [POWER.md](POWER.md)); tie ADO to GND for address 0x68 on the real build |

### Tamper loop (as drawn — simplified)

The netlist shows the rope loop wired as: T-Beam header pin → series resistor → rope end A → rope through the full strap → rope end B → GND rail. **This is a simplified stand-in.** The as-built circuit per [TAMPER.md](TAMPER.md) / [HARDWARE.md](HARDWARE.md) is: `GPIO25 → 10 kΩ → sense node`, then rope end A → strap (break-away link at the buckle) → rope end B → **100 Ω far end** → GND. The sense node is tapped by the **GPIO36 ADC**, which reads the voltage divider to detect strap cuts and bypass attempts. In the CirKit diagram, GPIO36 is routed through a **second T-Beam component (T-Beam(1))** placed solely for its GPIO36 pin — this is not a stray duplicate but the intended connection point for the ADC sense line. The single-resistor drawing cannot distinguish a strap bypass from an intact loop the way the two-resistor reference does — build the reference circuit on the strap harness. A diagram cleanup for the next edit: replace the single resistor with the 10 kΩ / 100 Ω pair and add the GPIO25/GPIO36 junction at the sense node. Service mode is a BLE command (no magnet sensor on the collar).

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
| ESP32 (GPIO17) | Relay (IN1) | Siren drive — patterned per alarm type (see [FIRMWARE.md](FIRMWARE.md)) |
| ESP32 (GND) | Relay (GND) | Relay logic ground |
| Relay (NO) | Siren (+) | Siren energizes when GPIO17 drives the relay |
| Siren (−) | Net 11 (GND rail) | Siren return |

**Connection summary:** AC → PSU (12 V 5 A) → two branches: (a) 12 V → relay COM → siren (NO), (b) 12 V → buck → 5 V → ESP32. GPIO17 → relay IN1 switches the siren. NC contact unused (siren off by default).

### Base LoRa receiver — SX1276 (Ra-02) → ESP32 DevKit (as drawn in the diagram)

| Ra-02 pin | ESP32 38-pin | Purpose |
|---|---|---|
|| VCC | 3V3 | 3.3 V **only** — 5 V destroys the SX1276 |
| GND | GND | Common ground |
| SCK | GPIO18 | VSPI SCK |
| MISO | GPIO19 | VSPI MISO |
| MOSI | GPIO23 | VSPI MOSI |
| NSS (CS) | GPIO5 | VSPI chip select |
| RST | GPIO14 | RadioLib reset |
| DIO0 | GPIO26 | TX/RX-done interrupt (RadioLib IRQ) |

> **Notes:** add 100 µF + 100 nF decoupling at the Ra-02 VCC (TX bursts ~120 mA brown out weak 3.3 V rails). Never transmit without the antenna attached. GPIO23 stays reserved for SPI MOSI — this is why the siren relay moved from GPIO23 to **GPIO17** (GPIO16/17 are free on the 38-pin WROOM board; GPIO17 is UART2-TX by default, unused here).

## 3. Deltas vs. the reference design (HARDWARE.md / BASE-STATION.md)

| Point | Diagram | Reference design | Why it differs / what to do |
|---|---|---|---|
|| Base LoRa (Ra-02 SX1276) | Drawn in the diagram (§2) | SPI (VSPI 18/19/23/5) + RST 14 + DIO0 26 | Bench test `B2` before deployment |
| DS3231 RTC | Not used (decided) | No RTC hardware — phone sets the base clock via BLE `TIME_SET` on connect | Nothing to wire; see [BASE-STATION.md](BASE-STATION.md) |
| Tamper loop | Drawn simplified — one series resistor, no GPIO25/GPIO36 junction; GPIO36 connected via second T-Beam(1) component | GPIO25 → 10 kΩ → node (GPIO36 taps) → rope → 100 Ω far end → GND | Build the reference circuit from [TAMPER.md](TAMPER.md) / [HARDWARE.md](HARDWARE.md); tidy the diagram on the next edit |
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
