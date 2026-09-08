# Power Strategy — Battery Now, Solar-Ready

Two-phase strategy, driven by the farm's own recommendation:

- **Phase 1 (prototype, this repo):** battery-powered — 18650 Li-ion, aggressive duty cycling, swap-and-charge rotation.
- **Phase 2 (upgrade):** solar trickle charging — small panel on a swappable lid. **The Phase-1 enclosure and electrical layout are already solar-ready** (see [ENCLOSURE.md](ENCLOSURE.md)) so the upgrade is a lid swap, not a redesign.

---

## Phase 1 — Battery Budget (per collar)

Assume the **LilyGO T-Beam v2.x** (ESP32 + NEO-M8N GPS + SX1276 LoRa + AXP2101 PMU) on one protected 18650 (3,500 mAh, ~11.5 Wh).

### Component current draw (order-of-magnitude, datasheet-typical)

| State | Subsystems on | Current | Duration per cycle |
|---|---|---:|---|
| GPS fix acquisition | ESP32 active + GPS | ~90 mA | 30–60 s (TTFF from cold; warm fix ~5–15 s) |
| LoRa TX (1 packet, SF7–SF9) | ESP32 active + radio | ~120 mA | 0.1–0.3 s |
| Accelerometer watch (movement rule) | MPU6050 in accel-only cycle mode | ~10–70 µA | continuous while sleeping |
| Deep sleep between cycles | AXP2101 + RTC + tamper ADC wakeup | ~0.5–2 mA* | rest of interval |

\* T-Beam deep sleep is famously **not** sub-100 µA unless PMU rails are turned off carefully — budget 1.5 mA average and measure the real figure early (see test below).

### Duty cycle scenarios (10-min reporting interval, 1-min GPS window)

Average current ≈ (GPS 60 s × 90 mA + TX 0.3 s × 120 mA + sleep 539 s × 1.5 mA) / 600 s ≈ **~10.5 mA**

| Reporting interval | Est. average current | Runtime on 3,500 mAh |
|---|---:|---:|
| 5 min | ~19 mA | ~7.5 days |
| **10 min (chosen)** | ~10.5 mA | **~2 weeks** |
| 15 min | ~7.5 mA | ~3 weeks |
| 30 min | ~4.7 mA | ~4.5 weeks |

**Design target: ≥ 2 weeks per charge at a 10-minute interval**, with a swap rotation (one charged spare per collar) so animals are never un-collared. The interval is configurable from the app per collar — racehorses get 5 min, calm herds 30 min.

> GPS duty-cycling dominates the budget. Two easy wins: (1) **warm starts** — keep GPS ephemeris in RAM (fix in 5–15 s instead of 30–60 s); (2) **motion-gated fixes** — if the accelerometer says the animal hasn't moved since the last fix, skip GPS and re-send the last known position (movement ≤ threshold), saving up to ~80% of GPS energy.

### The 24-hour no-movement rule and power

The accelerometer must run *while the collar sleeps* — use the **MPU6050 motion-detect interrupt**: accel-only cycle mode (register PWR_MGMT_1 = accel on, gyro + temp off; LP_WAKE_CTRL rate 5 Hz) detects activity at ~10–70 µA and wakes the ESP32 only on movement (MOT_THR / MOT_DUR configured for head-shake / grazing-scale motion, not vibration noise). The collar keeps a rolling activity counter; if the counter hasn't incremented in 24 h, the next LoRa packet is flagged `NO_MOVEMENT` → base station raises the alarm.

**GY-521 module caution (sleeper current):** the common purple GY-521 breakout has a power LED (~1–2 mA) and an LDO (~1 mA quiescent) that never sleep — 2–3 mA that would dominate the whole sleep budget. Fix at assembly: **strip the power LED** and **bypass the LDO** (feed 3V3 straight to VCC), or budget for it. Bare-die MPU6050 on a custom board avoids this entirely. Verify with the µA meter — same rule as the T-Beam rails.

### Measured-first policy

All figures above are estimates. In Week 1, measure for real (see [TESTING.md](TESTING.md)):

1. Actual deep-sleep current of the T-Beam build (USB power meter + inline ammeter; expect surprises)
2. Actual TTFF warm vs cold at the farm (open sky)
3. Actual energy per GPS+TX cycle

Then finalize the reporting interval that gives ≥ 2 weeks.

## Battery Care & Safety

- **Protected cells only** (over-discharge/short protection) — a collar is an unattended device on an animal.
- Low-battery alarm at 20% (app notification), critical at 10% (collar reduces interval to conserve, still must alarm).
- Charge at 0.5 C max on a smart charger; rotate cells FIFO; retire cells at < 80% rated capacity (log capacity per cell ID in the app maintenance record).
- Temperature: Li-ion must never charge below 0 °C or above 45 °C — carabao collar interior can exceed this in direct sun; the charger must be temperature-compensated or charging happens indoors during the swap (Phase 1: **charging happens off-animal, indoors** — simplest and safest).
- Cell holder with tabs soldered (not spring contacts if vibration is severe — springs walk off; crimped tab + holder hybrid chosen).

## Phase 2 — Solar Sizing

Sizing logic (10-min interval, ~10.5 mA avg @ 3.7 V ≈ **~39 mW** continuous):

| Requirement | Value |
|---|---:|
| Daily energy need | ~0.93 Wh |
| PH sun (conservative effective) | 3 h/day full-sun equivalent |
| Panel needed (with 70% path efficiency incl. shading/dirt/charge loss) | ~0.45 W → **spec a 0.5–1 W panel** |

So a 0.5 W panel *in perfect conditions* breaks even; **1 W with partial-shading tolerance is the safe spec**. Target: battery as buffer, solar as trickle — the collar rides through nights and rainy days on the 18650, and the panel tops it up during grazing. Rainy-season expectations: solar extends the swap interval from 2 weeks toward 1–2 months rather than making it infinite.

Electrical chain: panel → **CN3065** (solar Li-ion charger, MPPT-ish, load sharing) → 18650 → T-Beam. The T-Beam v2.x charges 18650 via its AXP2101 USB path — bypass that for solar (USB charger stays for bench charging) and feed the solar charge module directly to the cell terminals, with the T-Beam drawing from the cell as normal.

## Solar Roadmap Checklist

- [ ] Verify Phase-1 enclosure accepts the solar lid with no case redesign
- [ ] Bench-test CN3065 + 1 W panel → 18650 charge profile in sun/window light
- [ ] Partial shading test (animal's head blocks half the panel) — charge current still positive?
- [ ] Lid gasket survives 50 lid-swaps (service simulation)
- [ ] 2-week field trial on one animal → log battery % vs sun exposure
