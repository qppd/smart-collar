# Hardware — Collar Electronics & Assembly

Covers the smart collar's electronics (T-Beam + tamper loop + accelerometer), wiring, and step-by-step assembly into the enclosure and strap. Base station hardware is in [BASE-STATION.md](BASE-STATION.md).

---

## 1. Collar Electronics

### LilyGO T-Beam v2.x pin map (used signals)

| Signal | T-Beam pin | Notes |
|---|---|---|
| Tamper loop ADC | GPIO36 (input-only, ADC1_CH0) | Loop sense node via 10 kΩ bias resistor |
| Tamper loop drive | GPIO4 | Excites the loop only during measurement (saves power, defeats corrosion-offset attacks) |
| Accelerometer SDA | GPIO21 (I²C SDA) | LIS3DH at addr 0x19 |
| Accelerometer SCL | GPIO22 (I²C SCL) | |
| Accelerometer INT1 | GPIO39 (input-only) | Wake-on-motion interrupt |
| Service-mode hall sensor | GPIO34 (input-only) | Hold service magnet here 10 s → maintenance window |
| LED status | Onboard / NeoPixel | Fix quality + alarm indication |

> **Version warning:** T-Beam **v1.x uses AXP192**, **v2.x uses AXP2101** — power-control code differs. Confirm your board version (silk or `adb`/serial boot log) before flashing, and set the matching PMU profile in firmware config.

### Wiring table

| # | From | To | Wire | Notes |
|---|---|---|---|---|
| 1 | GPIO36 + 10 kΩ to 3V3 | Loop sense node | Kynar 30 AWG | Sense node junction |
| 2 | Loop far-end | GND via 100 Ω series (at far end) | via loop | Sets the "signature" resistance |
| 3 | Tamper wire rope end A | Sense node | crimp lug | at enclosure gland A |
| 4 | Tamper wire rope end B | GND rail | crimp lug | at enclosure gland B |
| 5 | Reed switch (buckle) | In series into loop, with 220 Ω shunt across it | enameled wire | Distinguishes "buckle open" from "strap cut" by resistance step |
| 6 | LIS3DH VCC/GND/SDA/SCL/INT1 | 3V3/GND/21/22/39 | 6-wire ribbon | Mount flat, axis Z up |
| 7 | Hall sensor (service) | 3V3 + GPIO34 with 10 kΩ pulldown | — | Inside enclosure wall |

### Tamper loop electrical summary

(design rationale in [TAMPER.md](TAMPER.md))

```
GPIO4 ──[10k]──●──[ wire rope + buckle reed + far-end 100Ω ]──●── GND
               └── GPIO36 (ADC)  →  expected window 40–260 Ω
```

- Read ADC with loop excited (GPIO4 high), average 8 samples, debounce 100 ms.
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
