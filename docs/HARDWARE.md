# Hardware — Collar Electronics & Assembly

Covers the smart collar's electronics (T-Beam + tamper loop + accelerometer), wiring, and step-by-step assembly into the enclosure and strap. Base station hardware is in [BASE-STATION.md](BASE-STATION.md).

---

## 1. Collar Electronics

### LilyGO T-Beam v2.x pin map (used signals)

| Signal | T-Beam pin | Notes |
|---|---|---|
| Tamper loop ADC | GPIO36 (input-only, ADC1_CH0) | Loop sense node via 10 kΩ bias resistor |
| Tamper loop drive | GPIO25 (RTC-capable, free on header) | Excites the loop only during measurement (saves power, defeats corrosion-offset attacks). **Not GPIO4** — GPIO4 is the onboard status LED; driving it LOW to idle the loop keeps the LED lit (~2 mA) and wrecks the sleep budget |
| Accelerometer SDA | GPIO21 (I²C SDA) | MPU6050 (GY-521) at addr 0x68 (AD0 low) |
| Accelerometer SCL | GPIO22 (I²C SCL) | Shared bus: AXP2101 (0x34) + MPU6050 (0x68) + optional OLED (0x3C) — no address clash |
| LED status | Onboard LED, GPIO4 (active-low) | Fix quality + alarm indication — T-Beam v2.x has no NeoPixel; single red LED behind 1 kΩ on GPIO4 |

> **Version warning:** T-Beam **v1.x uses AXP192**, **v2.x uses AXP2101** — power-control code differs. Confirm your board version (silk print or serial boot log) before flashing, and set the matching PMU profile in firmware config.


### T-Beam v2.x onboard pin budget (from LilyGO's `utilities.h` and the Meshtastic `tbeam` variant)

Every GPIO below is already committed on-board, which is why the free pins matter:

| GPIO | Onboard role (T-Beam v2.x) | Verdict |
|---|---|---|
| 34 | GPS NMEA input (input-only) | taken (GPS) |
| 12 | GPS command output (u-blox config) | taken |
| 4 | Status LED (red, active-low, via 1 kΩ) | keep as LED status; loop drive moved off it |
| 35 | AXP2101 PMU IRQ (input-only) | taken |
| 38 | user button | taken |
| 5/18/19/23/26/27/32/33 | LoRa radio (SPI, CS, DIO0/1/2, RST) | taken |
| 21/22 | I²C bus — AXP2101 (0x34) + MPU6050 (0x68) | shared bus, addresses coexist |
| 16/17 | PSRAM (8 MB) | taken — do not touch |
| 0/2/12/15 | boot straps (boot mode / must be low at boot / MTDI / debug-UART at boot) | avoid |
| 13/14/25 | free, strap-free, exposed on the header | **25 → loop drive** (13/14 spare) |
| 36/39 | free, input-only, RTC-domain | **36 → loop ADC** (39 spare — no accel INT; motion is polled) |

Both chosen pins are boot-safe: GPIO25 carries no strap function, so the loop drive cannot disturb flashing or boot. Two pins to leave alone: GPIO12 doubles as the MTDI flash-voltage strap (never repurpose it), and GPIO37 is the GPS I²C (DDC) line, unused in this design.

> **Radio variant check:** T-Beam v2.x ships as **SX1276 (915 MHz)**, **SX1262 (dual-band)** or older SX1278 (433 MHz). For the 915 MHz plan buy the **SX1276** variant. On an SX1262 board the SPI pins are identical but init differs (BUSY = GPIO32, IRQ = GPIO33, RadioLib `SX1262` class).


### Wiring table

| # | From | To | Wire | Notes |
|---|---|---|---|---|
| 1 | GPIO25 (drive) via 10 kΩ | Loop sense node | Kynar 30 AWG | Sense node junction — GPIO36 (ADC) taps the same node |
| 2 | Loop far-end | GND via 100 Ω series (at far end) | via loop | Sets the "signature" resistance — the commissioning step records R₀ and firmware bands are relative to it (see [TAMPER.md](TAMPER.md)) |
| 3 | Tamper wire rope end A | Sense node | crimp lug | at enclosure gland A |
| 4 | Tamper wire rope end B | GND rail | crimp lug | at enclosure gland B |
| 5 | MPU6050 (GY-521) VCC/GND/SDA/SCL | 3V3/GND/21/22 | 4-wire ribbon | Mount flat, axis Z up — strip PWR LED + bypass LDO (see POWER.md). Motion flag **polled each wake** — no INT wire |

### Tamper loop electrical summary

The tamper loop circuit, classification bands (R₀ ± 30% OK / < 40 Ω SHORT / > 2 kΩ OPEN), and detection rationale are fully documented in [TAMPER.md](TAMPER.md). During assembly, wire the loop per the table above and record the baseline resistance R₀ (target 40–200 Ω incl. far-end 100 Ω) — this becomes the reference for all tamper bands.

## 2. Assembly Order

### Step 1 — Bench bring-up (before any waterproofing)

1. Flash firmware skeleton (see [FIRMWARE.md](FIRMWARE.md)); verify console, GPS fix outdoors (first fix can take 2–5 min cold — stand in the open, not indoors).
2. Verify LoRa TX to a second LoRa node (base-station bench rig): packets arrive, RSSI reasonable (−30 to −60 dBm across the room is normal).
3. Wire the accelerometer (4 wires — VCC/GND/SDA/SCL); run the accel cycle-mode config (motion flag polled each wake); confirm sleep current with a USB meter.
4. Wire the tamper loop on the bench (alligator clips first); confirm cut/bypass/buckle detection logic before committing anything to the strap.

### Step 2 — Strap preparation

1. Cut webbing to length (neck circumference + 15 cm adjustment margin; typical carabao neck 90–130 cm → **130–160 cm strap**).
2. Sew/attach a **fabric channel** along the strap center-line for the wire rope, zig-zag route (allows strap stretch without loading the rope).
3. Thread the wire rope; crimp lug terminals at both ends; **measure and record** the loop resistance (target 40–200 Ω total incl. far-end 100 Ω — this becomes **R₀**, the baseline the tamper bands are relative to); seal each crimp with a small T7000 dab against corrosion.
4. Attach the buckle and install the **break-away link** (screw-terminal or crimp barrel) so the loop crosses the buckle through it; verify with a multimeter that unbuckling separates the link (loop → open) and buckling restores continuity.
5. Burn/seal webbing cut ends (hot knife or lighter edge-melt) so they never fray.

### Step 3 — Enclosure build

1. Drill lid holes: 1× bulkhead SMA (LoRa, top-rear), 2× PG7 glands (loop pass-through + solar future), vent patch location.
2. Mount GPS patch under the lid's RF window (see [ENCLOSURE.md](ENCLOSURE.md)); u.FL to T-Beam, no sharp bends (u.FL cables break at ~10 cycles — route once, final).
3. Conformal-coat the T-Beam (except connectors/antenna); install with rubber grommets to the strap slots.
4. Terminate the loop at the internal terminal block; label glands A/B to match loop ends.
5. Battery in holder (tab-soldered), desiccant sachet, torque lid evenly; install vent patch.
6. Run T7000 beads: gland threads/locknuts, gasket seat, loop lug entries — then **wait 24–48 h full cure** before immersion testing.

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
