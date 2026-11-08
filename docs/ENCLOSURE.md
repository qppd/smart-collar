# Enclosure, Waterproofing & Mechanical Design

The collar lives in one of the harshest consumer-IoT environments possible: **water (carabaos wallow and swim), mud, impact from rubbing/horns/trees, pulling forces of a 400–900 kg animal, and constant flexing for months.** The electronics are only as good as the package around them.

---

## Requirements

| Requirement | Target |
|---|---|
| Water/dust ingress | **IP67** minimum (1 m submersion, 30 min — wallow depth is realistic) |
| Impact | Survives being slammed against trees/ground; UV-stable for outdoor life |
| Pull force | Strap rated ≥ 500 kgf; enclosure mounting must not be the weak point |
| Operating temp | 0–50 °C (incl. direct sun on a black enclosure — prefer light colors) |
| Antennas | LoRa + GPS must work through/around the housing without significant loss |
| Serviceability | Battery swap without destroying the seal (glue-once = dead collar later) |
| Solar-ready | Lid can accept a small panel later **without redesign** |
| Animal comfort | No sharp edges, anti-rotation on the strap, balanced weight |

## Enclosure Options

| Option | Waterproofing | Antenna | Cost | Verdict |
|---|---|---|---|---|
| Off-the-shelf IP67 junction box + PG7 glands | Excellent | Needs glands for external antennas | Low | **Baseline choice** — fast, reliable, ugly but proven |
| 3D-printed ASA/PC + silicone gasket + bolted lid | Good (IP65–67 achievable with care) | Glands or embedded antenna window | Medium | Good for custom form factor; print in **ASA** (UV + heat), never PLA |
| Potting the whole board in epoxy | Excellent | Antennas must be fully external | Low | Phase-2 consideration; makes repair impossible |

**Chosen approach: IP67 box + glands for Phase 1**, custom 3D-printed ASA housing on the solar-ready roadmap.

## Sealing Strategy

1. **Lid seal:** replace the stock foam gasket with **silicone cord or die-cut silicone gasket** seated on a thin bead of **Suxun T7000**, torque screws evenly in cross pattern; add stainless screws (galvanic corrosion with aluminum heatsinks otherwise). The T7000 seals the joint — the screws and gasket compression still do the structural work.
2. **Cable pass-throughs:** everything exits via **PG7 glands with O-rings**, with a drip loop below the gland so water tracks away from the seal; run a **T7000 bead on gland threads and locknuts** at final assembly. Antenna connectors (SMA bulkhead) double as sealed pass-throughs.
3. **Conformal coat the PCB** (acrylic conformal coating at minimum) so even interior condensation doesn't kill the board — temperature swings cause breathing/condensation even in a "sealed" box.
4. **Desiccant sachet** inside (silica gel, replaced at every battery service) absorbs residual moisture.
5. **Pressure equalization:** a small **Gore-style vent patch (IP-rated breathable membrane)** on the lid stops the vacuum/breathing cycle that pulls water past seals over time.
6. **Serviceability:** battery is reached by opening the lid — replace gasket on re-close; log every opening in the app maintenance record.
7. **Cure time:** Suxun T7000 needs **24–48 h to fully cure** before immersion testing — schedule waterproof tests at least two days after assembly.

## Antenna Strategy (the discrete build advantage)

With separate modules, antenna placement is more flexible than the integrated T-Beam design:

| Approach | LoRa link budget | GPS | Mechanical risk | Verdict |
|---|---|---|---|---|
| Stock SMA whips through glands | Best | Good (patch) | Whips break on impact; two more holes in the seal | Reject |
| **Flexible whip LoRa antenna inside a sealed tube/bulkhead SMA** | Good | n/a | Low — flexes on impact | **Chosen (LoRa)** |
| u.FL → external antenna on a **bulkhead SMA on top of the enclosure**, flexible whip above | Good | n/a | Low | **Chosen (LoRa), final layout** |
| GPS ceramic patch inside under the lid (RF-transparent top window) | n/a | Acceptable (~2–4 dB loss) | None | **Chosen (GPS)** |

Design rules:

- **Metal lids kill antennas.** Use a plastic (or 3D-printed) top window region for the GPS patch — the enclosure top above the GPS antenna must be non-metallic and RF-clean (no metal screws directly above).
- LoRa antenna exits **top-rear of the enclosure, bulkhead-mounted, with a flexible whip** — a rigid whip WILL be broken by the animal within days. Add a small rubber strain-relief boot.
- Antenna must sit **above the waterline of typical wallowing** if possible: enclosure rides on the **top/side of the neck**, antennas pointing up.
- The antenna ground plane: a small ground plane improves LoRa range; a piece of copper foil on the lid interior behind the whip, if it doesn't conflict with GPS.

## Collar Form Factor & Mechanical Layout

```mermaid
flowchart TB
    TOP["TOP OF NECK — ANTENNAS UP<br/>GPS window: RF-transparent lid region, no metal above<br/>LoRa whip: bulkhead SMA top-rear + strain-relief boot"]
    BOX["ENCLOSURE · 150×150×70 mm (sourced IP67 box)<br/>ESP32 DevKit + NEO-M8N + Ra-02 · tamper terminal block · desiccant<br/>Phase-2 CN3065 space reserved · PG7 glands + vent patch low edge"]
    PAD["BOTTOM — FACES NECK<br/>rounded corners · closed-cell foam padding (never open-cell)"]
    TOP --- BOX
    BOX --- PAD
    STRAP["STRAP · 38–50 mm webbing, breaking strength ≥ 500 kgf<br/>captive slide-through slots + rubber grommets<br/>anti-rotation keeper blocks / sleeves both sides"] --- BOX
```

- **Strap:** 38–50 mm **nylon/polyester webbing** (same material as tie-out/tow straps), breaking strength ≥ 500 kgf. Wide strap = pressure spread = no neck sores.
- **Attachment:** enclosure **captively slid onto the strap** through molded/printed slots with rubber grommet isolators (anti-vibration + anti-slide) — the enclosure must not spin around the neck or slide to the throat.
- **Anti-rotation:** two rubber "keeper" blocks on the strap either side of the box keep it riding on the top of the neck.
- **Padding:** closed-cell foam on the underside for comfort; open-cell foam would soak water — avoid.
- **Edges:** everything chamfered/rounded; no zip-tie heads against skin.
- **Weight target:** entire collar < 600 g (comfort + avoids behavior change) — re-weigh with the sourced 150×150×70 box.
- **Tamper baseline (build-time):** the loop's *normal* resistance is whatever the built strap measures (target 40–200 Ω incl. far-end 100 Ω) — record it per collar. The firmware alarm bands are defined **relative to that baseline**, not fixed numbers (see [TAMPER.md](TAMPER.md)).
- **Torque/pull test:** hang 100 kg from assembled collar for 10 min — no deformation, no seal breach, loop resistance unchanged.

## Component Dimensions & Fit Check (measured)

All board sizes measured from manufacturer datasheets and verified on arrival — not vendor marketing copy.

| Part | Footprint (mm) | Stack height (mm) | Source |
|---|---|---|---|
| **ESP32 DevKit 38-pin** — PCB | **51 × 26** | 1.6 (board) + ~3 top-side components | Espressif ESP32-DevKitC-32E datasheet |
| — with GPIO headers (both sides) | 51 × 26 | **~8 total** (includes header pins) | Standard DevKit variant |
| NEO-M8N GPS breakout (u.FL) | 25 × 25 (ceramic patch) + ~15 × 10 (module) | ~4 (patch) + ~3 (module) | u-blox NEO-M8 datasheet |
| MPU6050 (GY-521 module) | **20.5 × 15.5** | ~4 incl. header pins | standard GY-521 |
| Ai-Thinker Ra-02 (SX1276) | **17 × 36** | ~3 incl. SMA tab | datasheet |
| CN3065 solar charger (sourced) | **~40 × 20** | ~8 incl. PH2.0 + micro-USB (listing publishes no dims — verify on arrival) | Circuitrocks listing |
| AP2112-3.3 LDO (SOT-23-5) | ~3 × 3 | ~1.5 | Diodes Inc. datasheet |
| SI2301 MOSFET (SOT-23) | ~3 × 1.5 | ~1 | Vishay datasheet |
| TP4056 charge module | **18 × 14** | ~3 | common module |
| 18650 cell (LiitoKala 3500) | Ø18.2–18.5 × 65.1 | — (sits in external holder) | cell spec |
| **Sourced enclosure (ALLAN/ZJW)** | **150 × 150 × 70 external** | **internal ≈ 140 × 140 × 58–60** (walls ~3–5 + lid lip — verify on arrival) | listing variants |

**Fit check vs the sourced 150×150×70 box (internal ≈ 140×140×60):**

- **Floor:** the ESP32 DevKit (26 wide × 51 long) along one wall leaves a **~89 × 140 mm free strip** beside it — NEO-M8N (~25×25), Ra-02 (17×36), GY-521 (20.5×15.5), CN3065 (40×20), tamper terminal block and desiccant all fit on that strip with room to spare; >50% of the floor stays free for Phase-2 solar wiring.
- **Height:** standoffs ~5 + DevKit board stack 8 + top components ~3 ≈ **16 mm**; lid hardware (GPS patch ~4 + u.FL + SMA bulkhead nut) consumes ~8 mm → **~24 mm of ~60 mm used, ~36 mm spare.**
- **The 150×150 size remains appropriate** — generous internal volume with the smaller DevKit board, allowing clean routing and future expansion.
- Weight note: re-weigh the assembled collar against the < 600 g target — the 150×150×70 ABS box alone is ~200–300 g.

## Solar-Ready Design (Phase 2 prep)

The farm recommends solar collars (sample video in `references/`). Phase 1 must not paint us into a corner:

1. **Lid is a separate, swappable part** — the Phase-2 lid carries a **parallel array of 2–3× 5 V 0.3 W panels** (the sourced Lazada units, ~70×70 mm epoxy — see [BOM.md](BOM.md); the originally sketched 6–8× 2×2 cm class isn't locally listed) bonded to the flat top with **Suxun T7000** (electronics-safe, non-corrosive cure — never use acidic-cure RTV near the board). Two panels (~145×75 mm combined) fit the 150×150 mm lid flanking the SMA bulkhead; a third goes forward of it. Round/protect panel corners — no snag points on branches.
2. **Wiring pre-provisioned:** a 2-pin sealed connector (JST with silicone gasket) from the lid area to the charge module space is already routed/terminated in Phase 1, capped with a dummy plug. Phase 2: a single 2-wire harness through the reserved PG7 gland with a drip loop below the gland. Electrical wiring details (Schottky diodes, CN3065, parallel topology): [POWER.md](POWER.md).
3. **Volume reserved** in the enclosure layout for a CN3065 solar charge module — real boards are **~40×20×8 mm** (not 15×15) — no relocation later. The 150×150×70 sourced box has this volume to spare.
4. Panels face up on the **top of the neck** — where the antennas are — so they sit **between** antennas (GPS window forward, solar panels rear) on the enclosure's flat top with antennas at the rear edge. Panel shading by the animal's head is a real loss factor; accept partial shading (per-panel Schottky diodes keep lit panels producing) — MPPT not needed at 0.6 W.
5. See [POWER.md](POWER.md) for the electrical side.

## Environmental Test Checklist

- [ ] **Immersion:** 1 m depth, 30 min (IP67) — no ingress; repeat after 10 lid cycles
- [ ] **Wallow simulation:** 24 h in mud + water slurry, then hose wash | Function OK; loop window intact
- [ ] **Pull test:** 100 kg static, 10 min, assembled collar | No deformation/slippage; loop resistance unchanged
- [ ] **Impact:** drop 2 kg mass onto enclosure from 1 m; strap whacked against post 50× | No crack; antenna intact; no false alarms
- [ ] **Flex:** 10k bend cycles at the gland/antenna boots — no conductor fatigue
- [ ] **Thermal:** enclosure interior < 60 °C under direct noon sun with battery inside (Li-ion must stay < 45 °C — light-colored enclosure if this fails)
- [ ] **Animal trial:** worn 72 h by actual carabao — no sores, no rotation, antennas intact, seal intact