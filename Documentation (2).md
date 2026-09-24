# Documentation – Retail Sales Dashboard

This document provides the detailed project write-up referenced from the main `README.md`: the project aim, dataset description, data-cleaning process, modeling decisions, final results, and insights.

---

## 1. Project Aim / Goal

To design and build a Power BI dashboard that gives retail leadership a fast, visual, self-service way to monitor overall sales and profit performance, compare regions/categories/channels, and track growth over time (year-over-year, month-over-month, YTD, MTD) — replacing manual spreadsheet pivoting with interactive filtering.

**Target audience:** Retail operations managers, regional managers, category/merchandising teams.

**Success criteria:**
- Every KPI card (Total Sales, Total Profit, Total Orders, Total Customers, Total Products) is visible on every page.
- Users can slice by Region, City, Category, Month, and Year, synced across all pages.
- Growth is visible at a glance via YoY, MoM, YTD, and MTD comparisons.

---

## 2. The Dataset Description

- **Raw source file:** `original-data/zepto_sales_raw.xlsx`.
- **Grain (raw file):** one row per order line item, including order ID, order date, customer, product/SKU, category, quantity, price, discount, tax, delivery fee, and total.
- **Grain (as modeled in Power BI):** the raw export was reshaped in Power Query into a **star schema** — a central `Fact Sales` transaction table connected to four dimension tables (`Dim Customer`, `Dim Product`, `Dim Calender`, `Dim Store`) — which is the structure the finished report actually queries.
- **Key dimensions surfaced in the dashboard:**
  - Region: East, North, South, West
  - City: Bangalore, Chennai, Delhi, Kolkata, Mumbai
  - Category: Beauty, Clothing, Electronics, Grocery, Home
  - Brand: Brand A–E
  - Channel: Online, Store
  - Time: Month (1–12), Year (2022–2025), Quarter

---

## 3. Data Cleaning Process

The following steps were carried out in Power Query before the data was loaded into the model:

1. **Header validation** – confirmed each source column had a single, unambiguous header (Order ID, Order Date, Customer, City, Category, Product, Quantity, Price, Discount, Tax, Fees, Total, etc.) with no merged cells above the header row.
2. **Data type correction** – set dates to Date type, monetary/quantity fields to decimal or whole numbers, and category-style fields (Region, City, Category, Channel) to Text, so Power BI does not misread any column.
3. **Date parsing/standardization** – the raw `Order_Date` column contained mixed formats (e.g. `2025-02-08` and `15 Jan 2025`); these were standardized into a single Date column feeding the `Dim Calender` table.
4. **Duplicate check** – checked Order ID for exact duplicates to confirm each row represents a distinct transaction line.
5. **Blank/null check** – scanned key fields (Order ID, Date, Category, Total) for blanks; incomplete rows were flagged/excluded so they don't distort KPI totals.
6. **Category standardization** – trimmed whitespace and normalized casing on categorical fields (Region, City, Category, Channel, Brand) so, for example, "online" and "Online" are not treated as two different channels.
7. **Outlier sanity check** – reviewed Price, Discount, and Total for values outside plausible ranges (e.g. negative totals, discount greater than subtotal).
8. **Star-schema extraction** – split the single flat export into:
   - `Dim Customer` (Region, City, Customer key)
   - `Dim Product` (Category, Brand, ProductID)
   - `Dim Calender` (Date, Month, Year, Quarter — a standalone date table for time intelligence)
   - `Dim Store` (Channel)
   - `Fact Sales` (Sales, Profit, Quantity, and the foreign keys linking back to each dimension)
9. **Relationships built** – one-to-many relationships from each dimension table to `Fact Sales`, enabling all slicers to filter every visual on the report.

---

## 4. Data Model Summary

- **Model type:** star schema — one fact table (`Fact Sales`) surrounded by four dimension tables.
- **Relationships:** `Dim Customer` → `Fact Sales`, `Dim Product` → `Fact Sales`, `Dim Calender` → `Fact Sales`, `Dim Store` → `Fact Sales` (all one-to-many, single-direction filtering).
- **Core measures (DAX):**
  ```
  Total Sales      = SUM(Fact Sales[Sales])
  Total Profit     = SUM(Fact Sales[Profit])
  Total Orders     = DISTINCTCOUNT(Fact Sales[OrderID])
  Total Customers  = DISTINCTCOUNT(Fact Sales[CustomerID])
  Total Products   = DISTINCTCOUNT(Dim Product[ProductID])
  ```
- **Time-intelligence measures:** Current/Previous Year Sales, Current/Previous Month Profit, TotalYTD, Previous Year(py), TotalMTD(Sales), totalMTD (profit) / previous month (profit) — all built using standard DAX time-intelligence functions (`SAMEPERIODLASTYEAR`, `PREVIOUSMONTH`, `TOTALYTD`, `TOTALMTD`) against `Dim Calender[Date]`.
- See `data-model/conceptual-model.png` and `data-model/physical-data-model.png` for the visual diagrams, and the main `README.md` for the full table.

---

## 5. Analysis Questions → Dashboard Mapping

| # | Question | Visual | Field(s) / Slicer |
|---|---|---|---|
| 1 | Which region should be prioritized for investment? | Column chart + pie chart | Region (Total Sales by Region, Total Profit by Region) |
| 2 | Can online growth improve profitability? | Donut chart | Channel (Sum of Sales by Channel: Online vs. Store) |
| 3 | Is sales concentrated in specific regions/categories (risk)? | Category table, Brand/City bar chart | Category, Brand, City, Region |
| 4 | How do seasonal/month trends affect interpretation? | Month & Year range slicers, MoM/YoY charts | Month, Year, Current vs Previous period measures |
| 5 | What additional metrics would help? | — (open-ended, addressed in Insights) | — |
| Growth trend | Company growth over time | Line/column charts | Year, Month, Quarter (Current vs Previous Year Sales, YTD, MTD) |
| Top products | Best-selling products | Bar chart | ProductID (Total Sales by ProductID) |

---

## 6. Final Result

A 3-page Power BI dashboard — **Overview**, **Time Series Analysis**, and **YTD / MTD Trends** — each carrying:
- The same 5 KPI cards (Total Sales, Total Profit, Total Orders, Total Customers, Total Products)
- The same 5 synced slicers (Region, City, Category, Month, Year)

...with page-specific visuals covering regional/category/channel breakdowns (Page 1), year-over-year and month-over-month comparisons plus quarterly online-sales trend (Page 2), and year-to-date/month-to-date running totals (Page 3).

**Headline numbers observed:**
- Total Sales: 49.64M
- Total Profit: 11.23M
- Total Orders: 5K
- Total Customers: 500
- Total Products: 200
- Online vs. Store split: 49.4% vs. 50.6% of sales — near parity
- South region: ~42% of total profit — the clear regional leader

---

## 7. Outcomes

- A single `.pbix` file now replaces manual Excel pivoting for regional, category, and channel sales analysis.
- Regional managers can self-serve a South-vs-other-regions comparison instead of requesting a custom report each time.
- Standalone `Dim Calender` table with proper time-intelligence measures makes year-over-year and month-over-month comparisons reliable and reusable across all three pages.
- Synced slicers mean a manager can set Region = South, Category = Electronics once and see that context reflected consistently on every page.

---

## 8. Insights

- **Region:** South outperforms East, North, and West on both sales volume and profit share, making it the top candidate for continued investment (see README § Insights & Outcomes for the full reasoning).
- **Channel:** Online and Store sales are almost perfectly split, so there is real headroom to grow the (typically lower-overhead) online channel without cannibalizing the store business.
- **Category:** Clothing, Electronics, and Home are closely matched top performers; Beauty is the clear laggard — a candidate for a targeted promotional push or portfolio review.
- **Time trend:** The Current-vs-Previous-Year and Current-vs-Previous-Month measures let the business separate genuine growth from normal seasonal swings, which is important given the Month/Year range slicers allow arbitrary period selection.

---

## 9. Limitations / Next Steps

- The raw export (`zepto_sales_raw.xlsx`) captures granular order-line detail (delivery slot, delivery status, payment method) that isn't yet surfaced in the dashboard — a future iteration could add delivery-performance or payment-method visuals.
- No profit-margin % measure exists yet (Profit ÷ Sales) — adding one would let the team compare efficiency, not just absolute profit, across regions and categories.
- No customer-repeat-rate or cohort measure exists — useful for understanding whether the 500 customers are mostly one-time or repeat buyers.
- Page 3's internal name ("Duplicate of Time Series Analysis") should be renamed to something clearer (e.g. "YTD / MTD Trends") directly in Power BI Desktop before publishing.
