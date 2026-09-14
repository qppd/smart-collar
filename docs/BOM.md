# Bill of Materials

> **Design tables** (component specs & roles) + **sourced order lists** with verified Lazada PH listings.
> Sourcing date: **2026-09-13**. Criteria: **≥4.0/5 rating, ≥8 ratings count, visible sold count, non-Overseas seller**; rare parts where no listing meets this fall back to **seller-level quality** (Seller Ratings ≥90%, established store) and are flagged FLAG.
> Sourced from **Lazada only** — Shopee PH blocks anonymous search (login wall), so no Shopee links. Prices are web prices (app/voucher prices may differ).

## Smart Collar (per animal)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| LilyGO T-Beam | v2.x — ESP32, NEO-M8N GPS, **SX1278** LoRa 433 MHz variant (T-Beam also ships an SX1276 868/915 variant — must match the base station), 18650 interface | 1 | Collar core: MCU + positioning + radio | 1,800–2,800 |
| 18650 Li-ion cell | 3,500 mAh — unprotected high-drain acceptable: the AXP2101 PMU provides OV/OC cut-off and cells are charged only in the smart charger, off-animal (see [POWER.md](POWER.md)) | 1 (+1 spare) | Collar power | 250–400 |
| Flexible whip antenna | 433 or 915 MHz, SMA/u.FL, flexible | 1 | LoRa uplink | 100–250 |
| GPS antenna | Passive patch, 25×25 mm, u.FL (usually bundled with T-Beam) | 1 | GPS reception | 0–150 |
| Accelerometer | MPU6050 (GY-521 module, I²C) | 1 | Movement activity / no-movement rule (motion flag polled each wake) | 60–150 |
| Stainless steel wire rope | ~1 mm, 7×7 strand, ~1.5 m | 1 | Tamper loop through the strap | 100–200 |
| Resistors (tamper loop) | 10 kΩ (loop bias at the T-Beam) + 100 Ω (far-end signature) — ¼ W, ±5% carbon film (common pack) | 1 pair | Loop drive bias + anti-bypass signature — see [TAMPER.md](TAMPER.md) | 10–30 (pack) |
| Collar strap | 38–50 mm nylon/polyester webbing, breaking strength ≥ 500 kgf | 1 | The collar itself (adjustable) | 150–300 |
| Heavy-duty buckle | Metal side-release or roller buckle, strap-matched | 1 | Adjustment mechanism (tamper-instrumented) | 80–200 |
| Enclosure | IP67 junction/case, **150×150×70 mm sourced** (or 3D-printed ASA + gasket) | 1 | Electronics housing | 100–250 |
| PG7 cable glands | With O-rings | 2–3 | Sealed antenna/cable pass-throughs | 15–30 ea |
| Sealant | **Suxun T7000 silicone adhesive** (black), 25–50 ml tube | 1–2 | Gland threads, gasket seating, crimp sealing, solar-lid bonding (Phase 2) | 60–150 ea |
| Conformal coating (optional) | Acrylic PCB lacquer spray | 1 | PCB protection vs internal condensation | 150–300 |
| Rubber grommets / dampeners | M3–M4 | 4 | Vibration isolation, strap slots | 50 |

**Collar subtotal: ~₱2,780–5,200** (typical build lands near **~₱3,450–3,950**)

### Collar — sourced order list (Lazada PH, verified 2026-09-13)

| # | Item | Qty to buy | ₱ | Rating | Sold | Seller | Listing |
|---|---|---:|---:|---|---|---|---|
| 1 | FLAG — T-Beam **Supreme** (L76K/u-blox, ESP32-**S3**, SX1262, 433 MHz) — see design flag below | 1 | 4,982 | unrated¹ | — | PC cold store (93%, 2.8K sold) | [buy](https://www.lazada.com.ph/products/pdp-i15586728623.html) · [alt V1.2](https://www.lazada.com.ph/products/pdp-i15595961308.html) |
| 2 | LiitoKala 18650 3,500 mAh 3.7 V (2-pcs pack) | 1 pack (=1+1 spare) | 399 | 4.8/5 (180) | 556 | Power World (98%) | [buy](https://www.lazada.com.ph/products/pdp-i3274268797.html) |
| 3 | FLAG — 433 MHz 5 dBi rubber whip + IPX→SMA pigtail | 1 | 83 | unrated¹ | — | KISSCATI (100%) | [buy](https://www.lazada.com.ph/products/pdp-i15608193046.html) |
| 4 | MPU6050 GY-521 module (I²C) | 1 | 185 | 5.0/5 (17) | 86 | diymore (97%, 6-yr store) | [buy](https://www.lazada.com.ph/products/pdp-i1001684320.html) |
| 6 | 304 SS wire rope 1 mm × 10 m | 1 | 125 | 4.9/5 (14) | 67 | Mago-Toyu (98%) | [buy](https://www.lazada.com.ph/products/pdp-i2903511446.html) |
| 7 | Nylon webbing 38 mm × 5 m (black) | 1 | 95 | 5.0/5 (27) | 149 | P S I Online Store (98%) | [buy](https://www.lazada.com.ph/products/pdp-i2676755461.html) · [alt](https://www.lazada.com.ph/products/pdp-i1139904263.html) |
| 8 | Metal side-release buckle 38 mm | 1 | 133 | 5.0/5 (22) | 69 | MEETEE (96%) | [buy](https://www.lazada.com.ph/products/pdp-i4636183104.html) · [alt YKK-plastic](https://www.lazada.com.ph/products/pdp-i3042222312.html) |
| 9 | IP67 ABS enclosure 150×150×70 mm² | 1 | 110 | 4.9/5 (305) | 3.8K | Allan Headset (97%) | [buy](https://www.lazada.com.ph/products/pdp-i2890742842.html) |
| 10 | IP68 nylon cable glands PG7 (10-pc pack)³ | 1 pack | ~120 | 4.9/5 (407) | 1.9K | Solar Bright (98%) | [buy](https://www.lazada.com.ph/products/pdp-i2686431923.html) · [alt kit](https://www.lazada.com.ph/products/pdp-i15409284123.html) |
| 11 | T7000 black adhesive 50 ml | 1 | 85 | 5.0/5 (68) | 203 | MYDirect (97%) | [buy](https://www.lazada.com.ph/products/pdp-i3516299335.html) |
| 12 | Rubber grommet assortment (70-pc kit) | 1 kit | 232 | 4.9/5 (33) | 138 | GlobalTechMart (97%) | [buy](https://www.lazada.com.ph/products/pdp-i5316174145.html) |
| 13 | Conformal coating, brush-on insulating varnish (optional)⁴ | 1 | 296 | 4.7/5 (15) | 57 | GlobalTechMart (97%) | [buy](https://www.lazada.com.ph/products/pdp-i15444551779.html) |

**Sourced collar total: ~₱6,549** (Supreme path, incl. grommet kit, excl. optional conformal coating) · **~₱4,060** if the classic T-Beam comes from AliExpress instead (see flag) · GPS patch antenna: bundled with the board — confirm in cart.

> **Design flag — T-Beam generation.** The classic **T-Beam v2.x (ESP32-WROOM + SX1278) is not currently listed on Lazada PH** — the only 433 MHz-capable T-Beam available locally is the **T-Beam Supreme (ESP32-S3 + SX1262)**, and an unrated V1.2. Two paths:
> **(a) Design-fidelity (recommended):** order the classic **T-Beam v2.x SX1278 (433 MHz)** from the LilyGO official store on AliExpress (~₱1,800–2,800 landed) — keeps every pin map in [HARDWARE.md](HARDWARE.md) (GPIO36/25/13/39, AXP2101) valid as written.
> **(b) Local/Supreme:** the Supreme is 433 MHz-capable, but it is an **ESP32-S3** — no GPIO36/39 (input-only pins used by our tamper ADC + accel INT), different ADC/touch/RTC domains, SX1262 radio init (BUSY/IRQ handling). Choosing it means a **pin-map + firmware rework** of [HARDWARE.md](HARDWARE.md)/[FIRMWARE.md](FIRMWARE.md). Flagged so nobody orders blind.
>
> ¹ Criteria exception: listing has no product rating yet — accepted on seller-level quality (established stores, high seller ratings, thousands sold by store).
> ² Exact-fit 100×100×70 mm (₱80) is out of stock — bump to 150×150×70; a 150×150 box is chunky on the strap, so also consider re-checking stock or 3D-printing ASA per [ENCLOSURE.md](ENCLOSURE.md).
> ³ **Select the PG7 variant on the PDP** — page defaults to PG21 (₱440). Card price ₱120 is the PG7 10-pack.
> Variant picks at checkout: LiitoKala **2-pcs** variant (page defaults to 6-pcs ₱999) · wire rope **1 mm × 10 m** (card shows 100 m price) · buckle **38 mm** · whip is **RP-SMA plug** — verify it mates the board's SMA port before checkout (pigtail included is IPX→SMA).
> ⁴ Brush-on liquid, not aerosol — actually better for spot-coating connector rows. Note: the LiitoKala cells are a 10 A high-drain (unprotected) class; protected-cell listings didn't qualify. This is now consistent policy (see [POWER.md](POWER.md)): the AXP2101 PMU provides the over-voltage/over-current cut-off an unprotected pack needs, and all main charging happens off-animal in the smart charger (Tools section) — never unattended, rotate cells indoors.

## Base Station (one per farm)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| ESP32 DevKit | ESP32-WROOM-32, 38-pin | 1 | Base station controller | 200–350 |
| LoRa module | Ai-Thinker Ra-02 (SX1278, 433 MHz) or equivalent 915 MHz module — must match collar frequency | 1 | LoRa receiver | 150–300 |
| Base antenna | 433/915 MHz, high-gain (3–5 dBi), with mast clamp | 1 | Maximize coverage | 200–500 |
| Siren | 12V DC, ≥ 110 dB, waterproof | 1 | Audible alarm | 150–350 |
| Relay module | 1-channel, 5V logic, 10A | 1 | Drive the 12V siren from ESP32 | 30–60 |
| Power supply | Enclosed switching PSU, **S-60-12 type: 12 V 5 A 60 W**, universal mains input, screw terminals | 1 | 12 V rail — siren direct + buck input | 250–350 |
| Buck converter | Circuitrocks DC-DC buck, 9–36 V in → 5 V, 5 A (aluminium-cased, screw terminals) | 1 | 5 V rail for ESP32 + LoRa | 100–120 |
| Enclosure | IP65 junction box, medium | 1 | Outdoor protection | 100–250 |
| Mast + mounting hardware | Pole, U-bolts, CAT5/antenna cable as needed | 1 | Antenna at 3–5 m height | 200–500 |

**Base station subtotal: ~₱1,160–2,510**

### Base station — sourced order list (Lazada PH, verified 2026-09-13)

| # | Item | Qty to buy | ₱ | Rating | Sold | Seller | Listing |
|---|---|---:|---:|---|---|---|---|
| 1 | ESP32 DevKit WROOM-32 (**select 38-pin**) | 1 | 385 | 4.9/5 (385) | 2.6K | Makerlab PH (98%, 845.6K store) | [buy](https://www.lazada.com.ph/products/pdp-i229344573.html) |
| 2 | Ai-Thinker Ra-02 **SX1278 433 MHz** (model field verified on PDP) | 1 | 429 | 4.9/5 (31) | 223 | Makerlab PH (98%) | [buy](https://www.lazada.com.ph/products/pdp-i3965590587.html) |
| 3 | 433 MHz 5 dB dipole antenna, SMA, 3 m cable⁵ | 1 | 99 | 4.6/5 (23) | 72 | Jimmy Ya (97%) | [buy](https://www.lazada.com.ph/products/pdp-i3736818609.html) |
| 4 | ES-626 siren 110 dB 12V⁶ | 1 | 146 | 4.7/5 (100) | 614 | Beautec (96%) | [buy](https://www.lazada.com.ph/products/pdp-i2913980824.html) · [alt IP65](https://www.lazada.com.ph/products/pdp-i5181387263.html) |
| 5 | 1-ch 5V relay module⁷ | 1 | 140 | 4.5/5 (9) | 139 | DavWorkx (89%) | [buy](https://www.lazada.com.ph/products/pdp-i8192223.html) · [alt KY-019](https://www.lazada.com.ph/products/pdp-i3675717534.html) |
| 6 | Switching power supply **S-60-12 — 12 V 5 A 60 W** (enclosed, screw terminals; **select the S-60-12 variant**) — powers the 12 V siren directly and feeds the buck | 1 | ~300 | — | — | (see listing) | [buy](https://www.lazada.com.ph/products/switching-power-supply-s-60-12-12v-5a-60w-lrs-50-5-5v-10a-12v-42a-24v-21a-50w-i118492443-s16906981189.html) |
| 7 | Circuitrocks buck / step-down DC-DC **36 V→5 V 5 A** (9–36 V in, screw terminals — replaces the LM2596: 5 A headroom, no trimpot drift, case doubles as heatsink) | 1 | 108 | 4.9/5 (28) | 287 | Circuitrocks (97%, 9-yr store) | [buy](https://www.lazada.com.ph/products/pdp-i299280311-s514886405.html) |
| 8 | ZJW IP65 CCTV junction box (**select 150×150×70**) | 1 | 120 | 5.0/5 (348) | 3.8K | ZJW_MALL (99%) | [buy](https://www.lazada.com.ph/products/pdp-i2199716972.html) |

**Sourced base-station total: ~₱1,720** + mast & mounting hardware (hardware-store items — pole, U-bolts, clamps: ~₱200–500).

> ⁵ Dipole style with 3 m cable — no explicit mast clamp; mount with U-bolt + zip ties. Higher-gain options (8 dBi, 12 dBi) exist at ₱49+ but are unrated.
> ⁶ 110 dB per title; "waterproof" not stated (titled Outdoor Security) — mount under the eave; the alt listing is explicitly IP65.
> ⁷ Meets numeric criteria but is the weakest row (4.5/5, 9 ratings, 89% seller); 10A/opto not stated in title. The KY-019 alt is the standard 10 A/250 VAC opto board — either works; prefer whichever ships with the ESP32 order.

## Total Project Budget

Per-block roll-up (sourced prices, 2026-09-13):

| Block | Cost (₱) |
|---|---:|
| Collar — classic T-Beam path (AliExpress board), each | ~4,060–4,560 |
| Collar — local Supreme path, each | ~6,549 |
| Base station (12 V PSU + 5 A buck — no RTC hardware) | ~1,720 |
| Mast & mounting hardware | ~200–500 |
| Tools & consumables (incl. LiitoKala Lii-500 smart charger) | ~1,050–1,250 |
| **Initial build — 3 collars + 1 base station (classic path)** | **~13,900–15,400** |
| Initial build — 3 collars (Supreme path) + 1 base | ~21,370 |
| Each additional collar beyond the initial 3 | +4,060–4,560 (classic) / +6,549 (Supreme) |
| Phase-2 solar per collar (optional) | +407–500 + DIY lid |

- **Initial build is 3 collars + 1 base station** — the smallest setup that demonstrates herd monitoring (multi-collar dedupe/jitter, app multi-marker map) while one collar can rotate through charging. Field trial in [TESTING.md](TESTING.md) runs on these three.
- The Lii-500 charger is a one-time shared tool (charges 2 cells per cycle — one collar's pair + spare; its capacity-test mode drives the cell-retirement log).
- No UPS / surge-protector line — removed by decision; see [BASE-STATION.md](BASE-STATION.md) for the brownout/lightning rationale.

## Phase 2 — Solar Upgrade (per collar, optional)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| Solar panel | 5 V, 0.3 W epoxy mini panel, ~70×70 mm (sourced; the exact 2×2 cm / 0.05 W class isn't listed locally) | 2–3 | Parallel array on the lid — trickle charge during grazing | 186–279 |
| Schottky diode | 1N5819 / SS14, one per panel (series) | 2–3 | Shading isolation + protects charger input | 10–30 |
| Li charge module | CN3065 mini solar charger (VIN 4.4–6 V, ≤500 mA, micro-USB backup input) | 1 | Safe solar charging of the 18650 | 65–85 |
| Solar lid | Enclosure top carrying the panel array + gasket | 1 | Drop-in replacement lid | 50–150 |

### Solar — sourced order list (Lazada PH, verified 2026-09-13)

| # | Item | Qty to buy | ₱ | Rating | Sold | Seller | Listing |
|---|---|---:|---:|---|---|---|---|
| 1 | Mini solar panel 5 V 0.3 W, epoxy, wired⁸ | 2–3 | 93 ea | 4.8/5 (32) | 179 | MotorFactory (95%) | [buy](https://www.lazada.com.ph/products/pdp-i4141765650.html) |
| 2 | SS14 Schottky diode, SMA DO-214AC (100-pc pack — ₱1.26/pc) | 1 pack | 126 | 5.0/5 (14) | 29 | China Electronic (98%) | [buy](https://www.lazada.com.ph/products/pdp-i3679917267.html) · [alt](https://www.lazada.com.ph/products/pdp-i1836021223.html) |
| 3 | FLAG — CN3065 mini USB solar Li charger board⁹ | 1 | 95 | 5.0/5 (1) | 22 | circuitrocks (97%, 9-yr store, 253K sold) | [buy](https://www.lazada.com.ph/products/pdp-i566666537.html) |


**Sourced solar total: ~₱407 (2 panels) / ~₱500 (3 panels)** + solar lid (DIY per [ENCLOSURE.md](ENCLOSURE.md)).

> ⁸ **Spec deviation note:** exact 20×20 mm / 0.04–0.06 W panels are not listed on Lazada PH; the closest qualifying listing is **0.3 W 5 V** (~2× the area). Physics: 2 × 0.3 W STC × 3 h × 70% ≈ **1.26 Wh/day** — already above the ~0.97 Wh/day need, so **2–3 panels replace the 6–8-array** in [POWER.md](POWER.md) (lid space permitting; CN3065 VIN 4.4–6 V still fine in parallel). If you want the exact 2×2 cm PET panels, Shopee solar sellers carry them (~₱15–40 each) — requires a Shopee account.
> ⁹ Criteria exception (1 rating): no CN3065 listing on Lazada PH met the ≥8-rating bar. Circuitrocks is a long-established Manila electronics store (253K store sales, 97% seller rating, 9-yr store) — accepted on seller-level quality. The TP4056 is **not** a substitute (no solar-voltage regulation — stalls in weak light). Erovoutika's own site is currently a parked domain (checked 2026-09-13) — their ₱85 price is no longer verifiable; the Circuitrocks Lazada listing (₱95) is the live channel.

## Tools & Consumables

- Soldering station, flux, heat-shrink
- Multimeter (continuity + low-resistance range for tamper loop work)
- crimper / wire set, zip ties, Suxun T7000 (sealant — see collar BOM)
- USB-serial adapter (if flashing T-Beam without native USB)
- **Smart 18650 charger — LiitoKala Lii-500** (4-slot, LCD, 0.3–1.0 A, capacity-test mode, ~₱726 — [search](https://www.lazada.com.ph/tag/liitokala-li-500/)): the swap-and-charge rotation in [POWER.md](POWER.md) is built on this
- Immersion test tub + dummy load for battery endurance logging

## Ordering Notes

1. **LoRa frequency must match** between collar and base station — both sourced listings above are **433 MHz** (T-Beam Supreme 433 variant + Ra-02 SX1278 433 MHz, model verified on PDP). Radio variants: **SX1278 = 433 MHz**, SX1276 = 868/915 MHz, SX1262 = dual-band (needs different RadioLib init — see [HARDWARE.md](HARDWARE.md)).
2. Buy **one spare 18650 per collar** — the 2-pcs LiitoKala pack covers this exactly.
3. Check T-Beam version pins before wiring: **v1.2 (AXP192 PMU)** vs **v2.x (AXP2101)** vs **Supreme (ESP32-S3)** differ in pinout and power control — see the design flag in the collar order list.
4. **Store clustering saves shipping:** Makerlab PH covers ESP32 + Ra-02 (the two priciest base-station electronics); ZJW_MALL/Allan cover both enclosures; Circuitrocks can additionally cover the buck converter and CN3065 solar charger in one store. Budget ~₱40 shipping per store (~₱400 across all stores at current mix).
5. **The app needs no extra hardware** — it runs on the farmer's own Android phone/laptop; MapLibre + OpenStreetMap data are downloaded once over any available WiFi during provisioning (₱0, see [APP.md](APP.md)).
6. **Suxun T7000 notes:** allow **24–48 h full cure** before any immersion/wallow test (tack-dry ≠ sealed). It is the joint sealant, **not structural** — strap, buckle, and enclosure attachments stay mechanical (bolts/slots/crimp), never glue-loaded. Buy from phone-repair suppliers (T7000 is phone-screen glue) to avoid refilled tubes.
7. **Variant-selection checklist at checkout:** LiitoKala 2-pcs · ESP32 38-pin · PSU **S-60-12 (12 V 5 A)** · Circuitrocks **36V→5V 5A** buck (replaces the LM2596) · ZJW 150×150×70 · Allan 150×150×70 · glands **PG7** (not PG21) · wire rope 1 mm × 10 m · buckle 38 mm · siren single · solar panel 1-pc SKU × qty 2–3.
8. All listings above were **in-stock and web-priced on 2026-09-13**; Lazada listings rot — each row keeps the seller + price so replacements are easy to find if a link 404s.
