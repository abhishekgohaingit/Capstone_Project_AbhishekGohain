# Mamaearth Regional Operations & Revenue Analysis

## Project Overview

This project analyzes Mamaearth's order data to identify revenue discrepancies, customer return patterns, payment-method risks, and monthly revenue trends. It combines SQL reporting, Python/Pandas data cleaning, exploratory data analysis (EDA), visualizations, and an AI-generated Situation–Complication–Resolution (SCR) business narrative.

The project uses three datasets:

- `data/customers.csv`
- `data/products.csv`
- `data/orders.csv`

The analysis identifies duplicate order records, standardizes payment methods, handles missing values, detects quantity outliers, calculates return rates, and reconciles raw and cleaned revenue.

## Project Structure

```text
project-root/
├── data/
│   ├── customers.csv
│   ├── products.csv
│   └── orders.csv
├── sql/
│   ├── schema.sql
│   ├── seed_data.sql
│   └── reports.sql
├── analysis/
│   ├── clean_and_eda.py
│   └── visualize.py
├── narrator/
│   ├── generate_narrative.py
│   ├── findings.json
│   └── sample_output.txt
├── visualizations/
│   ├── return_rate_by_payment.png
│   └── monthly_revenue_trend.png
└── README.md
```

## Prerequisites

Install the following before running the project:

- Python 3.10 or later
- MySQL Server and MySQL Workbench, or a compatible MySQL command-line client
- Python libraries: `pandas`, `numpy`, and `matplotlib`
- The `google-genai` library for online Gemini narrative generation (optional for offline execution)

Install the Python dependencies from a terminal:

```bash
python -m pip install pandas numpy matplotlib
```

For Gemini online generation, install the additional library:

```bash
python -m pip install google-genai
```

## 1. Load the SQL Schema and Seed Data, Then Run the Reports

The SQL scripts create the database tables, load the initial datasets, and execute SQL reports.

### Step 1.1 — Create the database and tables

Open MySQL Workbench and connect to your MySQL Server.

Open and execute:

```text
sql/schema.sql
```

This script creates the `capstone_project` database and the `customers`, `products`, and `orders` tables, including their primary and foreign key constraints.

### Step 1.2 — Load the seed data

Open and execute:

```text
sql/seed_data.sql
```

This inserts the customer, product, and order records into the database. Execute the schema before loading the seed data because the tables must exist first.

### Step 1.3 — Run the SQL reports

Open and execute:

```text
sql/reports.sql
```

The reports demonstrate SQL techniques including:

- Aggregations using `COUNT`, `SUM`, and `AVG`
- `INNER JOIN` and `LEFT JOIN`
- `GROUP BY` and `HAVING`
- Subqueries and `DISTINCT`
- Customer spending rankings
- `LIKE` pattern matching
- `ALTER TABLE` and `UPDATE` using `CASE`

The SQL reports establish the raw-data baseline. The initial dataset contains 180 order records, and the raw order revenue is **99,860.20** before duplicate removal and cleaning.

**Important:** Run the schema and seed scripts against a fresh database for a clean setup. Some statements in `reports.sql` modify the schema, so rerunning the complete workflow against an already-modified database may require resetting the database first.

## 2. Run the Python Cleaning, EDA, and Visualization Pipeline

The Python analysis independently loads the original CSV files. It does not require the SQL database to be running.

### Step 2.1 — Check the input files

Ensure that all three CSV files are located in the `data/` directory:

```text
data/orders.csv
data/customers.csv
data/products.csv
```

Run the commands from the project root so that the scripts can resolve the relative file paths.

### Step 2.2 — Run data cleaning and exploratory analysis

Execute:

```bash
python analysis/clean_and_eda.py
```

The script performs the Part 2 Tasks 1–10 workflow, including:

1. Loading and inspecting the CSV files.
2. Standardizing payment-method values.
3. Detecting and removing duplicate orders.
4. Imputing missing discount and rating values.
5. Merging orders with products and customers.
6. Calculating order values.
7. Detecting quantity outliers using the interquartile range (IQR) rule.
8. Analyzing return rates by payment method and customer segment.
9. Exploring correlations between selected variables.
10. Comparing monthly revenue with and without quantity outliers.

The analysis also verifies the expected results and exports the structured findings used by the narrative generator.

### Step 2.3 — Generate `findings.json`

**Task 5 of Part 2** builds the findings dictionary from the calculated analysis results and writes it to a JSON file. This file supplies the figures used by the AI narrative generator, avoiding the need to manually type those values into the prompt.

The expected narrative input location is:

```text
narrator/findings.json
```

Check the output path printed by `clean_and_eda.py`. If your current version writes the file to `json/findings.json`, copy it to the expected location before proceeding:

```bash
mkdir -p narrator
cp json/findings.json narrator/findings.json
```

On Windows Command Prompt, use:

```bat
if not exist narrator mkdir narrator
copy json\findings.json narrator\findings.json
```

Run these commands from the project root. The generated findings file should contain the calculated revenue totals, payment-method return rates, highest-risk segment, true peak month, and outlier-inflated month.

### Step 2.4 — Generate the visualizations

After the analysis has completed, execute:

```bash
python analysis/visualize.py
```

The visualization script produces the following charts:

- `visualizations/return_rate_by_payment.png` — compares return rates across payment methods.
- `visualizations/monthly_revenue_trend.png` — displays the monthly revenue trend after excluding quantity outliers.

The visualizations use the same cleaning and outlier logic as the analysis pipeline. Ensure the `visualizations/` directory exists or allow the script to create it.

## 3. Generate the SCR Narrative with Gemini or the Offline Fallback

The final stage reads `narrator/findings.json` and generates a business narrative using the Situation–Complication–Resolution (SCR) framework.

The narrative explains the current financial situation, highlights the operational complication, and recommends actions based on the calculated findings.

### Step 3.1 — Run the narrative generator

Execute:

```bash
python narrator/generate_narrative.py
```

The script reads the findings JSON, attempts Gemini generation when a valid API key and the required library are available, and uses the deterministic offline function when no API key is configured or the API call fails.

The output is saved to:

```text
narrator/sample_output.txt
```

The script also checks that the narrative contains the five required figures.

### Step 3.2 — Configure a Gemini API key for online generation

To use Gemini, first install the library:

```bash
python -m pip install google-genai
```

Then configure an API key as an environment variable. Obtain a key from Google AI Studio and set it in your environment. Do not commit the key to GitHub.

**Windows PowerShell:**

```powershell
$env:GEMINI_API_KEY = "YOUR_GEMINI_API_KEY"
python narrator/generate_narrative.py
```

**Windows Command Prompt:**

```bat
set GEMINI_API_KEY=YOUR_GEMINI_API_KEY
python narrator/generate_narrative.py
```

**macOS or Linux:**

```bash
export GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
python narrator/generate_narrative.py
```

Replace `YOUR_GEMINI_API_KEY` with your actual key. These examples configure the variable for the current terminal session. For persistent configuration, use your operating system's environment-variable settings or a secure secrets manager.

If running in Google Colab, the notebook version can retrieve the API key from Colab Secrets using the configured secret name.

The online function uses an explicit generation configuration, including `temperature=0.0`, a maximum output-token limit, and a request timeout. It returns a structured success or error result rather than exposing an uncaught API exception.

### Step 3.3 — Run without an API key (offline mode)

**An API key is not required to run the offline path.**

If no API key is configured, the generator uses:

```python
generate_scr_narrative_offline(findings: dict) -> dict
```

This function builds the three SCR sections directly from the supplied findings. It does not call Gemini, require an API key, or use the network.

The offline narrative uses the same findings and must include the same five required figures. It is therefore suitable for reproducible execution and environments where external API access is unavailable.

The same command runs the script:

```bash
python narrator/generate_narrative.py
```

When the key is absent, the offline path should run automatically, provided the findings file and Python dependencies needed for the analysis are available.

## 4. Verified Findings and Numeric Accuracy

The analysis is expected to reproduce the following key results from the supplied dataset.

| Finding | Expected result |
|---|---:|
| Raw revenue before cleaning | 99,860.20 |
| Cleaned total revenue | 97,358.30 |
| Duplicate-driven reconciliation delta | 2,501.90 |
| COD return rate | 44.4% |
| CARD return rate | 14.7% |
| UPI return rate | 18.9% |
| COD + Tier-2 highest-risk segment return rate | 54.5% |
| True peak month after outlier correction | March 2026 |
| True peak-month revenue | 20,318.90 |
| January revenue including quantity outliers | 29,582.10 |
| January revenue after outlier correction | 11,637.10 |

The cleaned revenue is lower than the raw revenue because duplicate order records are removed during data cleaning. The reconciliation delta represents the difference between the raw and cleaned totals.

January appears to be the leading month before outlier correction because of two unusually large quantity orders. After those quantity outliers are excluded, March becomes the true peak month.

The SCR narrative's numeric checker verifies the required values in the generated text. For online generation, review the saved `narrator/sample_output.txt` to confirm that the generated narrative contains the required figures and month name. The offline path should pass the same checks using its deterministic template.

## 5. Recommended Execution Order

For a reproducible run from the beginning, follow these steps in order:

1. Execute `sql/schema.sql`.
2. Execute `sql/seed_data.sql`.
3. Execute `sql/reports.sql`.
4. From the project root, run `python analysis/clean_and_eda.py`.
5. Confirm that the findings JSON was generated and is available at `narrator/findings.json`. Copy it from `json/findings.json` if your current script uses that output location.
6. Run `python analysis/visualize.py`.
7. Configure `GEMINI_API_KEY` if you want online generation. Otherwise, leave it unset to use offline generation.
8. Run `python narrator/generate_narrative.py`.
9. Review `narrator/sample_output.txt` and verify the numeric-checker results.

Following this order reproduces the SQL reports, cleaned analytical findings, visualizations, and SCR narrative from the project's source data.

## Security and Reproducibility

- Never commit API keys, passwords, or other secrets to the repository.
- Keep the original CSV files unchanged so the analysis can be reproduced.
- Run commands from the project root unless a script explicitly supports another working directory.
- Use the generated findings JSON as the source for the SCR narrative rather than manually entering the figures.
- Review the saved narrative and numeric-checker output before submitting the project.
