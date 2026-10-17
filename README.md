# German E-commerce: Revenue & Customer Analytics

A reproducible SQL case study exploring **sales growth, repeat purchasing, and customer value** in a synthetic German online-retail dataset. Five analyses turn order records into revenue trends, purchase-cohort retention, customer segments, category contributions, and city-level customer rankings.

**Business question:** Where does revenue come from, and which customer groups warrant further investigation for retention and targeted marketing?

## Project at a glance

| Area | Scope |
|---|---|
| Dataset | 22,000 synthetic orders; 2,998 purchasing customers from 3,000 generated customer profiles |
| Coverage | January 10, 2023 to June 18, 2025; 10 German cities; 7 product categories |
| Workflow | Python data generation → CSV → in-memory DuckDB → five SQL result tables |
| Tools | SQL, Python standard library, DuckDB |
| Deliverable | Executable analytical queries and console results; no dashboard or live data connection |

This is an adapted learning case study. All records are generated; findings describe the simulation, not the German retail market or a real business.

## Business questions and analytical approach

| Question | Approach | Decision it can inform |
|---|---|---|
| How are sales changing over time? | Monthly revenue, order counts, and month-over-month growth | Periods requiring investigation into acquisition, basket value, or product mix |
| Do first-time buyers return? | Monthly first-purchase cohorts and activity in months 0–6 | Timing and audience selection for re-engagement experiments |
| Which customers are valuable or becoming inactive? | Recency, frequency, and monetary (RFM) segmentation | Differentiated loyalty, onboarding, and win-back approaches |
| How concentrated is revenue? | Category shares and cumulative revenue | Categories to prioritize for deeper commercial analysis |
| Who are the leading buyers in each city? | Customer spending ranked within city | Local customer targeting and account review |

## Findings reproduced from the code

Results below were verified using the bundled generator with random seed `7` and DuckDB `1.5.5`.

| Observation | Verified result | Interpretation |
|---|---|---|
| Revenue grows substantially in the sample | January 2023: €2,281; December 2023: €51,797 | Demonstrates monthly growth analysis; the generator's signup and ordering rules influence the trend |
| Revenue is concentrated in three categories | Electronics: 36.0%; Garden: 18.5%; Sport: 15.1%; combined approximately 69.5% from unrounded values | A starting point for investigating category dependence and assortment priorities |
| RFM distinguishes engaged and inactive buyers | 637 Champions, approximately 21.2% of purchasing customers, with €1,495 average observed spend | A candidate audience for loyalty or cross-selling experiments |
| Previously frequent buyers can become inactive | 661 customers classified as At Risk, with €1,277 average observed spend | A candidate audience for investigating purchase gaps before testing win-back campaigns |
| Early cohorts show uneven repeat purchasing | Month-1 retention: January cohort 36.4% (4/11); February cohort 22.7% (5/22) | Small cohorts are volatile; these two examples should not be generalized to all customers |

Revenue and average-spend figures are rounded for presentation. These are descriptive findings, not measured improvements from a marketing intervention. Category differences are also influenced by the generator's category-specific price assumptions.

## Metric definitions

- **Revenue:** `SUM(amount_eur * quantity)`. The analysis treats `amount_eur` as a unit amount; there are no separate discounts, taxes, returns, refunds, or costs.
- **Monthly growth:** percentage change from the preceding observed month's revenue. The first month has no comparison value. Missing calendar months are not explicitly filled.
- **Purchase-cohort retention:** distinct customers purchasing in month N divided by the number of customers whose first purchase occurred in the cohort month. It measures activity in that month, not uninterrupted retention.
- **Recency:** days since the customer's last purchase, measured against the latest order date in the dataset rather than today's date.
- **Frequency / monetary value:** observed order count and revenue per customer over the dataset window.
- **RFM segments:** quartile scores combined into Champions, Loyal, Recent / Promising, At Risk, and Hibernating. These are relative behavioral labels, not validated predictions of churn.
- **Top customers by city:** spending rank within each city. Ties can produce more than three customers at ranks 1–3.

The SQL output calls average observed customer spend `avg_lifetime_eur`; this is historical spending, not predicted customer lifetime value or profit.

## Run the project

Use Python 3 and install DuckDB. Run commands from the repository root. The steps below were verified with Python 3.11 and DuckDB 1.5.5.

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install duckdb==1.5.5
New-Item -ItemType Directory -Path data -Force | Out-Null
.\.venv\Scripts\python.exe generate_data.py
.\.venv\Scripts\python.exe run_analysis.py
```

### macOS / Linux

```bash
python3 -m venv .venv
.venv/bin/python -m pip install duckdb==1.5.5
mkdir -p data
.venv/bin/python generate_data.py
.venv/bin/python run_analysis.py
```

The generator overwrites `data/orders.csv`. The analysis runner loads the CSV into an in-memory database and executes every SQL file in `queries/` in filename order. It prints the first 12 rows of each result plus the total row count when more rows exist. It does not persist the database or export full result files.

No credentials or external data services are required. The generated CSV is ignored by Git.

## Repository guide

| File | Purpose |
|---|---|
| `generate_data.py` | Reproducible synthetic order generation with a fixed random seed |
| `run_analysis.py` | CSV ingestion and sequential execution of the query modules |
| `queries/01_monthly_revenue_growth.sql` | Monthly sales and growth using `LAG()` |
| `queries/02_cohort_retention.sql` | First-purchase cohorts and month-offset retention |
| `queries/03_rfm_segmentation.sql` | RFM quartile scoring with `NTILE()` and segment rules |
| `queries/04_category_running_total.sql` | Category shares and cumulative revenue with window aggregates |
| `queries/05_top_customers_per_city.sql` | Customer ranking with `RANK()` and `PARTITION BY` |

The SQL demonstrates date handling, aggregation, multi-step CTEs, joins, window functions, and conditional classification in a business context.

## Limitations and next steps

- **Synthetic behavior:** growth, prices, and repeat purchases reflect generation rules. The dataset cannot establish real market demand, seasonality, or causal effects.
- **Incomplete observation windows:** June 2025 is partial. Recent cohorts have not had six full months to return, so missing future observations must not be interpreted as 0% retention. The query does not generate a complete cohort/calendar grid.
- **Historical coverage:** profiles are generated before orders, but only purchasing customers enter the analysis. Acquisition conversion cannot be measured from the order table alone.
- **Revenue scope:** no order status, profit margin, returns, or marketing spend is modeled. The project does not measure net revenue, CAC, or campaign ROI.
- **Production readiness:** input-schema checks, automated data-quality controls, persisted exports, and an interactive reporting layer remain possible extensions.

A real-data adaptation would require mapping source fields, verifying whether amounts are unit or order totals, defining valid transaction statuses, and setting consistent observation windows.

## Reuse status

The source material did not include a license file. This documentation update does not grant a license or transfer ownership of the underlying implementation.
