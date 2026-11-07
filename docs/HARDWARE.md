# Hardware — Collar Electronics & Assembly

Covers the smart collar's electronics (ESP32 DevKit + NEO-M8N GPS + SX1276 LoRa + tamper loop + accelerometer), wiring, and step-by-step assembly into the enclosure and strap. Base station hardware is in [BASE-STATION.md](BASE-STATION.md).

Full migration details from the original T-Beam v2.x design are in [MIGRATION-SUMMARY.md](../MIGRATION-SUMMARY.md).

---

## 1. Collar Electronics

### ESP32 DevKit 38-pin pin map (used signals)

| Signal | ESP32 pin | Notes |
|---|---|---|
| Tamper loop ADC | GPIO36 (input-only, ADC1_CH0) | Loop sense node via 10 kΩ bias resistor |
| Tamper loop drive | GPIO25 (GPIO-capable, free on header) | Excites the loop only during measurement (saves power, defeats corrosion-offset attacks) |
| Accelerometer SDA | GPIO21 (I²C SDA) | MPU6050 (GY-521) at addr 0x68 (AD0 low) |
| Accelerometer SCL | GPIO22 (I²C SCL) | I²C bus: MPU6050 (0x68) — no address clash |
| GPS UART TX (to M8N RX) | GPIO16 (UART2_TX) | Connected to NEO-M8N RX via 1 kΩ |
| GPS UART RX (from M8N TX) | GPIO17 (UART2_RX) | Connected to NEO-M8N TX |
| Status LED | GPIO2 or GPIO4 (board-dependent) | Fix quality + alarm indication; verify silk screen |

### GPS Module — u-blox NEO-M8N breakout (UART2)

| Neo-M8N Pin | ESP32 DevKit Pin | Wire | Notes |
|---|---|---|---|
| VCC | 3.3 V | Red | Internal LDO may need ≥ 5 V input if using raw 18650 rail — prefer feeding 5 V from buck if module regulator drops out below ~3.5 V |
| GND | GND | Black | Common ground |
| TX | GPIO16 (UART2_RX) | Green | Bi-directional level shift via 1 kΩ if needed |
| RX | GPIO17 (UART2_TX) via 1 kΩ | Blue | 1 kΩ limits current in case of logic mismatch; both sides are 3.3 V tolerant |
| PPS | *unconnected* | — | Not used (GPS fix status checked via NMEA) |
| BAT | *unconnected* | — | Not used; powered externally |

### LoRa Module — Ai-Thinker Ra-02 (SX1276, 915 MHz) via VSPI

| Ra-02 Pin | ESP32 DevKit Pin | Purpose |
|---|---|---|
| VCC | 3.3 V **only** | 5 V destroys the SX1276 |
| GND | GND | Common ground |
| SCK | GPIO18 | VSPI SCK |
| MISO | GPIO19 | VSPI MISO |
| MOSI | GPIO23 | VSPI MOSI |
| NSS (CS) | GPIO5 | VSPI chip select |
| RST | GPIO14 | RadioLib reset |
| DIO0 | GPIO26 | TX/RX-done interrupt (RadioLib IRQ) |
| DIO1–DIO5 | *unconnected* | Not needed for basic operation |

> **Decoupling:** add 100 µF + 100 nF ceramic capacitors close to the Ra-02 VCC/GND pins. TX bursts draw ~120 mA — a weak 3.3 V rail will brown out without adequate bulk capacitance. Never transmit without the antenna attached.

### Tamper Loop Wiring

| # | From | To | Wire | Notes |
|---|---|---|---|---|
| 1 | GPIO25 (drive) via 10 kΩ | Sense node | Kynar 30 AWG | Sense node junction — GPIO36 (ADC) taps the same node |
| 2 | Loop far-end | GND via 100 Ω series (at far end) | via loop | Sets the "signature" resistance — commissioning records R₀ and firmware bands are relative to it (see [TAMPER.md](TAMPER.md)) |
| 3 | Tamper wire rope end A | Sense node | crimp lug | at enclosure gland A |
| 4 | Tamper wire rope end B | GND rail | crimp lug | at enclosure gland B |
| 5 | MPU6050 (GY-521) VCC/GND/SDA/SCL | 3V3/GND/21/22 | 4-wire ribbon | Mount flat, axis Z up — strip PWR LED + bypass LDO (see POWER.md). Motion flag **polled each wake** — no INT wire |

### I²C Bus Summary

Only one I²C device is on the bus now (no AXP PMU):

| Address | Device | Notes |
|---|---|---|
| 0x68 | MPU6050 (GY-521) | AD0 pinned to GND |

### Power Architecture (post-migration)

The integrated AXP2101 PMU on the T-Beam v2.x provided per-rail shutdown and an accurate fuel gauge. The discrete build replaces this with:

- **TP4056 charging module** (micro-USB input, constant-current/constant-voltage) for bench/USB charging. The cell is charged off-animal in the LiitoKala Lii-500 smart charger as before — TP4056 is a fallback for quick top-ups only.
- **AP2112-3.3 SMD LDO** (800 mA, ~1 µA quiescent) for the main 3.3 V rail powering the ESP32, sensors, and LoRa.
- **GPIO-gated rails for deep sleep:** Two N-channel MOSFETs (SI2301 or AO3401) controlled by GPIO13 (GPS rail) and GPIO15 (LoRa rail) shut down the external modules before sleep. The ESP32's own 3.3 V rail stays live (needed for RTC and wake).
- **No PMU fuel gauge.** Battery percentage comes from direct ADC reading of the cell voltage through a resistive divider (GPIO35 or any available ADC1 pin). Accuracy: ±5% across the 3.0–4.2 V range.

### Deep-Sleep Current Target

Target: **< 2 mA whole-system sleep**, ideally **< 500 µA**.

With GPIO-gated rail cutoff:
- ESP32 deep sleep: ~10 mA base (RTC awake) → actual ~0.1 mA when only RTC runs
- GPS rail OFF via GPIO13 → ~0 µA
- LoRa rail OFF via GPIO15 → ~0 µA
- MPU6050 running in accel-only cycle mode → ~10–70 µA
- AP2112 quiescent → ~1 µA

**Measure real current with a µA meter during Week 1 bring-up** (see [TESTING.md](TESTING.md) B3).

## 2. Assembly Order

### Step 1 — Bench bring-up (before any waterproofing)

1. Flash firmware skeleton (see [FIRMWARE.md](FIRMWARE.md)); verify console output on USB-serial at 115200 baud.
2. Verify GPS fix outdoors (first fix can take 2–5 min cold — stand in the open, not indoors). Confirm NMEA stream on `Serial2`.
3. Verify LoRa TX to a second LoRa node (base-station bench rig): packets arrive, RSSI reasonable (−30 to −60 dBm across the room is normal).
4. Wire the accelerometer (4 wires — VCC/GND/SDA/SCL); run the accel cycle-mode config (motion flag polled each wake); confirm sleep current with a USB meter.
5. Wire the tamper loop on the bench (alligator clips first); confirm cut/bypass/buckle detection logic before committing anything to the strap.

### Step 2 — Strap preparation

1. Cut webbing to length (neck circumference + 15 cm adjustment margin; typical carabao neck 90–130 cm → **130–160 cm strap**).
2. Sew/attach a **fabric channel** along the strap center-line for the wire rope, zig-zag route (allows strap stretch without loading the rope).
3. Thread the wire rope; crimp lug terminals at both ends; **measure and record** the loop resistance (target 40–200 Ω total incl. far-end 100 Ω — this becomes **R₀**, the baseline the tamper bands are relative to); seal each crimp with a small T7000 dab against corrosion.
4. Attach the buckle and install the **break-away link** (screw-terminal or crimp barrel) so the loop crosses the buckle through it; verify with a multimeter that unbuckling separates the link (loop → open) and buckling restores continuity.
5. Burn/seal webbing cut ends (hot knife or lighter edge-melt) so they never fray.

### Step 3 — Enclosure build

1. Drill lid holes: 1× bulkhead SMA (LoRa, top-rear), 2× PG7 glands (loop pass-through + GPS u.FL future), vent patch location.
2. Mount GPS patch under the lid's RF window (see [ENCLOSURE.md](ENCLOSURE.md)); u.FL to NEO-M8N, no sharp bends (u.FL cables break at ~10 cycles — route once, final).
3. Mount the ESP32 DevKit board on standoffs inside the enclosure; connect GPS u.FL and LoRa whip via bulkhead SMA.
4. Conformal-coat the ESP32 DevKit PCB (except connectors and antenna ports); install with rubber grommets to the strap slots.
5. Terminate the tamper loop at the internal terminal block; label glands A/B to match loop ends.
6. Battery in holder (tab-soldered), desiccant sachet, torque lid evenly; install vent patch.
7. Run T7000 beads: gland threads/locknuts, gasket seat, loop lug entries — then **wait 24–48 h full cure** before immersion testing.

### Step 4 — Seal test & strap-in

1. Immersion test **before** electronics go in (dummy weight), then again fully assembled **after T7000 has fully cured (≥ 48 h)** — IP67 (1 m / 30 min).
2. Slide enclosure onto strap through its slots with rubber grommets; add keeper blocks; fit padding underside.
3. Full function test on the bench: 24 h soak with a report every 10 min → log battery drop, GPS success rate, loop readings.

### Step 5 — Fit on animal

Fit two fingers of slack (no more — a tight strap causes sores; loose = hoof catch). Confirm antennas point up when the animal grazes head-down (this is the natural rest position — check with the head down!).

## 3. Verification Checklist

- [ ] GPS fix outdoors < 90 s cold (GPS_TIMEOUT; typical 30–60 s) / < 15 s warm
- [ ] LoRa packet round-trip to base rig at 100 m line-of-sight
- [ ] Tamper: cut / bypass / unbuckle all alarm (unbuckle reads as OPEN)
- [ ] Sleep current measured and logged (µA-level reading taken)
- [ ] Immersion passed twice (dummy + assembled)
- [ ] Pull test 100 kg on assembled collar — loop resistance unchanged after
- [ ] Fit check: two-finger slack, antennas up at graze, no rotation after 1 h observation