# Testing & Evaluation Protocol

Every test below has a pass criterion — the capstone evaluation needs measured results, not vibes. Log all runs in `docs/test-logs/` (create as needed).

---

## 1. Bench Tests (electronics first, before any waterproofing)

| # | Test | Method | Pass |
|---|---|---|---|
| B1 | GPS fix time | 20 fixes: 10 cold (ephemeris cleared), 10 warm | Cold < 90 s, warm < 15 s, open sky |
| B2 | LoRa bench link | Collar→base across 10 m + one wall, 100 packets | 100% received, RSSI > −100 dBm |
| B3 | Sleep current | µA meter in-line, full sleep build | < 2 mA (target < 500 µA) — see [POWER.md](POWER.md) |
| B4 | Tamper cut | Bench loop, cut at 5 positions | Every cut → LOOP_OPEN alarm |
| B5 | Tamper bypass | Jumper across loop terminals | LOOP_SHORT alarm |
| B6 | Buckle detection | Unbuckle w/o service mode | BUCKLE alarm; with service mode: logged, no alarm |
| B7 | Battery gauge | AXP2101 % vs discharge curve | ±10% of actual |
| B8 | 48 h soak run | Full system, 10-min interval, bench range | 0 missed packets; battery drop matches POWER.md model |

## 2. Range & Coverage (site survey)

| # | Test | Method | Pass |
|---|---|---|---|
| R1 | Boundary walk | Carry collar (power bank) to farm corners; log RSSI/SNR at each | ≥ −110 dBm SNR > −10 dB at all boundary points at chosen SF |
| R2 | Dead zone map | Repeat across paddocks; mark weak spots | No paddock interior < −115 dBm (or plan: mast/SF/2nd base) |
| R3 | Wallow test | Submerge collar antenna-deep in pond 30 min while transmitting | Packets still received during immersion |
| R4 | Far-paddock endurance | One collar at farthest point, 24 h | ≥ 95% expected packets |

## 3. Mechanical & Environmental

| # | Test | Method | Pass |
|---|---|---|---|
| M1 | Immersion (IP67) | 1 m, 30 min, electronics installed, ×2 (incl. after lid cycle) | No ingress; full function after |
| M2 | Wallow sim | 24 h mud+water slurry, then hose wash | Function OK; loop window intact |
| M3 | Pull test | 100 kg static, 10 min, assembled collar | No deformation/slippage; loop resistance unchanged |
| M4 | Impact | 2 kg drop from 1 m on enclosure, 5×; strap whack on post 50× | No crack; antenna intact; no false alarms |
| M5 | Flex cycles | 10k bends at gland/boot points | No conductor failure, no seal weep |
| M6 | Thermal | Enclosure interior temp, full noon sun, dummy battery | < 60 °C interior; Li-ion < 45 °C |
| M7 | Fit & comfort | 72 h on animal; check sores, rotation, antenna orientation | No sores; box stays top-of-neck; antennas intact |

## 4. Alarm Scenario Tests (system-level)

| # | Scenario | Method | Pass |
|---|---|---|---|
| S1 | Geofence breach | Walk collar across fence line | Siren ≤ 2 packets after crossing; correct pattern; ACK stops it |
| S2 | GPS jitter false alarm | Stationary collar near fence edge, 24 h | **No** geofence alarm (hysteresis works) |
| S3 | Collar cut on animal | Cut loop on worn collar (staged) | Tamper siren + subtype CUT; app shows last position |
| S4 | Buckle opened | Unbuckle w/o service mode | BUCKLE alarm |
| S5 | No-movement | Collar stationary on bench 24 h+ (accelerometer sees zero) | NO_MOVE alarm at 24 h mark |
| S6 | Low battery | Drain to < 20% | App alert, no siren |
| S7 | Collar silence | Power off collar mid-test | RX-watchdog alert after 3× interval |
| S8 | Multi-alarm priority | Trigger tamper + geofence together | Tamper pattern wins |
| S9 | Brownout recovery | Cut base power 10 s during alarm | Base reboots; alarm state re-evaluated; history intact |
| S10 | Replay attack | Inject duplicate/old packets at base | Deduped; no duplicate alarms |

## 5. Field Trial (final)

**Multi-day herd monitoring** — the capstone evaluation run:

- Duration: **7 days minimum**, all collars on animals, base installed at final mast position
- Log: battery % per day, GPS fix success rate, packet success rate, alarm events (incl. any false alarms — count them explicitly), user actions
- Farmer usability: have the actual farmer operate the app unassisted — task list: install app, view herd, draw geofence, ACK an alarm, export CSV; time each task, note where they hesitate

### Evaluation criteria (capstone-style)

| Criterion | Measure |
|---|---|
| Accuracy | Correct GPS positions vs known points; correct alarm classification |
| Response time | Event → siren latency; event → app latency |
| Reliability | Packet success rate; zero false alarms over trial; IP67 integrity |
| Usability | Farmer task completion + times (SUS-style survey if used) |
| Power endurance | Actual days per charge vs POWER.md model |
| Range | Coverage map vs farm boundary (R1/R2 results) |

## Test Log Template

```
test-logs/YYYY-MM-DD-<name>.md
Date/Time:
Test ID (B#/R#/M#/S#):
Setup:
Result: PASS / FAIL / partial
Measurements:
Notes/failure cause:
```
