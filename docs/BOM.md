# Bill of Materials

## Smart Collar (per animal)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| ESP32 DevKit 38-pin | ESP32-WROOM-32, 38-pin breakout | 1 | Collar MCU | 200–350 |
| NEO-M8N GPS module | u-blox NEO-M8N breakout, u.FL antenna port | 1 | GPS positioning | 600–900 |
| SX1276 LoRa module | Ai-Thinker Ra-02, 915 MHz | 1 | LoRa uplink/downlink | 150–300 |
| Flexible whip antenna | 915 MHz, SMA/u.FL, flexible | 1 | LoRa antenna | 100–250 |
| GPS patch antenna | Passive ceramic, 25×25 mm, u.FL | 1 | GPS reception | 80–150 |
| Accelerometer | MPU6050 (GY-521 module, I²C) | 1 | Movement activity / no-movement rule | 60–150 |
| AP2112-3.3 LDO | SOT-23-5, 800 mA, 1 µA quiescent | 1 | Main 3.3 V rail | 15–30 |
| SI2301 MOSFET | SOT-23, N-channel (×2 for GPS/LoRa rail gating) | 2 | Deep-sleep rail cutoff | 10–20 (pack) |
| 18650 Li-ion cell | 3,500 mAh — unprotected high-drain acceptable | 1 (+1 spare) | Collar power | 250–400 |
| TP4056 charge module | Micro-USB input, CC/CV, ≤1 A | 1 | Bench USB charging fallback | 30–60 |
| Stainless steel wire rope | ~1 mm, 7×7 strand, ~1.5 m | 1 | Tamper loop through the strap | 100–200 |
| Resistors (tamper loop) | 10 kΩ (loop bias) + 100 Ω (far-end signature) — ¼ W, ±5% | 1 pair | Loop drive bias + anti-bypass signature | 10–30 (pack) |
| Resistor divider (batt) | 100 kΩ + 10 kΩ (3.3 V → ADC range) — ¼ W | 1 pair | Battery voltage sense via GPIO35 | 5–10 |
| Collar strap | 38–50 mm nylon/polyester webbing, breaking strength ≥ 500 kgf | 1 | The collar itself (adjustable) | 150–300 |
| Heavy-duty buckle | Metal side-release or roller buckle, strap-matched | 1 | Adjustment mechanism (tamper-instrumented) | 80–200 |
| Enclosure | IP67 junction/case, **150×150×70 mm** | 1 | Electronics housing | 100–250 |
| PG7 cable glands | With O-rings | 2–3 | Sealed antenna/cable pass-throughs | 15–30 ea |
| Sealant | **Suxun T7000 silicone adhesive** (black), 25–50 ml tube | 1–2 | Gland threads, gasket seating, solar-lid bonding (Phase 2) | 60–150 ea |
| Conformal coating (optional) | Acrylic PCB lacquer spray | 1 | PCB protection vs internal condensation | 150–300 |
| Rubber grommets / dampeners | M3–M4 | 4 | Vibration isolation, strap slots | 50 |
| Terminal block | 2-pos screw terminal (tamper loop wiring) | 1 | Internal loop termination point | 10–20 |
| JST-PH2.0 connector | 2-pin (battery holder to board) | 1 | Detachable battery connection | 10–20 |

**Collar subtotal: ~₱1,865–3,370** (typical build lands near **~₱2,400–2,800**)

### Collar — recommended sellers and search terms (Lazada PH)

Search terms are provided because product pages rotate frequently on Lazada. Filter by seller name when you find listings:

| # | Item | Qty to buy | Est. price (₱) | Seller / Store | Search term for Lazada |
|---|---|---:|---:|---|---|
| 1 | ESP32 DevKit 38-pin WROOM-32 (**select 38-pin variant**) | 1 | 200–350 | Makerlab PH | `esp32 devkit 38 pin ch340g` |
| 2 | Neo-8M GPS module breakout (u.FL) | 1 | 600–900 | Makerlab PH | `neo m8n gps u.fl` |
| 3 | SX1276 LoRa module Ra-02 **915 MHz** | 1 | 150–300 | Makerlab PH, Circuitrocks | `sx1276 lora 915mhz ra-02` |
| 4 | 915 MHz rubber whip antenna + IPX→SMA pigtail | 1 | 100–250 | KISSCATI | `lora whip antenna 915mhz sma` |
| 5 | MPU6050 GY-521 module (I²C) | 1 | 60–150 | diymore | `mpu6050 gy-521` |
| 6 | LiitoKala 18650 3,500 mAh 3.7 V (2-pcs pack) | 1 pack (=1+1 spare) | 250–400 | Power World | `liitokala 18650 3500mah 2pcs` |
| 7 | SS14 Schottky diode (for future solar — Phase 2) | 3 | 30–60 | China Electronic, any electronics supplier | `ss14 sma schottky diode` |
| 8 | IP67 ABS enclosure 150×150×70 mm | 1 | 100–250 | Allan Headset, ZJW_MALL | `ip67 enclosure abs 150x150x70` |
| 9 | IP68 nylon cable glands PG7 (10-pc pack) | 1 pack | 100–200 | Solar Bright | `pg7 cable gland ip68 nylon` |
| 10 | T7000 black sealant adhesive 50 ml | 1 | 60–150 | MYDirect | `t7000 black sealant` |
| 11 | Rubber grommet assortment (70-pc kit) | 1 kit | 150–250 | GlobalTechMart | `rubber grommet assortment kit` |
| 12 | 304 SS wire rope 1 mm × 10 m | 1 | 100–200 | Mago-Toyu | `304 stainless wire rope 1mm` |
| 13 | Nylon webbing 38 mm × 5 m (black) | 1 | 95–200 | P S I Online Store | `nylon webbing 38mm black 5m` |
| 14 | Metal side-release buckle 38 mm | 1 | 80–200 | MEETEE | `side release buckle 38mm metal` |
| 15 | Conformal coating, brush-on insulating varnish (optional) | 1 | 150–300 | GlobalTechMart | `conformal coating acrylic pcb` |
| 16 | Si2301 MOSFET SOT-23 (×2 for GPS/LoRa rail control) | 1 pack (50 pcs) | 40–80 | Circuitrocks, AliExpress PH | `si2301 mosfet sot23` |
| 17 | AP2112-3.3 SMD LDO (SOT-23-5, 800 mA, 1 µA quiescent) | 1 (or small pack) | 15–30 | Makerlab PH, AI-VA | `ap2112-3.3 smd` |
| — | **GPS patch antenna** | bundled with board | — | — | Confirm included in cart; no separate purchase needed |

**Sourced collar total: ~₱2,873** (excl. optional conformal coating).

Notes:

- **Antenna verification:** whip is an RP-SMA plug — confirm it mates the board's SMA port before checkout; pigtail included is IPX→SMA.
- The 150×150×70 box size is driven by the NEO-M8N breakout dimensions rather than preference; the smaller ESP32 DevKit (~51 × 26 mm) leaves significantly more internal volume than the original T-Beam layout. Consider re-checking stock or 3D-printing ASA per [ENCLOSURE.md](ENCLOSURE.md).
- PG7 gland variant on some listings defaults to PG21 (more expensive); select PG7 carefully at checkout.
- Brush-on liquid conformal coating is preferable to aerosol for spot-coating connector rows. LiitoKala cells are a 10 A high-drain (unprotected) class; protected-cell listings didn't qualify for this build (see [POWER.md](POWER.md)): main charging happens off-animal in the smart charger — never unattended, rotate cells indoors.
- Two N-channel MOSFETs (SI2301 or equivalent AO3401) gate the GPS and LoRa 3.3 V rails for deep sleep; order a small pack as spares.
- AP2112-3.3 SMD LDO replaces the AXP2101 PMU's 3.3 V rail; low quiescent current (~1 µA) preserves the deep-sleep budget. Verify availability; alternative: LD1117S33TR (higher quiescent ≈ 5 µA).
- **Stock note:** all prices and sellers reflect listings found around November 2026. Lazada inventory rotates — always verify stock and current pricing before ordering.

## Base Station (one per farm)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| ESP32 DevKit | ESP32-WROOM-32, 38-pin | 1 | Base station controller | 200–350 |
| LoRa module | Ai-Thinker Ra-02 (SX1276, 915 MHz) or equivalent — must match collar frequency | 1 | LoRa receiver | 150–300 |
| Base antenna | 915 MHz, high-gain (3–5 dBi), with mast clamp | 1 | Maximize coverage | 200–500 |
| Siren | 12V DC, ≥ 110 dB, waterproof | 1 | Audible alarm | 150–350 |
| Relay module | 1-channel, 5V logic, 10A | 1 | Drive the 12V siren from ESP32 | 30–60 |
| Power supply | Enclosed switching PSU, **S-60-12 type: 12 V 5 A 60 W**, universal mains input, screw terminals | 1 | 12 V rail — siren direct + buck input | 250–350 |
| Buck converter | Circuitrocks DC-DC buck, 9–36 V in → 5 V, 5 A (aluminium-cased, screw terminals) | 1 | 5 V rail for ESP32 + LoRa | 100–120 |
| Enclosure | IP65 junction box, medium | 1 | Outdoor protection | 100–250 |
| Mast + mounting hardware | Pole, U-bolts, CAT5/antenna cable as needed | 1 | Antenna at 3–5 m height | 200–500 |

**Base station subtotal: ~₱1,160–2,510**

### Base station — recommended sellers and search terms (Lazada PH)

| # | Item | Qty to buy | Est. price (₱) | Seller / Store | Search term for Lazada |
|---|---|---:|---:|---|---|
| 1 | ESP32 DevKit WROOM-32 (**select 38-pin**) | 1 | 200–350 | Makerlab PH | `esp32 devkit 38 pin` |
| 2 | Ai-Thinker Ra-02 **SX1276 915 MHz** | 1 | 150–300 | Makerlab PH, Circuitrocks | `ra-02 sx1276 915mhz` |
| 3 | 915 MHz 5 dBi dipole antenna, SMA, 3 m cable | 1 | 200–300 | Jimmy Ya, generic | `915mhz dipole antenna sma 3m` |
| 4 | ES-626 siren 110 dB 12V | 1 | 150–250 | Beautec, ALAZON | `es-626 siren 12v 110db` · [alt IP65] |
| 5 | 1-ch 5V relay module | 1 | 30–80 | DavWorkx, Circuitrocks | `1 channel 5v relay module opto` |
| 6 | Switching power supply **S-60-12 — 12 V 5 A 60 W** (enclosed, screw terminals; **select the S-60-12 variant**) — powers the 12 V siren directly and feeds the buck | 1 | 250–350 | Generic electronics suppliers | `s-60-12 psu 12v 5a` |
| 7 | Circuitrocks buck / step-down DC-DC **36 V→5 V 5 A** (9–36 V in, screw terminals) | 1 | 100–130 | Circuitrocks | `dc dc buck converter 36v to 5v 5a` |
| 8 | ZJW IP65 CCTV junction box (**select 150×150×70**) | 1 | 100–250 | ZJW_MALL | `ip65 junction box cctv 150x150x70` |

**Sourced base-station total: ~₱1,720** + mast & mounting hardware (hardware-store items — pole, U-bolts, clamps: ~₱200–500).

Notes:

- Dipole antenna has a 3 m cable with no explicit mast clamp; mount with U-bolt + zip ties. Higher-gain options (8 dBi, 12 dBi) exist at ₱49+ but are unrated.
- Siren rated 110 dB per title; waterproofing not stated in the product name (titled "Outdoor Security") — mount under the eave; an explicitly IP65-rated alternative is available on the same listing page.
- Relay module: the cheapest rows in this section have lower ratings. An opto-isolated KY-019 board is listed as an alternative; both work — prefer whichever ships with the ESP32 order.
- **Stock note:** verify active stock before ordering. Sellers may cycle through variants.

## Total Project Budget

Per-block roll-up (estimates, November 2026):

| Block | Cost (₱) |
|---|---:|
| Collar — discrete ESP32 build, each | ~2,873–3,370 |
| Base station (12 V PSU + 5 A buck — no RTC hardware) | ~1,720 |
| Mast & mounting hardware | ~200–500 |
| Tools & consumables (incl. LiitoKala Lii-500 smart charger) | ~1,050–1,250 |
| **Initial build — 3 collars + 1 base station** | **~10,870–12,360** |
| Each additional collar beyond the initial 3 | +2,873–3,370 |
| Phase-2 solar per collar (optional) | +407–500 + DIY lid |

- **Initial build is 3 collars + 1 base station** — the smallest setup that demonstrates herd monitoring (multi-collar dedupe/jitter, app multi-marker map) while one collar can rotate through charging. Field trial in [TESTING.md](TESTING.md) runs on these three.
- The Lii-500 charger is a one-time shared tool (charges 2 cells per cycle — one collar's pair + spare; its capacity-test mode drives the cell-retirement log).
- No UPS / surge-protector line — removed by decision; see [BASE-STATION.md](BASE-STATION.md) for the brownout/lightning rationale.

## Phase 2 — Solar Upgrade (per collar, optional)

| Component | Spec | Qty | Role | Est. price (₱) |
|---|---|---:|---|---:|
| Solar panel | 5 V, 0.3 W epoxy mini panel, ~70×70 mm | 2–3 | Parallel array on the lid — trickle charge during grazing | 186–279 |
| Schottky diode | 1N5819 / SS14, one per panel (series) | 2–3 | Shading isolation + protects charger input | 10–30 |
| Li charge module | CN3065 mini solar charger (VIN 4.4–6 V, ≤500 mA, micro-USB backup input) | 1 | Safe solar charging of the 18650 | 65–85 |
| Solar lid | Enclosure top carrying the panel array + gasket | 1 | Drop-in replacement lid | 50–150 |

### Solar — recommended sellers and search terms (Lazada PH)

| # | Item | Qty to buy | Est. price (₱) | Seller / Store | Search term for Lazada |
|---|---|---:|---:|---|---|
| 1 | Mini solar panel 5 V 0.3 W, epoxy, wired | 2–3 | 90–130 ea | MotorFactory, generic solar sellers | `5v 0.3w solar panel epoxy mini` |
| 2 | SS14 Schottky diode, SMA DO-214AC (100-pc pack) | 1 pack | 100–150 | China Electronic, any electronics supplier | `ss14 sma schottky diode 100pc` |
| 3 | CN3065 mini USB solar Li charger board | 1 | 90–120 | Circuitrocks, AI-VA | `cn3065 usb solar lithium charger` |

**Sourced solar total: ~₱407 (2 panels) / ~₱500 (3 panels)** + solar lid (DIY per [ENCLOSURE.md](ENCLOSURE.md)).

- Exact 20×20 mm / 0.04–0.06 W panels are not commonly listed on Lazada PH; the closest qualifying listing is **0.3 W 5 V** (~2× the area). Physics check: 2 × 0.3 W STC × 3 h × 70% ≈ **1.26 Wh/day** — already above the ~0.97 Wh/day need, so **2–3 panels replace the originally sketched 6–8-array** in [POWER.md](POWER.md) (lid space permitting; CN3065 VIN 4.4–6 V still fine in parallel). If you want the exact 2×2 cm PET panels, Shopee solar sellers carry them (~₱15–40 each) — requires a Shopee account.
- CN3065 availability on Lazada PH varies; Circuitrocks is a long-established Manila electronics store (253K store sales, 97% rating, 9-year store) and a reliable fallback. The TP4056 is **not** a substitute — it lacks solar-voltage regulation and stalls in weak light.

## Tools & Consumables

- Soldering station, flux, heat-shrink
- Multimeter (continuity + low-resistance range for tamper loop work)
- crimper / wire set, zip ties, Suxun T7000 (sealant — see collar BOM)
- USB-serial adapter (if flashing without native USB)
- **Smart 18650 charger — LiitoKala Lii-500** (4-slot, LCD, 0.3–1.0 A, capacity-test mode, ~₱726 — search `liitokala lii-500` on Lazada): the swap-and-charge rotation in [POWER.md](POWER.md) is built on this
- Resistor assortment (covers the tamper-loop 10 kΩ bias + 100 Ω far-end pair + voltage divider — see [TAMPER.md](TAMPER.md))
- Immersion test tub + dummy load for battery endurance logging
- SOP-/SOT-23 soldering iron tip (for AP2112, SI2301 SMD components)

## Ordering Notes

1. **LoRa frequency must match** between collar and base station — both target **915 MHz** (Ra-02 SX1276 915 MHz). Radio variants: **SX1276 = 868/915 MHz**, SX1278 = 433 MHz, SX1262 = dual-band (needs different RadioLib init — see [HARDWARE.md](HARDWARE.md)).
2. Buy **one spare 18650 per collar** — the 2-pcs LiitoKala pack covers this exactly.
3. **Store clustering saves shipping:** Makerlab PH covers ESP32 + Ra-02 + NEO-M8N (the three priciest collar electronics); ZJW_MALL/Allan cover enclosures; Circuitrocks can additionally cover the buck converter and CN3065 solar charger in one store. Budget ~₱40 shipping per store.
4. **The app needs no extra hardware** — it runs on the farmer's own Android phone/laptop; MapLibre + OpenStreetMap data are downloaded once over any available WiFi during provisioning (₱0, see [APP.md](APP.md)).
5. **Suxun T7000 notes:** allow **24–48 h full cure** before any immersion/wallow test (tack-dry ≠ sealed). It is the joint sealant, **not structural** — strap, buckle, and enclosure attachments stay mechanical (bolts/slots/crimp), never glue-loaded. Buy from phone-repair suppliers (T7000 is phone-screen glue) to avoid refilled tubes.
6. **Variant-selection checklist at checkout:** LiitoKala 2-pcs · ESP32 38-pin · PSU **S-60-12 (12 V 5 A)** · Circuitrocks **36V→5V 5A** buck (replaces the LM2596) · ZJW 150×150×70 · Allan 150×150×70 · glands **PG7** (not PG21) · wire rope 1 mm × 10 m · buckle 38 mm · siren single · solar panel 1-pc SKU × qty 2–3.