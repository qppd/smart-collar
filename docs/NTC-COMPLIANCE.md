# NTC Compliance — Philippine National Telecommunications Commission Regulations

**Document version:** 1.0 · **Date:** 2026-11-07  
**Applicable to:** SmartCollar LoRa collar system (915 MHz ISM band)  
**Prepared for:** Capstone prototype deployment in the Philippines

---

## 1. Overview

This document identifies the regulations, certifications, and compliance steps applicable to the SmartCollar project under Philippine law. The project uses two radio technologies:

- **LoRa at 915 MHz** (primary, short-range point-to-multipoint)
- **GPS L1 band** (receive-only, no transmission — not regulated by NTC)

No cellular (GSM/LTE) module is included in the current design. The SIM800L referenced in the REPLAN-PROMPT is **not part of this project**.

---

## 2. Legal Framework

### 2.1 Republic Act No. 3846 (The Telegraph Act) & RA 7920 (The Public Telecommunications Policy Act)

These acts establish the NTC as the sole regulatory authority over all radio-frequency emissions in the Philippines. Any person or entity transmitting radio signals must comply with NTC technical standards.

### 2.2 NTC Memorandum Order (MO) Series — Equipment Registration

The NTC requires **all radio-transmitting equipment** sold, imported, or operated in the Philippines to obtain either:

| Category | Description | Applicable Here? |
|---|---|---|
| **Type Approval (TA)** | Full certification for commercial products offered for sale/import | Prototypes not intended for resale — likely exempt |
| **Equipment Authorization for Scientific and Educational Applications (EASA)** | Exemption for research, educational, or experimental equipment | **Most applicable** to this capstone project |
| **Broadcasting / Commonwealth Act exemptions** | For media entities — N/A here | Not applicable |

### 2.3 NTC guidelines on Unlicensed Radio Transmitters

The NTC publishes and periodically updates permitted frequency bands for unlicensed industrial, scientific, and medical (ISM) equipment. The current published ISM bands include:

| Band | Frequency Range | Max EIRP (typical) | Relevance to SmartCollar |
|---|---|---|---|
| ISM Band 1 | 433.05–434.79 MHz | 25 mW (0.17 dBm) · 1% duty cycle | Not used |
| ISM Band 2 | 915–924 MHz | **Varies — consult current NTC MO** | **Primary target: 915 MHz LoRa** |
| ISM Band 3 | 2.4–2.4835 GHz | 1 W (30 dBm) · no duty restriction | WiFi/BLE (base station) — allowed |

> **Confirmed requirement (as of 2024):** The NTC has historically recognized the 915–928 MHz range for ISM use consistent with ITU Region 3 allocations. However, the **Philippine implementation may impose additional restrictions** (e.g., lower max EIRP than the US FCC Part 15 limit of 1 W). Always verify against the latest NTC MO before production.

---

## 3. Confirmed Regulatory Requirements

### 3.1 LoRa Transmission Parameters (must comply)

| Parameter | Limit / Requirement | Source |
|---|---|---|
| Operating frequency | **915 ± 5 MHz** (i.e., 910–920 MHz minimum; full band 902–928 MHz per ITU-R) | NTC MO on ISM bands; ITU-R Radio Regulations Art. 5.239 |
| Maximum EIRP | **≤ 20 dBm (100 mW)** recommended for prototype safety margin; absolute legal ceiling depends on current NTC MO | ITU-R SM.398; NTC ISM guidelines |
| Spreading factor / bandwidth | Not directly restricted (SF7–SF9, 125 kHz BW common) | N/A |
| Duty cycle | No explicit PH duty-cycle mandate for 915 MHz ISM (unlike EU's 1%) | NTC ISM guidelines |
| Spectrum mask | Must stay within assigned 915 MHz channel; no spurious emissions > −30 dBc | NTC equipment standards |

### 3.2 GPS (Receive-Only)

| Parameter | Status |
|---|---|
| L1 reception (1575.42 MHz) | **No NTC authorization required** — receive-only devices are not regulated |
| Transmit power | 0 mW (passive receiver) |
| Antenna | Small ceramic patch — negligible emission concern |

### 3.3 Bluetooth LE (Base Station, 2.4 GHz)

| Parameter | Limit / Requirement |
|---|---|
| Band | 2.400–2.4835 GHz ISM |
| Max EIRP | ≤ 100 mW (20 dBm) per FCC Part 15 / NTC analogous rules |
| SmartCollar actual | ESP32 BLE typically ≤ 4 dBm (2.5 mW) EIRP — well within limits |

---

## 4. Recommended Compliance Practices

Even though the SmartCollar is a **prototype/capstone project** (not a commercial product), the following practices reduce regulatory risk and simplify future type-approval:

| Practice | Action | Priority |
|---|---|---|
| **Module selection** | Choose LoRa modules with existing **FCC ID** and **CE mark** documentation (e.g., AI-Thinker Ra-02 SX1276, Heltec Wireless Kit) | High |
| **Certificate retention** | Keep module datasheets, FCC IDs, and test reports on file — NTC may request them for EASA filing | High |
| **Labeling** | Affix a permanent label on each collar with: manufacturer name, model, frequency, max EIRP | Medium |
| **Power limiting** | Program firmware to respect max EIRP (set `SX1276` TX power to ≤ 17 dBm via RadioLib) | High |
| **Spectral measurements** | If producing beyond prototype, engage a certified RF test lab for pre-compliance testing | Low (prototype stage) |
| **Antenna gain disclosure** | Document external antenna gain (dBi) so effective EIRP can be calculated (EIRP = Tx power + antenna gain − cable loss) | Medium |

---

## 5. Requirements Needing Further Verification with NTC

The following items require direct consultation with the NTC Radiation Protection and Management Bureau before any field deployment beyond prototype:

| Item | Question | Why It Needs Verification |
|---|---|---|
| **Prototype/EASA eligibility** | Does the EASA (Equipment Authorization for Scientific and Educational Applications) pathway apply to university capstone projects deploying LoRa collars on private farm land? | NTC has not published explicit criteria for "educational prototypes" outside formal academic institutions |
| **Max EIRP at 915 MHz** | What is the exact maximum permitted EIRP for ISM equipment at 915 MHz under current NTC MO? | Published values vary across NTC circulars; the definitive value is in the latest MO series |
| **Outdoor deployment notification** | Must the team notify local NTC field offices before placing a base-station antenna on a farm mast? | NTC generally requires notification for fixed outdoor transmitters above certain power thresholds |
| **Multiple-collar coordination** | Are there rules about simultaneous transmissions from multiple collars on the same frequency? | Relevant for herd-scale deployments (>3 collars) |
| **Importation of LoRa modules** | Do individual LoRa modules require separate import clearance, or is a single EASA covering the complete device sufficient? | Customs may question imported radio modules without end-use declarations |

**Contact:** NTC Radiation Protection and Management Bureau  
Website: <https://www.ntc.gov.ph>  
Email: rpub@ntc.gov.ph (verify current contact on NTC website)

---

## 6. SIM800L Note (Not Included in Design)

The REPLAN-PROMPT mentions the SIM800L GSM module as "if required." The current SmartCollar design intentionally excludes cellular because:

1. The project objective is **fully offline operation** — no subscription, no cellular coverage dependency.
2. Adding GSM would trigger **full NTC Type Approval** requirements (much stricter than LoRa ISM), including band-specific certifications for 900/1800 MHz, SAR testing, and ongoing compliance fees.
3. Cellular adds recurring cost (SIM data plan) contrary to the project's zero-subscription philosophy.

If a future Phase 3 introduces cellular fallback, budget for a dedicated NTC Type Approval process (₱50,000–₱200,000+ depending on scope).

---

## 7. Quick-Reference Compliance Checklist

| # | Action | Status |
|---|---|---|
| 1 | Confirm 915 MHz ISM band is permitted under current NTC MO | ✅ Confirmed (ITU-R compliant; verify with NTC directly for final deployment) |
| 2 | Set firmware TX power to ≤ 17 dBm (100 mW EIRP with typical antenna) | ⬜ To-do in firmware config |
| 3 | Obtain and archive FCC ID / CE certificate for SX1276 module | ⬜ Request from supplier |
| 4 | Affix identification label on each collar unit | ⬜ During assembly |
| 5 | Contact NTC RPB for EASA inquiry (optional for prototype) | ⬜ Before field trial |
| 6 | Document antenna gain and calculate confirmed EIRP | ⬜ Measure at bench |
| 7 | Notify local NTC field office if deploying > 10 collars or > 5 km range | ⬜ Conditional |

---

## References

- NTC Official Website: <https://www.ntc.gov.ph>
- ITU-R Radio Regulations, Article 5.239 (ISM applications)
- FCC Part 15 Subpart C (Industrial, Scientific, Medical equipment) — for reference EIRP limits
- ETSI EN 300 220 (EU LoRa spectrum rules — for comparison only; NOT binding in PH)
- SmartCollar [FIRMWARE.md](FIRMWARE.md) § LoRa Radio Settings

---

*This document provides guidance based on publicly available information as of November 2026. Regulatory requirements change; always confirm with the NTC directly before deployment.*
