</> # **Mamaearth Returns & Growth Intelligence Pipeline** 



Repository Structure
Plaintext
<repo>/
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
    └── generate_narrative.py
Pipeline & Data Flow Guide
Execute the pipeline in three consecutive layers where output from each layer flows directly into the next.

1. SQL Relational Database & Reporting (sql/)
Initialize the database schema: Execute sql/schema.sql to create customers, products, and orders tables.

Populate core records: Run sql/seed_data.sql.

Verify metrics: Execute sql/reports.sql for order volumes, net revenues, and customer lifetime spend rankings.

2. Python Data Cleaning, EDA & Visualization (analysis/ & visualizations/)
Place raw CSV datasets (orders.csv, customers.csv, products.csv) into the data/ directory.

Run analysis/clean_and_eda.py: Standardizes payment casing, deduplicates 5 order rows (₹2,501.90 reconciliation delta), imputes missing values, flags IQR quantity outliers, and executes multi-level segmentation (COD Tier-2 risk at 54.5%).

Automatic Export: Task 5 automatically writes verified figures to narrator/findings.json.

Run analysis/visualize.py: Generates analytical charts (return_rate_by_payment.png and monthly_revenue_trend.png) into visualizations/.

3. GenAI Insight Narrator & Offline Fallback (narrator/)
Run python narrator/generate_narrative.py (reads narrator/findings.json from Layer 2).

Environment Configuration:

Online Mode (Gemini API): Set your free API key as an environment variable or secret:

Bash
export SY_API_KEY="your_free_gemini_api_key"
Offline Keyless Fallback: If no key or network access is available, the script automatically triggers generate_scr_narrative_offline() with zero API spend.

Validation: Saves the final SCR report to narrator/sample_output.txt and runs the numeric accuracy checklist.
