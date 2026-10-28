# Olist E-commerce Analysis

Business analysis of the Olist Brazilian e-commerce public dataset (about 99k orders, spanning 2016 to 2018).
Tools: Python, DuckDB (SQL), pandas, Power BI.

## Business questions
1. Which product categories generate the most revenue, and how has revenue trended month by month?
2. How late are deliveries compared with the estimated date, and does lateness hurt review scores?
3. Which states have the most customers, and where is delivery slowest?
4. How many customers buy more than once?
5. Which payment methods are used most, and how do they differ in order value?

## Definitions and data notes
- **Revenue** = sum of item price, excluding freight. Freight is a shipping charge, not product sales.
- Only **delivered** orders are counted (realized revenue). Canceled, unavailable and in-progress orders are excluded.
- Currency is Brazilian reais (R$).
- Reviews are deduplicated to one per order (814 duplicate review IDs in the raw data).
- Monthly trends use full months only (Jan 2017 to Aug 2018). There are no delivered orders in Nov 2016 or after Aug 2018.

## Steps to reproduce
1. Clone the repo and create an environment (Python 3.12 recommended):
   ```
   git clone https://github.com/eQuaEnoo/first-analysis.git
   cd first-analysis
   python -m venv .venv
   .venv\Scripts\activate
   pip install -r requirements.txt
   ```
2. Download the **Brazilian E-Commerce Public Dataset by Olist** from Kaggle and place the 9 CSV files in `data/raw/`.
3. Open `notebooks/01-explore.ipynb`, select the `.venv` kernel and run all cells in order. The first cell builds a local DuckDB database from the CSVs, and the rest run the cleaning and analysis.

The raw data and the database file are not stored in this repo (see `.gitignore`).

## Findings

### Q1: Revenue by category and over time
- Revenue is spread across many categories: the top 5 make up about 40% of revenue and the top 10 about 62%.
- health_beauty (R$1.23M, 9.3%) and watches_gifts (R$1.17M, 8.8%) lead. watches_gifts earns its place with far fewer orders than bed_bath_table (5,493 vs 9,272), meaning a higher spend per order (about R$212 vs R$110).
- Monthly revenue grew from about R$112k in Jan 2017 to R$648k in Oct 2017, then jumped 52% to R$988k in Nov 2017. Daily data shows why: on Black Friday (24 Nov 2017) revenue hit R$150k, 6.6x the median day in 1 to 20 Nov (R$23k) and about 15% of the whole month. Revenue stayed elevated in the days after and mostly above early-November levels through December, so the rise reflects the Black Friday spike plus a general seasonal lift. Monthly revenue then settled around R$0.83 to 0.98M through 2018.
- Jan to Aug 2018 revenue was about 2.4x the same months of 2017 (from a small early-2017 base).

![Monthly revenue](reports/monthly_revenue.png)
![Top categories](reports/top_categories.png)
![Daily revenue around Black Friday](reports/black_friday_daily.png)
