# PO Foreign Currency Automation

Converts Tall Emu Purchase Orders from foreign currency (e.g. EUR) to AUD using XE confirmation PDFs. Opens a single browser session, processes all POs, and saves each one before moving to the next.

---

## What it does

1. Scans each subfolder in `input/` — the folder name is the PO number (e.g. `P-972`)
2. Parses the XE confirmation PDF (`C*.pdf`) for the exchange rate and AUD total
3. Opens Tall Emu in a browser, finds the PO, and applies one of three actions:
   - **Already updated** (total matches XE total) → skips
   - **Small rounding diff** (< $1) → adjusts freight only
   - **Full conversion** (> $1 off) → divides all line item prices by the XE rate, sets currency to AUD
4. Saves and moves to the next PO

---

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
playwright install chromium
```

### 2. Configure credentials

Copy `dotenv.example` to a file named `dotenv` inside the `Passwords DO NOT LEAK` folder (one level up from this project):

```
TALL_EMU_URL=https://your-instance.tallemu.com
TALL_EMU_EMAIL=you@example.com
TALL_EMU_PASSWORD=yourpassword
INPUT_DIR=input   # optional, defaults to "input"
```

### 3. Add PO folders

```
input/
├── P-972/
│   └── C20419284.pdf      ← XE confirmation PDF (must start with "C")
└── P-973/
    └── C20419371.pdf
```

---

## Usage

```bash
# Process all PO folders
python run.py

# Parse XE PDFs only — skip Tall Emu (useful for testing)
python run.py --dry-run
```

---

## Project structure

```
Item Costs/
├── run.py                      # Entry point
├── requirements.txt
├── dotenv.example
├── input/                      # One subfolder per PO
└── po_automation/
    ├── config.py               # Credentials + paths from .env
    ├── xe_parser.py            # Parses C*.pdf for rate + AUD total
    └── tall_emu.py             # Playwright automation
```

---

## Notes

- The browser runs **visible** (`headless=False`) so you can watch and intervene if needed
- If a conversion still leaves the total more than $5 off from the XE total, the script discards all changes (goes back without saving) and logs an error — the PO is left untouched
- XE rate format: `0.60205` means `1 AUD = 0.60205 EUR`, so `EUR → AUD = EUR amount / rate`
- Already-processed POs are detected by comparing the PO total to the XE total — no manual tracking needed
