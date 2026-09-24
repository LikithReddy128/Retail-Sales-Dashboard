# Retail-Sales-Dashboard
A 3-page Power BI report analyzing retail transactions across region, city, product category, brand, and sales channel — built on a star-schema data model (`Fact Sales` + 4 dimension tables) with time-intelligence measures (YTD, MTD, current vs. previous period) and five synced slicers.
![Power BI](https://github.com/LikithReddy128/Retail-Sales-Dashboard/blob/main/Retails%20Store%20Dashboard.pbix)
![Excel](https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-Measures-blue?style=for-the-badge)

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Dataset](#dataset)
4. [Data Model](#data-model)
5. [KPIs & DAX Measures](#kpis--dax-measures)
6. [Dashboard Pages](#dashboard-pages)
7. [Slicers Used & How to Add Them](#slicers-used--how-to-add-them)
8. [How to Open / Use This Project](#how-to-open--use-this-project)
9. [How This Repository Was Created (Git & GitHub Steps)](#how-this-repository-was-created-git--github-steps)
10. [Insights & Outcomes](#insights--outcomes)
11. [Tech Stack](#tech-stack)
12. [Author](#author)

---

## Project Overview

**Title:** Retail Sales Dashboard

**Aim / Goal:** Give retail leadership a single interactive report to monitor total sales, profit, orders, customers, and product reach — sliced by region, city, category and time — and to spot growth trends (current vs. previous year/month, YTD/MTD) without manually pivoting the raw sales export.

**Business questions this dashboard is designed to answer:**
1. If you were a retail manager, which region would you prioritize for investment, and why?
2. How could the company leverage online sales growth to improve profitability?
3. What risks might arise from having sales concentrated in certain regions or categories?
4. How might seasonal trends (the Month filter) affect interpretation of the dashboard?
5. What additional metrics would make this dashboard more actionable?

(See [Insights & Outcomes](#insights--outcomes) for candidate answers based on the current data.)

---

## Repository Structure

```
Retail-Sales-Dashboard/
│
├── README.md                        # You are here – full project summary
├── Documentation.md                 # Detailed write-up: aim, cleaning steps, results, insights
│
├── original-data/
│   └── zepto_sales_raw.xlsx         # Raw/source sales export used to build the model
│
├── data-model/
│   ├── conceptual-model.png         # High-level entity relationship view
│   └── physical-data-model.png      # Power BI model view (tables, columns, relationships)
│
├── power-bi-file/
│   └── Retails_Project_1.pbix       # The working Power BI report
│
└── screenshots/
    ├── overview-page.png            # Page 1 – Overview
    ├── time-series-analysis-page.png  # Page 2 – Time Series Analysis
    └── ytd-mtd-page.png             # Page 3 – YTD / MTD Trends
```

> This mirrors the standard set of items expected in a Power BI GitHub submission: **original data, conceptual model, physical data model, the .pbix file, documentation, and a README.**

---

## Dataset

| Property | Detail |
|---|---|
| Raw file | `zepto_sales_raw.xlsx` |
| Loaded into model as | `Fact Sales` (transactions) + 4 lookup/dimension tables |
| Core measures available | Sales, Profit, Quantity, Orders, Customers, Products |
| Time grain | Order/Sale date, rolled up to Month / Quarter / Year |

The raw export was shaped in Power Query into a clean **star schema** (one fact table surrounded by dimension tables) before being loaded into the Power BI model — see [Data Model](#data-model) below.

---

## Data Model

The model follows a classic **star schema**:

```
                Dim Customer (Region, City)
                        │
Dim Product ── Fact Sales ── Dim Calender (MONTH, YEAR, QUARTER)
(Category,             │
 Brand,           Dim Store
 ProductID)        (Channel)
```

**`Fact Sales`** (fact table) — one row per transaction line, holding the numeric values that every measure aggregates: Sales, Profit, Quantity, Order count, Customer count, Product count.

**`Dim Customer`** — `Region` (East, North, South, West), `City` (Bangalore, Chennai, Delhi, Kolkata, Mumbai).

**`Dim Product`** — `Category` (Beauty, Clothing, Electronics, Grocery, Home), `Brand` (Brand A–E), `ProductID` (e.g. P50, P18, P132…).

**`Dim Calender`** — `MONTH` (1–12), `YEAR` (2022–2025), `QUARTER` — a standalone date table used for all time-intelligence measures.

**`Dim Store`** — `Channel` (Online, Store).

**Relationships:** each dimension table has a one-to-many relationship into `Fact Sales` on its respective key (Customer key, Product key, Date, Store/Channel key) — single-direction filtering from dimension → fact, the standard Power BI star-schema pattern.

- **Conceptual Model** (`data-model/conceptual-model.png`): shows `Fact Sales` as the central transaction entity surrounded by the four dimensions above.
- **Physical Data Model** (`data-model/physical-data-model.png`): the actual Power BI **Model view** screenshot, showing the tables, columns, and the four one-to-many relationships.

> *Export both images from Power BI (Model view, and a hand-drawn/whiteboard-style conceptual diagram from draw.io or PowerPoint) and save them into `data-model/` using the exact file names above.*

---

## KPIs & DAX Measures

**Headline KPI cards** (shown on every page):

| KPI Card | Value (sample) | DAX pattern |
|---|---|---|
| Total Sales | 49.64M | `Total Sales = SUM(Fact Sales[Sales])` |
| Total Profit | 11.23M | `Total Profit = SUM(Fact Sales[Profit])` |
| Total Orders | 5K | `Total Orders = DISTINCTCOUNT(Fact Sales[OrderID])` |
| Total Customers | 500 | `Total Customers = DISTINCTCOUNT(Fact Sales[CustomerID])` |
| Total Products | 200 | `Total Products = DISTINCTCOUNT(Dim Product[ProductID])` |

**Time-intelligence measures** (page 2 & 3):

| Measure | Purpose | DAX pattern |
|---|---|---|
| Current Year Sales / Previous Year Sales | Year-over-year sales comparison | `CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Dim Calender'[Date]))` |
| Current Month Profit / Previous Month Profit | Month-over-month profit comparison | `CALCULATE([Total Profit], PREVIOUSMONTH('Dim Calender'[Date]))` |
| Total Sales(%) / Total Profit(%) | Share of sales/profit by category | `DIVIDE([Total Sales], CALCULATE([Total Sales], ALL('Dim Product'[Category])))` |
| Online Sales | Sales filtered to the Online channel | `CALCULATE([Total Sales], 'Dim Store'[Channel] = "Online")` |
| TotalYTD / Previous Year(py) | Year-to-date running total, and its prior-year equivalent | `TOTALYTD([Total Sales], 'Dim Calender'[Date])` |
| TotalMTD(Sales) / previous month (profit) | Month-to-date running total | `TOTALMTD([Total Sales], 'Dim Calender'[Date])` |

---

## Dashboard Pages

**Page 1 — Overview**
- KPI cards: Total Sales, Total Profit, Total Orders, Total Customers, Total Products
- Total Sales by Region (column chart)
- Total Sales by Brand and City (clustered bar chart)
- Sum of Sales by Channel (donut: Online vs. Store)
- Total Profit by Region (pie chart)
- Category table: Total Profit, Total Sales, Sum of Quantity by Category
- Total Sales by ProductID (bar chart)

**Page 2 — Time Series Analysis**
- Same KPI cards + slicer panel
- Sales: Current Year vs Previous Year (clustered column, by Year)
- Profit: Current Month vs Previous Month (clustered column, by Month)
- Total Sales(%) and Total Profit(%) by Category (clustered column)
- Online Sales by Quarter (line chart)

**Page 3 — YTD / MTD Trends** *(internally named "Duplicate of Time Series Analysis" — recommend renaming the page tab to "YTD / MTD Trends" for clarity)*
- Same KPI cards + slicer panel
- TotalYTD vs Previous Year(py) by Year (line chart)
- TotalMTD(Sales) vs Previous Month Profit by Month (line chart)
- totalMTD (profit) vs previous month (profit) by Month (line chart)

---

## Slicers Used & How to Add Them

**Slicers included on every page (synced):**
- **Region** (East, North, South, West) — from `Dim Customer`
- **City** (Bangalore, Chennai, Delhi, Kolkata, Mumbai) — from `Dim Customer`
- **Category** (Beauty, Clothing, Electronics, Grocery, Home) — from `Dim Product`
- **Month** (1–12, range slider) — from `Dim Calender`
- **Year** (2022–2025, range slider) — from `Dim Calender`

### Step-by-step: how to add a slicer in Power BI Desktop
1. Open the `.pbix` file in **Power BI Desktop**.
2. On the report canvas, go to the **Visualizations** pane and click the **Slicer** icon.
3. With the empty slicer placeholder selected, drag the field you want to filter by (e.g. `Dim Customer[Region]`) from the **Data** pane into the **Field** well.
4. Resize/position the slicer in the right-hand rail, matching the layout in the screenshots.
5. Format it: select the slicer → **Format visual** pane →
   - **Slicer settings → Options** to choose *List*, *Dropdown*, or *Tile* style (checkbox-list style was used for Region/City/Category; a **Between** range slider was used for Month and Year).
   - **Selection** controls to allow single- or multi-select.
6. Repeat for `City`, `Category`, `Month`, and `Year`.
7. To make slicers apply across all three report pages, select each slicer → **View** ribbon → **Sync slicers** → tick **Overview**, **Time Series Analysis**, and **YTD / MTD Trends**.
8. For `Month` and `Year`, set the slicer visual type to **Between** (a numeric range slider) so users can drag both ends of the range, matching the dashboard screenshots.
9. Save the file (`Ctrl + S`).

---

## How to Open / Use This Project

1. Install **Power BI Desktop** (free, from the Microsoft Store or [powerbi.microsoft.com](https://powerbi.microsoft.com/desktop/)).
2. Clone or download this repository.
3. Open `power-bi-file/Retails_Project_1.pbix`.
4. If prompted, point the data source to `original-data/zepto_sales_raw.xlsx` on your local machine (Home → Transform data → Data source settings → Change Source).
5. Click **Refresh** on the Home ribbon to load the latest data.
6. Use the Region, City, Category, Month, and Year slicers to explore Overview, Time Series Analysis, and YTD/MTD pages.

---

## How This Repository Was Created (Git & GitHub Steps)

1. **Create the repository on GitHub**
   - Go to [github.com](https://github.com) → click **New repository**.
   - Name it, e.g., `Retail-Sales-Dashboard`.
   - Add a short description ("Power BI retail sales dashboard – region, category, and channel performance with YTD/MTD trends").
   - Choose **Public**, initialize with a `README.md`, and (optionally) a `.gitignore` for Power BI (`*.pbix.bak`) — click **Create repository**.

2. **Clone it locally**
   ```bash
   git clone https://github.com/<your-username>/Retail-Sales-Dashboard.git
   cd Retail-Sales-Dashboard
   ```

3. **Add the project folders/files**
   ```bash
   mkdir original-data data-model power-bi-file screenshots
   # copy zepto_sales_raw.xlsx into original-data/
   # copy conceptual-model.png and physical-data-model.png into data-model/
   # copy the .pbix file into power-bi-file/
   # copy the three page screenshots into screenshots/
   ```

4. **Add the README and Documentation files** (this file, and `Documentation.md`) to the repository root.

5. **Stage, commit, and push**
   ```bash
   git add .
   git commit -m "Add Retail Sales Dashboard: data, model, Power BI file, and documentation"
   git push origin main
   ```

6. **Verify on GitHub** that all six items render correctly: raw data, both model images, the `.pbix` file (GitHub shows it as a downloadable binary), and both markdown files.

7. **(Optional) Add topics/tags** on the repo page — e.g. `power-bi`, `data-analytics`, `retail-analytics`, `dax`, `dashboard`, `star-schema` — to make the project discoverable.

---

## Insights & Outcomes

**KPI snapshot:** 49.64M total sales, 11.23M total profit, 5K orders, 500 customers, 200 products across the reporting period.

**Candidate answers to the reflection questions:**

1. **Which region to prioritize?** South leads on both sales (tallest bar in "Total Sales by Region") and profit share (42% of total profit, the largest single slice) — it's the strongest candidate for further investment, since it's already outperforming East, North, and West by a wide margin.
2. **Leveraging online sales growth?** Online and in-store channels are almost evenly split (49.4% vs. 50.6% of sales). Because online typically carries lower fixed overhead, shifting incremental growth toward online (targeted promotions, faster delivery slots, loyalty offers) is a low-risk way to lift overall margin.
3. **Concentration risk?** Category performance is fairly balanced (Clothing, Electronics, and Home each ~23–24M profit), but Beauty trails noticeably (18.7M) and South's regional dominance means a downturn there would disproportionately hurt total results — a case for deliberately growing the weaker regions/categories rather than over-relying on the leaders.
4. **Seasonality caveat?** Because Month and Year are range slicers, a user comparing two arbitrary periods could unintentionally compare a peak month against an off-peak month — the Current-vs-Previous-Year/Month measures are built specifically to control for this by always comparing like-for-like periods.
5. **Additional metrics to consider:** Average Order Value, profit margin % (Profit ÷ Sales) by region/category, customer repeat-purchase rate, and category-level YoY growth rate would all make the dashboard more directly actionable for planning decisions.

---

## Tech Stack
- **Power BI Desktop** – data modeling, DAX, visualization
- **Excel** – source data (`zepto_sales_raw.xlsx`)
- **DAX** – KPI measures and time-intelligence calculations (YTD, MTD, YoY, MoM)
- **Git & GitHub** – version control and project publishing

---

## Author
Maintained as part of a retail analytics portfolio project. Feel free to fork this repository and adapt the model/measures to your own sales dataset.
