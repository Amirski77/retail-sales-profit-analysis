# 🛒 Retail Sales Analysis & Profit Forecasting

> **End-to-end analytics pipeline (SQL → Power BI → Python ML) on 2,120 U.S. retail transactions spanning 2014–2017.
> Core discovery: the Discount coefficient (−346.23) is ~8,600× stronger than the Sales coefficient (+0.04) — proving that aggressive discounting is the single largest driver of profit destruction in this business.**

---

## 📌 Project Overview

This project tackles a real business problem: **What factors most influence a retail company's profitability — and can we predict profit per transaction before the sale happens?**

The analysis follows the full data lifecycle — from raw data ingestion and SQL-based cleaning, through interactive Power BI dashboards, to a Linear Regression model in Python — mirroring the workflow of a professional analytics team.

### 🎯 Objectives

- Identify top-performing products, regions, and customer segments
- Quantify the **true cost of discounting** on profit margins
- Detect seasonal trends and revenue patterns across 4 years of data
- Build a **Linear Regression model** to predict per-transaction profit
- Deliver actionable recommendations through **interactive Power BI dashboards**

---

## 📐 Data at a Glance

| Metric | Value |
|:-------|:------|
| **Total Transactions** | 2,120 rows |
| **Time Period** | 2014 – 2017 (4 years) |
| **Geography** | United States — 4 regions (East, West, Central, South) |
| **Features** | 22 columns across 4 source tables |
| **Null Values** | 0 across all columns |
| **Target Variable** | `Profit` (continuous, USD) |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|:-----|:--------|
| **Google BigQuery** | Cloud data warehouse — storage, SQL querying, table joins & deduplication |
| **SQL** | Exploratory analysis — aggregations, `WINDOW` functions, `CTEs`, `QUALIFY` |
| **Power BI** | Interactive dashboards — drill-through reports, dynamic filters, KPI cards |
| **Python** | EDA, data preprocessing, Linear Regression modeling |
| **pandas · matplotlib · seaborn** | Data manipulation & statistical visualization |
| **scikit-learn** | Machine learning — `LinearRegression`, `train_test_split`, evaluation metrics |

---

## 🔄 Pipeline Architecture

```
┌───────────────────────┐     ┌───────────────────────┐     ┌───────────────────────┐     ┌───────────────────────┐
│       RAW DATA        │────▶│    GOOGLE BIGQUERY    │────▶│   PYTHON (Jupyter)    │────▶│       POWER BI        │
│        4 CSVs         │     │                       │     │                       │     │       DASHBOARD       │
│                       │     │  • Import & audit     │     │  • EDA plots          │     │                       │
│  customers            │     │  • Deduplicate        │     │  • Distribution       │     │  • 7+ business        │
│  products             │     │    (QUALIFY +         │     │    analysis           │     │    questions          │
│  orders               │     │     ROW_NUMBER)       │     │  • Train/Test split   │     │  • Drill-through      │
│  sales                │     │  • JOIN into 1        │     │  • Linear Regression  │     │  • Actual vs.         │
│                       │     │    unified table      │     │  • R², MAE, RMSE      │     │    Predicted          │
│                       │     │  • Export CSV         │     │  • forecast.csv       │     │    profit overlay     │
└───────────────────────┘     └───────────────────────┘     └───────────────────────┘     └───────────────────────┘
```

---

## 📂 Project Structure

```
📁 retail-sales-analysis/
│
├── 📄 README.md                     # You are here
│
├── 📂 sql/
│   └── queries.sql                  # All SQL queries (BigQuery-compatible)
│
├── 📂 python/
│   └── final_forecast_csv.ipynb     # Jupyter Notebook — EDA + regression model
│
├── 📂 powerbi/
│   └── Final_Project.pbix           # Power BI interactive report
│
├── 📂 data/
│   ├── final_project_query.csv      # Unified dataset exported from BigQuery
│   └── forecast.csv                 # Dataset with Predicted_Profit column added
│
└── 📂 docs/
    └── presentation.pdf             # Final project presentation slides
```

---

## 🗄️ Data Model

The source data is organized as **4 normalized tables**:

| Table | Description | Key Fields |
|:------|:------------|:-----------|
| `customers` | Client profiles | `Customer ID`, Name, Segment, Region, City |
| `products` | Product catalog | `Product ID`, Category, Sub-Category, Product Name |
| `orders` | Order metadata | `Order ID`, Order Date, Ship Date, Ship Mode |
| `sales` | Transaction details | `Order ID`, `Product ID`, Sales, Quantity, Discount, Profit |

> All four tables were joined into a **single unified analytics table** using `CTEs` with `QUALIFY ROW_NUMBER()` deduplication in BigQuery — serving as the single source of truth for all downstream analysis.

---

## 📊 Part 1 — SQL Analysis (Google BigQuery)

### 🔧 Data Preparation

- Imported all four CSV source tables into BigQuery
- Validated data types and checked for duplicates across every table
- Built a clean, unified table using `QUALIFY ROW_NUMBER() OVER(PARTITION BY ...)` window functions combined with multi-table `LEFT JOIN`

### 🔍 Analytical Queries

| # | Query | SQL Technique | Business Purpose |
|:-:|:------|:-------------|:-----------------|
| 1 | **Top 5 Products by Revenue** | `SUM()` + `GROUP BY` + `ORDER BY DESC` | Identify revenue drivers for investment prioritization |
| 2 | **Average Discount by Region** | `AVG()` + `GROUP BY` | Reveal which regions over-discount |
| 3 | **Top 10 Most Loyal Customers** | `COUNT(DISTINCT Order ID)` ranking | Target high-value customers for retention programs |
| 4 | **Profit by Sub-Category** | `SUM(Profit)` grouped & sorted | Expose sub-categories that drain profitability |
| 5 | **Discounted Sales Share** | Conditional aggregation — `CASE WHEN` | Quantify what share of revenue depends on discounts |
| 6 | **Unified Master Table** | 4-table `JOIN` via `CTEs` with dedup | Create single source of truth for Python & Power BI |

<details>
<summary>💻 <b>Click to view SQL — Unified Table Creation</b></summary>

<br>

```sql
CREATE OR REPLACE TABLE `da-nfactorial.student_Amir_Rashidov.final_table` AS
WITH Clean_Orders AS (
  SELECT `Order ID`, `Order Date`, `Ship Date`, `Ship Mode`, `Customer ID`
  FROM `da-nfactorial.student_Amir_Rashidov.final_orders`
  QUALIFY ROW_NUMBER() OVER(PARTITION BY `Order ID` ORDER BY `Order Date` DESC) = 1
),
Clean_Customers AS (
  SELECT *
  FROM `da-nfactorial.student_Amir_Rashidov.final_customers`
  QUALIFY ROW_NUMBER() OVER(PARTITION BY `Customer ID` ORDER BY `Customer ID`) = 1
),
Clean_Products AS (
  SELECT *
  FROM `da-nfactorial.student_Amir_Rashidov.final_products`
  QUALIFY ROW_NUMBER() OVER(PARTITION BY `Product_ID` ORDER BY `Product_ID`) = 1
),
Clean_Sales AS (
  SELECT DISTINCT *
  FROM `da-nfactorial.student_Amir_Rashidov.final_sales`
)
SELECT
    s.`Order ID`,
    o.`Order Date`,
    EXTRACT(YEAR FROM CAST(o.`Order Date` AS DATE)) AS Year,
    EXTRACT(MONTH FROM CAST(o.`Order Date` AS DATE)) AS Month,
    o.`Ship Date`,
    o.`Ship Mode`,
    o.`Customer ID`,
    c.`Customer Name`,
    c.Segment,
    c.Country,
    c.City,
    c.State,
    c.`Postal Code`,
    c.Region,
    s.`Product ID`,
    p.Category,
    p.`Sub_Category`,
    p.`Product_Name`,
    s.Sales,
    s.Quantity,
    s.Discount,
    s.Profit
FROM Clean_Sales s
LEFT JOIN Clean_Orders o    ON s.`Order ID` = o.`Order ID`
LEFT JOIN Clean_Customers c ON o.`Customer ID` = c.`Customer ID`
LEFT JOIN Clean_Products p  ON s.`Product ID` = p.`Product_ID`;
```

<br>

</details>

---

## 📈 Part 2 — Power BI Dashboard

Built a **multi-page interactive report** answering **7+ business questions**:

| Business Question | Visualization Type |
|:------------------|:-------------------|
| Which categories are most profitable per region? | Stacked bar chart with region slicer |
| Does discount size correlate with profit loss? | Scatter plot + trend line |
| Are there seasonal revenue patterns? | Line chart — monthly revenue by year |
| Which cities drive the most revenue? | Map / ranked bar chart |
| Which sub-categories get the heaviest discounts? | Horizontal bar chart |
| Are any products consistently sold at a loss? | Conditional formatting table |
| How does shipping method affect profit? | Grouped bar chart |

### ⚙️ Dashboard Features

| Feature | Description |
|:--------|:------------|
| 🔽 **Dynamic Filters** | Filter by Year, Month, Segment, and Region |
| 🔎 **Drill-Through** | Click any region → opens a detailed breakdown page |
| 🏷️ **Discount Toggle** | Isolate only discounted transactions |
| 📦 **Category Tracker** | Select a product category → track its sales & profit over time |
| 🤖 **Forecast Overlay** | Actual vs. Predicted profit side-by-side (data from Python model) |
| 🧭 **Page Navigation** | Button-based navigation across report sections |

---

## 🐍 Part 3 — Python: EDA & Predictive Modeling

### Exploratory Data Analysis (EDA)

| Analysis | Finding |
|:---------|:--------|
| **Sales Distribution** | Right-skewed — most transactions fall in the $0–$500 range |
| **Profit Distribution** | Centered near $0 with significant negative outliers (losses up to −$500) |
| **Discount Distribution** | Bimodal — heavily concentrated at 0% and 20% discount levels |
| **Discount vs. Profit** | Clear negative correlation — higher discounts consistently erode profit |

### 🤖 Linear Regression Model

**Objective:** Predict `Profit` for each transaction based on order characteristics.

```
Target variable :  Profit
Features        :  Sales, Discount, Quantity
Algorithm       :  sklearn.linear_model.LinearRegression
Split           :  80% train (1,696 rows) / 20% test (424 rows)
random_state    :  42
```

#### 📋 Model Coefficients

*All values from the train/test split model (notebook Cell 12):*

| Symbol | Feature | Coefficient | What It Means in Business Terms |
|:------:|:--------|:-----------:|:--------------------------------|
| β₀ | *Intercept* | **+60.22** | Baseline expected profit of ~$60 before any feature effects |
| β₁ | `Sales` | **+0.04** | Every additional $1 in sale amount adds $0.04 to profit |
| β₂ | `Discount` | **−346.23** | A 10% discount increase (0.1) cuts profit by **$34.62** per order |
| β₃ | `Quantity` | **−0.77** | Each additional unit sold reduces profit by $0.77 (margin dilution) |

> 🚨 **Key Finding:**
>
> The Discount coefficient (−346.23) is **~8,600× larger in magnitude** than the Sales coefficient (+0.04).
>
> In practical terms: a 20% discount wipes out **$69.25** per transaction — exceeding the $60.22 baseline entirely. This means **any order with a 20%+ discount starts in the red before other factors even apply.**

#### 📊 Model Evaluation (Test Set — 424 Unseen Transactions)

| Metric | Value | Interpretation |
|:-------|:-----:|:---------------|
| **R² Score** | **0.3513** | The model explains ~35% of profit variance using only 3 features |
| **MAE** | **$59.34** | On average, predictions are off by ~$59 |
| **RMSE** | **$106.07** | Larger errors are penalized more heavily; outlier impact is significant |

> **What does R² = 0.35 mean?** The model captures the dominant directional trends (discounts → loss, higher sales → gain) but 65% of profit variance comes from factors not in the model — such as product cost structure, customer negotiation, regional pricing, and seasonality. This is expected with only 3 features, and the coefficient insights remain highly valuable for business decision-making even though the predictive accuracy is moderate.

#### 🔄 Forecast Pipeline

```
Full dataset (2,120 rows)
        │
        ├── 80% train (1,696 rows)
        │         │
        │         ▼
        │   LinearRegression.fit(X_train, y_train)
        │         │
        │         ▼
        │   Evaluate on 20% test (424 rows)
        │   R² = 0.35 | MAE = $59 | RMSE = $106
        │
        ▼
model.predict(full dataset) → Predicted_Profit column
        │
        ▼
Export to forecast.csv
        │
        ▼
Import into Power BI → Actual vs. Predicted profit overlay
```

---

## 💡 Key Business Insights

> *Findings prioritized by business impact — what a retail executive would act on first.*

### 🚨 Critical — Immediate Action Required

| # | Insight | Supporting Evidence | Recommended Action |
|:-:|:--------|:-------------------|:-------------------|
| 1 | **Discounts above 20% nearly always produce negative profit** | β₂ = −346.23; at 20% discount, the loss (−$69.25) exceeds the baseline profit (+$60.22) | Cap discounts at 15–20%; require margin approval for anything above |
| 2 | **Tables & Bookcases are consistently sold at a loss** | Sub-category profit analysis shows persistent negative profit totals | Renegotiate supplier terms or phase out lowest-margin SKUs |

### 📊 Strategic — Planning & Optimization

| # | Insight | Supporting Evidence | Recommendation |
|:-:|:--------|:-------------------|:---------------|
| 3 | **Technology leads revenue; Office Supplies lead margin consistency** | Category-level profit breakdown across all 4 years | Protect Office Supplies margins while scaling Technology volume |
| 4 | **West & East regions outperform Central & South** | Regional profit aggregation in SQL + Power BI drill-through | Investigate underperformance drivers — logistics, demand, or competition |
| 5 | **Q4 seasonal spike every year (Nov–Dec)** | Monthly revenue line chart shows consistent pattern 2014–2017 | Pre-stock high-demand categories; time promotions around the seasonal peak |
| 6 | **Standard Class shipping dominates volume with solid margins** | Ship Mode profit comparison visualization | Maintain as default option; evaluate whether premium tiers justify their cost |

---

## ⚠️ Limitations & Future Work

### Honest Assessment of Current Scope

| Limitation | Why It Matters | How I Would Address It |
|:-----------|:---------------|:-----------------------|
| **R² = 0.35 (moderate fit)** | The model captures the right directional trends but misses 65% of profit variance | Engineer additional features: product cost, customer lifetime value, time-based variables |
| **Linear model only** | Assumes a perfectly linear discount→profit relationship; real dynamics may have non-linear thresholds | Benchmark against **Random Forest** and **XGBoost** to capture interaction effects |
| **No time-series features** | Seasonality (Q4 spikes) is visible in EDA but not encoded in the regression model | Add month, quarter, and year-over-year growth as features |
| **Single dataset scope** | Results are specific to this U.S. retail dataset; generalizability is untested | Validate on a different time period or retail dataset |

### Planned Next Steps

- 🌲 Benchmark against **tree-based models** (Random Forest, XGBoost) to improve R²
- 📅 Add **time-series features** (month, quarter, YoY growth) to capture seasonality
- 👥 Build a **customer segmentation model** (RFM analysis) to identify high-value vs. at-risk customers
- 🔁 Implement **cross-validation** (5-fold) for more robust performance estimates
- ☁️ Deploy the Power BI dashboard to **Power BI Service** for live stakeholder access

---

## 🚀 How to Reproduce

### 1️⃣ SQL
Import the four source CSV files into **Google BigQuery** and execute the queries in `sql/queries.sql`.

### 2️⃣ Python
```bash
pip install pandas matplotlib seaborn scikit-learn numpy
```
Open `final_forecast_csv.ipynb` in **Jupyter Notebook** or **Google Colab**.
Make sure `final_project_query.csv` is in your working directory.

### 3️⃣ Power BI
Open `Final_Project.pbix` in **Power BI Desktop** (free download from Microsoft).
Reconnect data sources if prompted.

---
## 👤 Author
**Amir Rashidov**

🎓 Data Analytics Student
📧 *amirrashidov14@gmail.com*
💼 www.linkedin.com/in/amir-r-673789203
---
