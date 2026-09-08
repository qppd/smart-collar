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
| Accelerometer INT | GPIO39 (input-only, RTC) | Motion-detect interrupt (wake-on-motion) — free on T-Beam, deep-sleep wake-capable |
| Service-mode hall sensor | GPIO13 (RTC, wake-capable) | Hold service magnet here 10 s → maintenance window. **Not GPIO34** — GPIO34 is the GPS NMEA input on every T-Beam; a hall output there fights the GPS line and kills fixes |
| LED status | Onboard LED, GPIO4 (active-low) | Fix quality + alarm indication — T-Beam v2.x has no NeoPixel; single red LED behind 1 kΩ on GPIO4 |

> **Version warning:** T-Beam **v1.x uses AXP192**, **v2.x uses AXP2101** — power-control code differs. Confirm your board version (silk print or serial boot log) before flashing, and set the matching PMU profile in firmware config.


### T-Beam v2.x onboard pin budget (verified: LilyGO `utilities.h` + Meshtastic `tbeam` variant)

Why these pins — every GPIO below is already committed on-board:

| GPIO | Onboard role (T-Beam v2.x) | Verdict |
|---|---|---|
| 34 | GPS NMEA input (input-only) | taken — hall sensor moved off it |
| 12 | GPS command output (u-blox config) | taken |
| 4 | Status LED (red, active-low, via 1 kΩ) | keep as LED status; loop drive moved off it |
| 35 | AXP2101 PMU IRQ (input-only) | taken |
| 38 | user button | taken |
| 5/18/19/23/26/27/32/33 | LoRa radio (SPI, CS, DIO0/1/2, RST) | taken |
| 21/22 | I²C bus — AXP2101 (0x34) + MPU6050 (0x68) | shared bus, addresses coexist |
| 16/17 | PSRAM (8 MB) | taken — do not touch |
| 0/2/12/15 | boot straps (boot mode / must be low at boot / MTDI / debug-UART at boot) | avoid |
| 13/14/25 | free, strap-free, exposed on the header | **13 → hall · 25 → loop drive** (14 spare) |
| 36/39 | free, input-only, RTC-domain | **36 → loop ADC · 39 → MPU6050 INT** |

Boot-safety of the chosen pins: GPIO13 (MTCK) and GPIO25 carry no strap function, so the hall sensor and loop drive cannot disturb flashing or boot. GPIO12 doubles as the MTDI flash-voltage strap — never repurpose it. GPIO37 is the GPS I²C (DDC) line — leave unused.

> **Radio variant check:** T-Beam v2.x ships as **SX1278 (433 MHz)**, **SX1276 (868/915 MHz)** or **SX1262 (dual-band)**. For the 433 MHz plan buy the **SX1278** variant. On an SX1262 board the SPI pins are identical but init differs (BUSY = GPIO32, IRQ = GPIO33, RadioLib `SX1262` class).


### Wiring table

| # | From | To | Wire | Notes |
|---|---|---|---|---|
| 1 | GPIO25 (drive) via 10 kΩ | Loop sense node | Kynar 30 AWG | Sense node junction — GPIO36 (ADC) taps the same node |
| 2 | Loop far-end | GND via 100 Ω series (at far end) | via loop | Sets the "signature" resistance |
| 3 | Tamper wire rope end A | Sense node | crimp lug | at enclosure gland A |
| 4 | Tamper wire rope end B | GND rail | crimp lug | at enclosure gland B |
| 5 | Reed switch (buckle) | In series into loop, with 220 Ω shunt across it | enameled wire | Distinguishes "buckle open" from "strap cut" by resistance step |
| 6 | MPU6050 (GY-521) VCC/GND/SDA/SCL/INT | 3V3/GND/21/22/39 | 6-wire ribbon | Mount flat, axis Z up — strip PWR LED + bypass LDO (see POWER.md) |
| 7 | Service sensor (reed, or DRV5032DU hall) | GPIO13 with 10 kΩ pull-up to 3V3, other end to GND (no 3V3 rail needed for a passive reed) | — | Inside enclosure wall. Idle HIGH, magnet → LOW (reed closes). 0 µA sleep draw for a reed; 1.8 µA for DRV5032. RTC pin — deep-sleep wake on magnet hold |

### Tamper loop electrical summary

(design rationale in [TAMPER.md](TAMPER.md))

```
GPIO25 ──[10k]──●──[ wire rope + buckle reed + far-end 100Ω ]──●── GND
               └── GPIO36 (ADC)  →  expected window 40–260 Ω
```

- Read ADC with loop excited (GPIO25 high), average 8 samples, debounce 100 ms.
- **> 2 kΩ → OPEN (cut/buckle removed)** · **< 30 Ω → SHORT (bypass)** · else OK.
- Reed shunt 220 Ω: buckle-open shifts reading into a distinct mid step (~sub-window band) so firmware can label the alarm `BUCKLE` vs `CUT`.

## 2. Assembly Order

### Step 1 — Bench bring-up (before any waterproofing)

1. Flash firmware skeleton (see [FIRMWARE.md](FIRMWARE.md)); verify console, GPS fix outdoors (first fix can take 2–5 min cold — stand in the open, not indoors).
2. Verify LoRa TX to a second LoRa node (base-station bench rig): packets arrive, RSSI reasonable (−30 to −60 dBm across the room is normal).
3. Wire the accelerometer; run the wake-on-motion config; confirm sleep current with a USB meter.
4. Wire the tamper loop on the bench (alligator clips first); confirm cut/bypass/buckle detection logic before committing anything to the strap.

### Step 2 — Strap preparation

1. Cut webbing to length (neck circumference + 15 cm adjustment margin; typical carabao neck 90–130 cm → **130–160 cm strap**).
2. Sew/attach a **fabric channel** along the strap center-line for the wire rope, zig-zag route (allows strap stretch without loading the rope).
3. Thread the wire rope; crimp lug terminals at both ends; **measure and record** the loop resistance (target 40–200 Ω total incl. far-end 100 Ω); seal each crimp with a small T7000 dab against corrosion.
4. Attach buckle per its hardware type; embed the magnet in the tongue; fix the reed + shunt on the strap side; verify the reed toggles when buckling/unbuckling (multimeter).
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

- [ ] GPS fix outdoors < 60 s cold / < 15 s warm
- [ ] LoRa packet round-trip to base rig at 100 m line-of-sight
- [ ] Tamper: cut / bypass / buckle all alarm distinctly
- [ ] Sleep current measured and logged (µA-level reading taken)
- [ ] Immersion passed twice (dummy + assembled)
- [ ] Pull test 100 kg on assembled collar — loop resistance unchanged after
- [ ] Fit check: two-finger slack, antennas up at graze, no rotation after 1 h observation
