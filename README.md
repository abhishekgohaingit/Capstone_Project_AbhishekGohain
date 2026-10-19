# Mamaearth Regional Operations & Revenue Analysis

## Overview

This project analyzes order revenue, duplicate records, payment-method
return rates, quantity outliers, and monthly revenue trends using SQL
and Python. It also generates a Situation--Complication--Resolution
(SCR) narrative from the calculated findings.

## Project Structure

``` text
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
    └── generate_narrative.py
```

## Prerequisites

-   Python 3.10+
-   MySQL Server and MySQL Workbench (or a MySQL command-line client)
-   Python packages: `pandas`, `numpy`, `matplotlib`
-   `google-genai` for online Gemini generation (optional)

Install the Python dependencies from the repository root:

``` bash
python -m pip install pandas numpy matplotlib
```

For online Gemini generation, also run:

``` bash
python -m pip install google-genai
```

## 1. Run the SQL Pipeline

In MySQL Workbench, connect to your server and execute these scripts
**in order**:

1.  `sql/schema.sql` --- creates the `capstone_project` database and its
    tables.
2.  `sql/seed_data.sql` --- loads the customer, product, and order
    records.
3.  `sql/reports.sql` --- runs the SQL reports.

Run the schema and seed scripts on a fresh database for a clean setup.
Some report statements modify the schema, so you may need to reset the
database before rerunning the full workflow.

## 2. Run Cleaning, EDA, and Visualizations

Run commands from the repository root so relative paths resolve
correctly.

First, clean the data and run exploratory analysis:

``` bash
python analysis/clean_and_eda.py
```

This script loads the CSVs, standardizes payment methods, removes
duplicate orders, handles missing values, analyzes returns and quantity
outliers, and calculates revenue. **Task 5 of Part 2 writes the
calculated findings to `narrator/findings.json`**, which is used by the
narrative generator.

Next, create the charts:

``` bash
python analysis/visualize.py
```

The script creates:

-   `visualizations/return_rate_by_payment.png`
-   `visualizations/monthly_revenue_trend.png`

## 3. Generate the SCR Narrative

Run:

``` bash
python narrator/generate_narrative.py
```

The script reads `narrator/findings.json` and saves the narrative to
`narrator/sample_output.txt`. It attempts online Gemini generation when
an API key is configured; otherwise, it uses the deterministic offline
path. The offline path does not require an API key or network access.

### Option A --- Online Gemini generation

Set your Gemini API key as an environment variable. Get a key from
Google AI Studio, and **never commit the key to GitHub**.

**Windows PowerShell:**

``` powershell
$env:GEMINI_API_KEY = "YOUR_GEMINI_API_KEY"
python narrator/generate_narrative.py
```

**Windows Command Prompt:**

``` bat
set GEMINI_API_KEY=YOUR_GEMINI_API_KEY
python narrator/generate_narrative.py
```

**macOS/Linux:**

``` bash
export GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
python narrator/generate_narrative.py
```

Replace `YOUR_GEMINI_API_KEY` with your actual key. These commands set
the variable for the current terminal session. If the Gemini request
fails or the service is unavailable, the script may fall back to offline
generation.

### Option B --- Offline generation (no key)

Leave `GEMINI_API_KEY` unset and run:

``` bash
python narrator/generate_narrative.py
```

The offline generator builds the SCR sections directly from
`narrator/findings.json`. It does not call Gemini and should include the
same required figures.

## Key Findings to Verify

After running the pipeline, check the generated analysis and
`narrator/sample_output.txt` against these expected results:

  Finding                                           Expected result
  ----------------------------------------------- -----------------
  Raw revenue before cleaning                             99,860.20
  Cleaned total revenue                                   97,358.30
  Duplicate-driven reconciliation delta                    2,501.90
  COD return rate                                             44.4%
  CARD return rate                                            14.7%
  UPI return rate                                             18.9%
  COD + Tier-2 highest-risk segment return rate               54.5%
  True peak month after outlier correction               March 2026
  True peak-month revenue                                 20,318.90
  January revenue including quantity outliers             29,582.10
  January revenue after outlier correction                11,637.10

January's apparent peak is inflated by two unusually large quantity
orders. After excluding those quantity outliers, March is the true peak
month. The narrative generator checks that the required figures appear
in the output.

## Reproducibility and Security

1.  Run the SQL scripts in the order shown above.
2.  Run `analysis/clean_and_eda.py` and confirm `narrator/findings.json`
    is created.
3.  Run `analysis/visualize.py`.
4.  Run `narrator/generate_narrative.py` with a Gemini key for online
    generation, or without a key for offline generation.
5.  Review the generated charts and `narrator/sample_output.txt`.

Keep the original CSV files unchanged and never commit API keys,
passwords, or other secrets.
