# Tamper Detection on an Adjustable Collar

The collar must be **adjustable** (fits different animals / neck growth) yet must still reliably detect:

1. **Cutting** the strap
2. **Opening/removing** the buckle
3. **Bypassing** the tamper circuit (e.g., jumpering the loop before cutting)

The core challenge: run a **continuous, supervised electrical loop through the adjustable portion** of the collar without interfering with the adjustment mechanism.

---

## Design Principle: Supervised Loop (Normally-Closed + End-of-Line)

A simple "wire broken = alarm" loop is easy to bypass (short the two ends, then cut freely). Instead we use a **supervised loop** — the collar firmware continuously measures loop **resistance**, not just continuity:

```mermaid
flowchart LR
    GPIO["ESP32 GPIO (drive)"] -- "Rbias · 10 kΩ" --> NODE["sense junction"]
    NODE -- "tamper loop wire<br/>+ far-end series resistor" --> GND["GND"]
    NODE -- "voltage at the junction" --> ADC["ADC — loop resistance signature"]
```

- The tamper loop itself has a **known, designed resistance** (e.g., ~50–200 Ω from the wire rope + a small series resistor at the far end).
- The ADC reading gives a unique "signature" for the intact collar.
- **Cut** → loop opens → ADC reads full pull-up → alarm.
- **Bypass (jumper across the strap)** → resistance drops (jumper has ~0 Ω, skipping the series resistor) → alarm.
- Any tampering with the measuring node → reading moves out of the expected window → alarm.

Firmware keeps a valid window **defined relative to the collar's recorded baseline R₀** — the measured intact resistance at build time (target 40–200 Ω incl. the far-end 100 Ω, recorded per collar in the app maintenance record):

| Band | Condition | Verdict |
|---|---|---|
| OK | R₀ ± 30% | normal |
| SHORT | < 40 Ω (fixed floor) | alarm, subtype SHORT — bypass/jumper |
| OPEN | > 2 kΩ (fixed ceiling) | alarm, subtype OPEN — cut/buckle removed |
| between bands | anything else | LOOP_SUSPECT (re-check, telemetry warning) |

Fixed absolute windows (e.g., a blanket 40–260 Ω) only work if every strap measures identically — real loops don't (rope length, crimps, buckle contact vary). Baseline-relative bands absorb that variation. Anything outside the window — including *too low* — is tamper territory. This defeats the classic "short it first, then cut" bypass.

### Why a series resistor at the far end?

The far end of the loop carries a resistor that the attacker cannot skip without cutting the loop. If they jumper near the collar body, they bypass the far-end resistor → reading too low → alarm. If they splice wire in to keep resistance, they must still cut somewhere → momentary open → alarm (firmware latches any brief open event).

---

## Loop Routing on the Adjustable Strap

```mermaid
flowchart LR
    OUT["ENCLOSURE (electronics)<br/>loop out"] -- "tamper wire — wire rope,<br/>runs the FULL working strap length<br/>(adjustable tail included)" --> BUCKLE["BUCKLE — loop crosses via a<br/>break-away link in the frame"]
    BUCKLE -- "returns along the strap" --> IN["ENCLOSURE<br/>loop in"]
```

Requirements:

- The conductive path runs the **entire working length** of the strap — the adjustable tail included — so cutting *anywhere* (including excess tail) breaks the loop.
- The path must **survive repeated adjustment** (sliding through the buckle) without fraying or breaking.

### Recommended loop material

| Material | Pros | Cons |
|---|---|---|
| **Stainless steel wire rope, ~1 mm 7×7** (chosen) | Extremely cut-resistant, survives mud/water, cheap, works with crimp terminals | Stiffer than thread; needs strain relief at ends |
| Conductive thread (stainless) sewn into webbing | Flexible, moves with strap | Can fray/wear through at the buckle over months; harder to terminate |
| Printed conductive traces | Clean look | Not durable enough; breaks under flex |
| FSR / optical path | No exposed metal | Needs power & conditioning; unreliable when muddy |

**Chosen: stainless wire rope in a protective sleeve, zig-zagged along the webbing** inside a fabric channel, anchored at the enclosure with crimped lugs. It acts both as **sensor and as structural reinforcement** against cutting.

---

## Buckle Instrumentation (detect "opened/removed buckle")

Adjusting the collar means opening the buckle — so the loop must cross the buckle:

- **Break-away link at the buckle**: the wire rope crosses the buckle through a deliberate detachable link (small screw-terminal or crimp barrel) anchored to both strap halves. Opening the buckle separates the halves, the link pulls apart, and the **loop opens → OPEN alarm**. No reed switch, no magnet — the physics does the sensing.
- **The tamper wire passes through the buckle frame** — removing the buckle entirely forces a loop break anyway.
- Opening the buckle while powered = alarm; the only "legit" removal path is **service mode**: put the collar into a maintenance window from the app (BLE command, Maintenance screen) before unbuckling, otherwise the siren fires.

> Balancing act: the collar must be removable for charging/service (Phase 1 is battery-powered), but removal must always be a **deliberate, logged action** — never silent.

---

## Firmware Detection Logic

```cpp
// Loop supervision state machine (runs every wake cycle — RTC-timer wake)
// R0 = recorded intact baseline (NVS, set at collar commissioning; e.g. 120 Ω)
enum LoopState { LOOP_OK, LOOP_OPEN, LOOP_SHORT, LOOP_SUSPECT };

LoopState readTamperLoop() {
  // ADC average over 8 samples, debounced 100 ms
  float r = loopResistanceOhms();
  if (r > OPEN_THRESHOLD)   return LOOP_OPEN;    // > 2 kΩ (fixed) → strap cut / buckle removed
  if (r < SHORT_THRESHOLD)  return LOOP_SHORT;   // < 40 Ω (fixed) → bypass attempt / jumper
  if (inOkBand(r, R0))      return LOOP_OK;      // R0 ± 30% → normal
  // brief out-of-band glitch that re-closed within the debounce window:
  if (glitchLatch)          return LOOP_SUSPECT; // telemetry warning — no siren (see below)
  return LOOP_SUSPECT;                           // between bands → suspect, re-check next cycle
}
```

- Any `LOOP_OPEN` or `LOOP_SHORT` → **immediate LoRa tamper alarm packet** (sent with highest priority + retries) and locally latched until cleared via service mode.
- A momentary open that re-closes within the window (adjustment friction) still latches a `LOOP_SUSPECT` event — reported in telemetry as a warning, not a siren trigger (prevents false alarms from strap flex).
- Alarm packets include collar ID + battery + last GPS fix so the farmer can go straight to where the collar *was*.

---

## False-Alarm & Wear Considerations

| Risk | Mitigation |
|---|---|
| Strap flex momentarily opens crimp joints | Crimp + solder dual termination; strain-relief boot at enclosure exit; flex-rated routing (service loop) |
| Corrosion raises loop resistance over months | Stainless rope + gold/crimp contacts; baseline-relative bands absorb drift (re-commission R₀ via service mode if it drifts > 30%); monthly self-test in telemetry |
| Animal scratches/bites the strap | Wire rope under a fabric flap; enclosure corners rounded |
| Water bridge across cut ends (false continuity) | Loop voltage is low; sea/wallow water resistance ≫ window; series resistor dominates measurement |
| Break-away link corrodes loose | Link is crimp + solder, seated inside the buckle channel; monthly telemetry self-test records loop R — re-crimp on drift |

## Test Checklist

- [ ] Cut the strap in 5 places (middle, near buckle, near tail) → every cut alarms
- [ ] Jumper the loop at the enclosure terminals → bypass alarm fires
- [ ] Unbuckle without service mode → alarm
- [ ] Unbuckle after service-mode entry → no alarm, event logged
- [ ] 500 buckle open/close cycles → no false alarms, resistance stays within R₀ ± 30%
- [ ] 24 h in mud + water, then cut → still alarms
- [ ] Resistance drift test after 2 weeks wear → stays within window
