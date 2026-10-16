# Mamaearth Returns & Growth Intelligence Pipeline

**Program:** Data Analytics with AI & Gen AI — E&ICT Academy, IIT Roorkee  
**Project:** 15-day capstone project  
**Goal:** Build an end-to-end SQL, Python/Pandas, visualization, and GenAI reporting pipeline for Mamaearth's order data.

## 1. Project Overview

The project has three connected layers:

1. **SQL reporting layer** — creates a relational database from the raw CSV files and runs baseline reports.
2. **Python/Pandas analysis layer** — independently loads the raw CSVs, standardizes payment methods, removes duplicate orders, imputes missing values, analyses returns and outliers, and writes verified findings.
3. **GenAI narrative layer** — uses the verified findings to generate a Situation–Complication–Resolution (SCR) business narrative. It supports Gemini API access and a deterministic offline fallback.

Keep the original CSV files unchanged. Data cleaning belongs in the Python pipeline, not in manual edits to the source files.

## 2. Repository Structure

```text
.
├── README.md
├── sql/
│   ├── schema.sql
│   ├── seed_data.sql
│   └── reports.sql
├── data/
│   ├── customers.csv
│   ├── products.csv
│   └── orders.csv
├── analysis/
│   ├── clean_and_eda.py
│   └── visualize.py
├── visualizations/
│   ├── return_rate_by_payment.png
│   └── monthly_revenue_trend.png
└── narrator/
    ├── findings.json
    ├── generate_narrative.py
    └── sample_output.txt
```

## 3. Requirements

- SQLite 3
- Python 3.10+
- Python packages: `pandas`, `matplotlib`
- `google-genai` for the optional Gemini API path
- Optional Gemini API key from Google AI Studio; the offline narrative path should work without a key or network access

Install the Python packages:

```bash
python -m pip install pandas matplotlib google-genai
```

Run all commands from the repository root (the folder containing this README).

## 4. Run the Pipeline in Order

### Step 1 — Create and load the SQL database

The project brief allows SQLite or another SQL engine supporting the required syntax. The commands below assume SQLite and that the SQL scripts are written for SQLite:

```bash
sqlite3 mamaearth.db < sql/schema.sql
sqlite3 mamaearth.db < sql/seed_data.sql
```

The three CSV files should remain in `data/`:

- `data/customers.csv` — 45 rows
- `data/products.csv` — 16 rows
- `data/orders.csv` — 180 rows

Check the row counts:

```bash
sqlite3 mamaearth.db "SELECT COUNT(*) FROM customers;"
sqlite3 mamaearth.db "SELECT COUNT(*) FROM products;"
sqlite3 mamaearth.db "SELECT COUNT(*) FROM orders;"
```

Expected counts: `customers = 45`, `products = 16`, `orders = 180`.

Run the SQL reports:

```bash
sqlite3 -header -column mamaearth.db < sql/reports.sql
```

`reports.sql` should run on the **raw, uncleaned** data. The expected checks include 180 orders, 165 non-null ratings, 15 unrated orders, and one customer without orders (`C045`, Vihaan). The five duplicate records are intentionally retained in the SQL layer; duplicate removal is a Python cleaning step.

**SQLite import note:** If the loader uses SQLite's `.import`, blank CSV cells may be imported as empty strings instead of SQL `NULL`. Convert blank `discount_pct` and `rating` cells to `NULL` before running the reports. If `seed_data.sql` inserts the CSV rows with proper `NULL` values, this cleanup is not needed.

### Step 2 — Run Python data cleaning and EDA

```bash
python analysis/clean_and_eda.py
```

This script should read `data/orders.csv`, `data/customers.csv`, and `data/products.csv` directly, independently of the SQL database. It should print the intermediate results required by the brief, including:

- Raw dimensions before cleaning (`orders.shape` should be `(180, 9)`)
- Payment-method values/counts before and after standardization
- The five duplicate IDs removed (`O0176`–`O0180`)
- Missing-value counts and rating median
- Raw-versus-cleaned revenue reconciliation
- IQR bounds and flagged quantity outliers
- Return rates by payment method and by payment method/city tier
- Correlations and their strength classifications
- Monthly revenue with and without flagged outliers

The expected cleaned dataset has **175 orders**. The expected cleaned revenue is **₹97,358.30**, versus **₹99,860.20** for the raw SQL report. The difference of **₹2,501.90** must be reconciled to the combined value of the five duplicate rows removed. Imputation of missing discount/rating values should not change the order-value total.

The brief identifies COD + Tier-2 cities as the highest-risk segment, with a **54.5%** return rate. The expected outlier-corrected peak is **March 2026**, with revenue of **₹20,318.90**.

At the end of this step, `clean_and_eda.py` must generate `narrator/findings.json` from the computed results. Do not hand-maintain the numbers in that JSON file.

### Step 3 — Generate visualizations

```bash
python analysis/visualize.py
```

The script should save these two files under `visualizations/`:

- `return_rate_by_payment.png` — bar chart of return rate by standardized payment method, ordered descending, with percentage labels and a title stating the finding.
- `monthly_revenue_trend.png` — line chart of outlier-corrected monthly revenue, with axis labels and a title identifying March 2026 as the peak month.

Both Python scripts should be re-runnable and regenerate their outputs from the raw CSVs.

### Step 4 — Generate the SCR narrative

```bash
python narrator/generate_narrative.py
```

The narrator should read `narrator/findings.json` and produce three labeled sections: **Situation**, **Complication**, and **Resolution**. All reported numbers must come from the supplied findings.

The narrative must include these figures: cleaned revenue **₹97,358.30**, COD return rate **44.4%**, COD + Tier-2 return rate **54.5%**, duplicate reconciliation difference **₹2,501.90**, and March peak revenue **₹20,318.90**.

#### Option A — Gemini API

Create a key in Google AI Studio. The code should read it from the `GEMINI_API_KEY` environment variable.

**Windows Command Prompt:**

```bat
set GEMINI_API_KEY=your_api_key_here
python narrator/generate_narrative.py
```

**Windows PowerShell:**

```powershell
$env:GEMINI_API_KEY="your_api_key_here"
python narrator/generate_narrative.py
```

**macOS/Linux:**

```bash
export GEMINI_API_KEY="your_api_key_here"
python narrator/generate_narrative.py
```

Never commit your API key or hardcode it in the source.

#### Option B — Offline fallback

Leave `GEMINI_API_KEY` unset and run:

```bash
python narrator/generate_narrative.py
```

If no key is configured, or the API call fails, the script should use `generate_scr_narrative_offline(findings)`. This fallback must not make a network request and must build its three SCR sections from the values in `findings.json`.

The project also requires a numeric checker that verifies each required figure in the narrative and prints a pass/fail result. For an online Gemini run, save the actual output in `narrator/sample_output.txt` and run the checker against that saved output.

## 5. Key Data-Quality Rules

- Preserve the raw CSVs unchanged.
- Standardize `payment_method` with trimming and uppercase conversion. The cleaned categories should be `CARD`, `COD`, and `UPI`.
- Detect duplicates using the natural key `customer_id`, `product_id`, `order_date`, `quantity`, `discount_pct`, `payment_method`, `rating`, and `returned`. Exclude `order_id`, because the simulated duplicates have different IDs.
- Fill missing `discount_pct` values with `0`.
- Fill missing `rating` values with the median of non-null ratings after deduplication.
- Flag, but do not delete, the IQR quantity outliers: `O0011` (quantity 25) and `O0098` (quantity 30).
- Use outlier-corrected monthly revenue for the trend chart.
- Do not call revenue “profit” or “margin”: the provided data has no product cost field, so true profit margins cannot be calculated from these columns alone.

## 6. Expected Findings

Use these values as acceptance criteria from the project brief and verify them by running the scripts:

| Metric | Expected value |
|---|---:|
| Raw order count | 180 |
| Cleaned order count | 175 |
| Raw revenue | ₹99,860.20 |
| Cleaned revenue | ₹97,358.30 |
| Duplicate-driven revenue difference | ₹2,501.90 |
| COD return rate | 44.4% |
| CARD return rate | 14.7% |
| UPI return rate | 18.9% |
| Highest-risk segment | COD + Tier-2 |
| Highest-risk segment return rate | 54.5% |
| Outlier-corrected peak month | March 2026 |
| Outlier-corrected peak-month revenue | ₹20,318.90 |

## 7. Troubleshooting

- **`sqlite3` is not recognized:** Install SQLite and add it to your PATH, or use a SQLite GUI such as DB Browser for SQLite.
- **Python cannot find a CSV:** Run commands from the repository root and check that the files are in `data/` with the exact filenames above.
- **`findings.json` is missing:** Run `python analysis/clean_and_eda.py` first.
- **Gemini fails or no API key is set:** Use the offline fallback path.
- **The SQL rating count is incorrect:** Check whether blank ratings were imported as empty strings. They must be `NULL` for `COUNT(rating)` to return 165.

## 8. Final Submission Checklist

- [ ] Keep the three original CSVs unchanged in `data/`.
- [ ] Run `sql/schema.sql`, then `sql/seed_data.sql`.
- [ ] Run `sql/reports.sql` and check its outputs.
- [ ] Run `python analysis/clean_and_eda.py`.
- [ ] Confirm that `narrator/findings.json` is generated and acceptance criteria match.
- [ ] Run `python analysis/visualize.py` and verify both PNGs.
- [ ] Run `python narrator/generate_narrative.py` with Gemini or offline mode.
- [ ] Verify the numeric checker and save `narrator/sample_output.txt` as required.
- [ ] Commit the project to one public GitHub repository and submit the repository link.

---

**Note:** This README describes the workflow and expected results specified in the project brief. Ensure that the actual SQL/Python scripts implement the steps above and test the commands in your environment before submission.

