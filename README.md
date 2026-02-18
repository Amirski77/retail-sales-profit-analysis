# 🛒 Retail Sales Analysis & Profit Forecasting

> **End-to-end analytics pipeline (SQL → Power BI → Python ML) on 2,120 U.S. retail transactions spanning 2014–2017.
> Core discovery: the Discount coefficient (−359.20) is ~7,200× stronger than the Sales coefficient (+0.05) — proving that aggressive discounting is the single largest driver of profit destruction in this business.**

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
| **scikit-learn** | Machine learning — `LinearRegression` model training & prediction |

---

## 🔄 Pipeline Architecture

```
┌───────────────────────┐     ┌───────────────────────┐     ┌───────────────────────┐     ┌───────────────────────┐
│       RAW DATA        │────▶│    GOOGLE BIGQUERY   │────▶│   PYTHON (Jupyter)    │────▶│       POWER BI        │
│        4 CSVs         │     │                       │     │                       │     │       DASHBOARD       │
│                       │     │  • Import & audit     │     │  • EDA plots          │     │                       │
│  customers            │     │  • Deduplicate        │     │  • Distribution       │     │  • 7+ business        │
│  products             │     │    (QUALIFY +         │     │    analysis           │     │    questions          │
│  orders               │     │     ROW_NUMBER)       │     │  • Linear             │     │  • Drill-through      │
│  sales                │     │  • JOIN into 1        │     │    Regression         │     │  • Actual vs.         │
│                       │     │    unified table      │     │  • Export             │     │    Predicted          │
│                       │     │  • Export CSV         │     │    forecast.csv       │     │    profit overlay     │
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
Training data   :  2,120 transactions (full dataset)
```

#### 📋 Model Coefficients

*All values verified from notebook output (Cell 10):*

| Symbol | Feature | Coefficient | What It Means in Business Terms |
|:------:|:--------|:-----------:|:--------------------------------|
| β₀ | *Intercept* | **+59.79** | Baseline expected profit of ~$60 before any feature effects |
| β₁ | `Sales` | **+0.05** | Every additional $1 in sale amount adds $0.05 to profit |
| β₂ | `Discount` | **−359.20** | A 10% discount increase (0.1) cuts profit by **$35.92** per order |
| β₃ | `Quantity` | **−1.21** | Each additional unit sold reduces profit by $1.21 (margin dilution) |

> 🚨 **Key Finding:**
>
> The Discount coefficient (−359.20) is **~7,200× larger in magnitude** than the Sales coefficient (+0.05).
>
> In practical terms: a 20% discount wipes out **$71.84** per transaction — exceeding the $59.79 baseline entirely. This means **any order with a 20%+ discount starts in the red before other factors even apply.**

#### 🔄 Forecast Pipeline

```
Training data (2,120 rows)
        │
        ▼
LinearRegression.fit(X, y)
        │
        ▼
model.predict(X) → new column: Predicted_Profit
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
| 1 | **Discounts above 20% nearly always produce negative profit** | Model coefficient β₂ = −359.20; at 20% discount, the loss (−$71.84) exceeds the baseline profit (+$59.79) | Cap discounts at 15–20%; require margin approval for anything above |
| 2 | **Tables & Bookcases are consistently sold at a loss** | Sub-category profit analysis shows persistent negative profit totals | Renegotiate supplier terms or phase out lowest-margin SKUs |

### 📊 Strategic — Planning & Optimization

| # | Insight | Supporting Evidence | Recommendation |
|:-:|:--------|:-------------------|:---------------|
| 3 | **Technology leads revenue; Office Supplies lead margin consistency** | Category-level profit breakdown across all 4 years | Protect Office Supplies margins while scaling Technology volume |
| 4 | **West & East regions outperform Central & South** | Regional profit aggregation in SQL + Power BI drill-through | Investigate underperformance drivers — logistics, demand, or competition |
| 5 | **Q4 seasonal spike every year (Nov–Dec)** | Monthly revenue line chart shows consistent pattern 2014–2017 | Pre-stock high-demand categories; time promotions around the seasonal peak |
| 6 | **Standard Class shipping dominates volume with solid margins** | Ship Mode profit comparison visualization | Maintain as default option; evaluate whether premium tiers justify their cost |

---

## Limitations & Future Work

### Honest Assessment of Current Scope

| Limitation | Why It Matters | How I Would Fix It |
|:-----------|:---------------|:-------------------|
| **No train/test split** | The model was trained and evaluated on the same 2,120 rows, so accuracy on unseen data is unknown | Apply `train_test_split` (80/20) and report R², MAE, RMSE on the held-out test set |
| **No evaluation metrics** | Coefficients alone don't tell you how well the model *fits* — R² and residual analysis are standard practice | Add `r2_score`, `mean_absolute_error`, `mean_squared_error` from sklearn.metrics |
| **Linear model only** | Assumes a perfectly linear discount→profit relationship; real dynamics may have non-linear thresholds | Test **Random Forest** or **XGBoost** to capture interaction effects |
| **No external features** | The model doesn't account for seasonality, customer lifetime value, or product cost structure | Engineer time-based and customer-based features for richer predictions |

### Planned Next Steps

- 🔄 Re-train with proper **train/test split** and publish evaluation metrics
- 🌲 Benchmark against **tree-based models** (Random Forest, XGBoost)
- 📅 Add **time-series features** (month, quarter, year-over-year growth)
- 👥 Build a **customer segmentation model** (RFM analysis) to identify high-value vs. at-risk customers
- ☁️ Deploy the Power BI dashboard to **Power BI Service** for live stakeholder access

---

## 🚀 How to Reproduce

### 1️⃣ SQL
Import the four source CSV files into **Google BigQuery** and execute the queries in `sql/queries.sql`.

### 2️⃣ Python
```bash
pip install pandas matplotlib seaborn scikit-learn
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
