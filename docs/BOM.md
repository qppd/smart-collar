# Bill of Materials

> Prices are **indicative estimates** from typical Lazada/Shopee PH marketplace listings (as of writing) — verify current price, rating, and stock before ordering. Quantities listed are for **one collar + one base station**.

## Smart Collar (per animal)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| LilyGO T-Beam | v2.x — ESP32, NEO-M8N GPS, SX1276 LoRa 433/915 MHz, 18650 interface | 1 | Collar core: MCU + positioning + radio | 1,800–2,800 |
| 18650 Li-ion cell | Protected, 3,500 mAh | 1 (+1 spare) | Collar power | 250–400 |
| Flexible whip antenna | 433 or 915 MHz, SMA/u.FL, flexible | 1 | LoRa uplink | 100–250 |
| GPS antenna | Passive patch, 25×25 mm, u.FL (usually bundled with T-Beam) | 1 | GPS reception | 0–150 |
| Accelerometer | MPU6050 or LIS3DH (I²C) | 1 | Movement activity / no-movement rule | 60–150 |
| Reed switch | Normally-open, glass | 1 | Buckle-open detection | 20–50 |
| Stainless steel wire rope | ~1 mm, 7×7 strand, ~1.5 m | 1 | Tamper loop through the strap | 100–200 |
| *Alternative:* conductive thread | Stainless, ~2–3 m | 1 | Tamper loop sewn into webbing | 100–300 |
| Collar strap | 38–50 mm nylon/polyester webbing, breaking strength ≥ 500 kgf | 1 | The collar itself (adjustable) | 150–300 |
| Heavy-duty buckle | Metal side-release or roller buckle, strap-matched | 1 | Adjustment mechanism (tamper-instrumented) | 80–200 |
| Enclosure | IP67 junction/case, ~100×60×30 mm (or 3D-printed ASA + gasket) | 1 | Electronics housing | 100–250 |
| PG7 cable glands | With O-rings | 2–3 | Sealed antenna/cable pass-throughs | 15–30 ea |
| Sealant | **Suxun T7000 silicone adhesive** (black), 25–50 ml tube | 1–2 | Gland threads, gasket seating, crimp sealing, solar-lid bonding (Phase 2) | 60–150 ea |
| Conformal coating (optional) | Acrylic PCB lacquer spray | 1 | PCB protection vs internal condensation | 150–300 |
| Rubber grommets / dampeners | M3–M4 | 4 | Vibration isolation, strap slots | 50 |

**Collar subtotal: ~₱2,800–5,000** (typical build lands near **~₱3,500–4,000**)

## Base Station (one per farm)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| ESP32 DevKit | ESP32-WROOM-32, 38-pin | 1 | Base station controller | 200–350 |
| LoRa module | Ai-Thinker Ra-02 (SX1278, 433 MHz) or equivalent 915 MHz module — must match collar frequency | 1 | LoRa receiver | 150–300 |
| Base antenna | 433/915 MHz, high-gain (3–5 dBi), with mast clamp | 1 | Maximize coverage | 200–500 |
| Siren | 12V DC, ≥ 110 dB, waterproof | 1 | Audible alarm | 150–350 |
| Relay module | 1-channel, 5V logic, 10A | 1 | Drive the 12V siren from ESP32 | 30–60 |
| Power supply | 12V 2A adapter | 1 | Base station power | 100–200 |
| Buck converter | Mini-360 / LM2596, 12V→5V | 1 | 5V rail for ESP32 + LoRa | 30–60 |
| Enclosure | IP65 junction box, medium | 1 | Outdoor protection | 100–250 |
| Mast + mounting hardware | Pole, U-bolts, CAT5/antenna cable as needed | 1 | Antenna at 3–5 m height | 200–500 |

**Base station subtotal: ~₱1,000–2,600**

## Phase 2 — Solar Upgrade (per collar, optional)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| Solar panel | 5V 0.5–1 W, PET/ETFE laminate, ~55×30 mm window size | 1 | Trickle charge during grazing | 150–300 |
| Li charge module | CN3065 (solar Li-ion charger) or TP4056 with solar input | 1 | Safe solar charging of the 18650 | 30–80 |
| Solar lid | Enclosure top with panel cutout + gasket | 1 | Drop-in replacement lid | 50–150 |

## Tools & Consumables

- Soldering station, flux, heat-shrink
- Multimeter (continuity + low-resistance range for tamper loop work)
- crimper / wire set, zip ties, Suxun T7000 (sealant — see collar BOM)
- USB-serial adapter (if flashing T-Beam without native USB)
- Immersion test tub + dummy load for battery endurance logging

## Ordering Notes

1. **LoRa frequency must match** between collar (T-Beam variant) and base station module — buy the same band (433 MHz or 915 MHz) for both. Confirm which band the T-Beam variant ships with before ordering the base module.
2. Buy **one spare 18650 per collar** — swap-and-charge rotation keeps animals collared full-time.
3. Check T-Beam version pins before wiring: **v1.2 (AXP192 PMU)** vs **v2.x (AXP2101)** differ in GPS/LoRa power control.
4. Prefer vendors with 4★+ ratings and actual stock photos for enclosures and buckles — dimensional mismatches are the most common problem.
5. **Suxun T7000 notes:** allow **24–48 h full cure** before any immersion/wallow test (tack-dry ≠ sealed). It is the joint sealant, **not structural** — strap, buckle, and enclosure attachments stay mechanical (bolts/slots/crimp), never glue-loaded. Buy from phone-repair suppliers (T7000 is phone-screen glue) to avoid refilled tubes.
