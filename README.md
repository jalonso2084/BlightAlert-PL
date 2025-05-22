# BlightAlert-PL

**BlightAlert-PL** is an **offline decision-support kit** that turns any Custom GPT built in ChatGPT’s *GPT Builder* into a late-blight forecaster for Polish potato growers.  
All risk logic (Hutton, Modified Irish Rules, NegFry) and local agronomic data are shipped as plain text files—**no external APIs, no extra LLM weights, no cloud inference**.

---

## 1  Repo contents

| File | Purpose |
|------|---------|
| `weather_rules.yaml` | Machine-readable thresholds for Hutton, MIR and NegFry (ARV/DRV, EBH, rain wash-off). |
| `negfry_drv_table.csv` | 14-row coefficient table + baseline/dry-penalties to compute NegFry Daily Risk Value from hourly wetness blocks. |
| `bu_thresholds.json` | BU limits applied between sprays: **40 / 45 / 50**. |
| `cultivar_resistance.csv` | 82 Polish potato varieties → COBORU/EuroBlight foliar score → BU class. |
| `quick_start.md` | One-page bilingual guide that growers can copy-paste into the GPT’s *Welcome* message. |
| `validation_report.pdf` | Evidence pack from IHAR-Bonin 2001-02 trials showing NegFry cut sprays by ~34 % with no yield loss. |

---

## 2  Quick start (Custom GPT)

1. **Create a new GPT** in ChatGPT → *GPT Builder*.  
2. **Capabilities** → switch **Code Interpreter ON**; Browsing & Image Gen OFF.  
3. **Knowledge** → drag in all six files from this repo.  
4. **Instructions** → paste the system prompt from `/docs/system_prompt.txt` (or use the one in the README [here](#system-prompt)).  
5. Save → type a test line:  

2025-06-12; Tmin=10.7; RH90h=9; Rain=3.1; Variety=Bryza; LastSpray=16

The GPT should reply **Ryzyko: Wysokie // High** and advise a spray within 48 h.

That’s it—BlightAlert-PL now works on any device that can open ChatGPT.

---

## 3  Updating data

* **New cultivar** → append a row to `cultivar_resistance.csv` and re-upload.  
* **Rule tweak** → edit `weather_rules.yaml`; no code changes needed.  
* **Yearly validation** → drop a new PDF into the repo; mention it in `quick_start.md`.

---

## 4  Folder structure

BlightAlert-PL/
├── weather_rules.yaml
├── negfry_drv_table.csv
├── bu_thresholds.json
├── cultivar_resistance.csv
├── quick_start.md
├── validation_report.pdf
└── README.md ← you are here


---

## 5  System prompt (reference)

<details>
<summary>Click to view</summary>

You are BlightAlert-PL, a self-contained decision-support assistant for Polish potato growers.
Reply first in Polish, then a concise English text after “//”.
Accept CSV line or one-liner (Date,Tmin,Tmax,RH90h,Rainfall,Variety,LastSprayDays).
Load rules from weather_rules.yaml, negfry_drv_table.csv, bu_thresholds.json.
Map Variety via cultivar_resistance.csv.
On each request:
• parse entry; update NegFry state; check Hutton, MIR;
• return Risk = Niski/Średni/Wysoki, rule(s) triggered, spray guidance, BU carry-over;
• log date, risk, action; show log on “pokaż historię”.
Never call external URLs or APIs; politely refuse non-blight questions.
(Helper code loads files and defines update_negfry.)

</details>

---

## 6  License

Source files are released under **AGPL-v3**.  
Scientific thresholds originate from openly published literature (cited in the spec).

---

## 7  Citation

If you use BlightAlert-PL in research or extension work, please cite:

> **Jorge Luis Alonso**, AI-Driven Agricultural Data Specialist, https://www.linkedin.com/in/jorgeluisalonso/

---

*Prepared 13 May 2025 · BlightAlert-PL v0.1*

