
# Building Custom GPTs for Agriculture – Case Study: The Late Blight Risk Advisor  
**By Jorge Luis Alonso with ChatGPT-4o**

## 1. Introduction: Why custom AI tools matter in agriculture
What if farmers could ask an intelligent assistant about the risk of plant disease — and get answers based on weather, crop type, and science?

Now they can. With custom GPTs — AI tools trained for specific tasks like farming — this is no longer just an idea. These assistants don't need large systems or expert users. With the right data and advice, they can help farmers make real decisions in the field.

...

## Appendix: Full GPT instruction set used during deployment
...

[This file is a partial placeholder for the full article. Replace with complete content for final export.]
# Weather Risk Rules Module – Specification for *BlightAlert PL*

This document contains **all numeric thresholds, decision logic and YAML examples** required to implement the three weather-based late-blight forecast rules that ship with the first offline release of *BlightAlert PL*.

---
## 1 Hutton Criteria (UK Met Office, 2017)
*Two-day binary alert – simplest fallback rule*
**Trigger** If *both* of two consecutive days satisfy:
- **Tmin ≥ 10 °C** (daily minimum)
- **≥ 6 h RH ≥ 90 %**

```yaml
Hutton:
  Tmin: 10  # °C
  RH_threshold: 90  # %
  RH_hours: 6  # h per day
  consecutive_days: 2 # rolling window
```

*Status*  No Polish calibration yet; ship as default UK thresholds and flag as "needs ROC on IMGW data".

---
## 2 Modified Irish Rules (MIR, Cucak et al. 2019)
*Continuous-hour state machine with Effective Blight Hours (EBH)*

| Parameter | Value (MIR default) | Notes |
|----------|----------------------|-------|
| **Tmin** | 12 °C | raise from original 10 °C (optional) |
| **RH** | ≥ 88 % | was 90 % in classic IR |
| **Sporulation hours** | 10 h | start EBH counter after 10 h |
| **Leaf wetness check** | rain ≥ 0.1 mm in ±3 h | skip 4 h delay if wet |
| **EBH break** | > 5 h of non-conducive weather | ends period |
| **EBH alert** | region-specific 4–11 h (default 12 h) | ≥ threshold ⇒ "High-risk" |

```yaml
MIR:
  Tmin: 12  # default; set MIR.override_Tmin: 10 to retain classic threshold
  RH_threshold: 88
  sporulation_h: 10
  wet_rain_mm: 0.1  # leaf wetness shortcut
  EBH_break_h: 5
  EBH_alert: 12  # override by voivodeship if desired
```

*Status*  Validated in Ireland; mark as **experimental** in Poland until local ROC done.

---

## 3 NegFry (Ullrich & Schrödter 1966 + Fry 1983) – **Primary engine for PL**

### 3.1 First-spray rule (Negative Prognosis)
```yaml
NegFry:
  first_spray:
    arv_threshold: 130  # Polish trials (Bonin)
    drv_threshold: 7  # last-day safeguard
```

### 3.2 Daily Risk Value (DRV) coefficient table - **clarified**
#### Wetness subclasses
* **Subclass A** = hours that satisfy RH ≥ 90 % **without measurable rain** (≤ 0.1 mm h⁻¹).
* **Subclass B** = hours that satisfy RH ≥ 90 % **with rain > 0.1 mm h⁻¹**.
* **Dry discount** rows apply when 70 % ≤ RH < 90 %. They are **mutually exclusive** with the global `dry_penalty_r`.

#### Coefficient table
| Temp band (°C) | Subclass | min_block_h* | r | YAML key |
|---------------|----------|--------------|---|-----------|
|10–11.9|A|4|0.8990|`wet_A4`|
|10–11.9|B|10|0.3924|`wet_B10`|
|14–15.9|A|0|0.4118|`wet_A0`|
|14–15.9|dry discount|0|0.0702|`dry_disc`|
|16–17.9|A|0|0.5336|`wet_A0`|
|16–17.9|dry discount|0|0.1278|`dry_disc`|
|18–19.9|A|0|0.8816|`wet_A0`|
|18–19.9|B|0|0.9108|`wet_B0`|
|20–21.9|A|0|1.0498|`wet_A0`|
|20–21.9|B|0|1.4706|`wet_B0`|
|22–23.9|A|0|0.5858|`wet_A0`|
|22–23.9|B|0|0.8550|`wet_B0`|
|15–19.9 (any RH)|baseline|-|+0.1639|`baseline_r`|
|any T, RH < 70 %|dry_penalty|-|−0.0468|`dry_penalty_r`|

*min_block_h = minimum continuous wet hours before the r-weight applies when RH ≥ 90 %.*

#### Precedence rules
**Wet precedence** B ➜ A ➜ dry discount ➜ baseline/dry-penalty  
**No double subtraction** dry discount excludes `dry_penalty_r`  
**Baseline & wet coexist** `baseline_r` always adds in 15–19.9 °C

#### Algorithm (per hour)
Determine wetness class (precedence rules).  
Track current wet spell; at end apply `DRV += r × wet_hours` for A/B.  
Apply dry-discount or dry_penalty instantly to those hours.  
Add `baseline_r` if 15 ≤ T < 20 °C.  
Sum 24 h → **DRV**.  
Daily DRV totals reproduce Bonin 2001–02 trial values within ± 0.1 units.

### 3.3 Repeat-spray rule (Fry/BU logic)
```yaml
repeat_spray:
  bu_threshold:
    susceptible: 40
    moderately_susceptible: 45
    moderately_resistant: 50
  rain_trigger_mm: 20  # wash-off
```

*Map Polish EuroBlight foliar scores to these classes.*

### 3.4 State algorithm
From emergence accumulate ARV; when ARV ≥ 130 **and** DRV ≥ 7 → first spray.  
After spray reset; accumulate BU + rain.  
If BU ≥ threshold **or** rain > 20 mm → spray, reset.

---

## 4 Integration logic
* Run **NegFry** daily (primary).
* Run **Hutton** & **MIR** in parallel for corroboration.
* Dashboard: *NegFry status | Hutton flag | MIR EBH*.

---

## 5 Validation targets (2025)
| Metric | Goal |
|--------|------|
| First spray lead | ≥ 0 d before lesions on > 90 % farms |
| Spray reduction | ≥ 25 % vs routine |
| False high rate | ≤ 1 / month (Hutton/MIR) |

---

## 6 Open items
ROC back-test of Hutton & MIR on 2015–24 IMGW data.  
Fine-map BU classes to PL resistance scores.  
Annual re-validation vs new climate & genotypes.

---

### Key sources
Kapsa J. et al. 2003 – Polish NegFry validation.  
Cucak G. et al. 2019 – MIR thresholds.  
UC IPM Late Blight Model – DRV & BU logic.  
AHDB/Met Office – Hutton Criteria.  
*Prepared 13 May 2025 for BlightAlert PL v0.1 implementation.*