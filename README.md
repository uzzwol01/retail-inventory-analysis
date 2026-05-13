# 🍕 Pizza Sales Data Analysis

An end-to-end business intelligence project analyzing **$817,860 in annual pizza sales** across 21,350 orders — built with SQL Server and Power BI. This project covers the full data analytics pipeline: raw CSV ingestion, relational database setup, SQL query development, data cleaning, and interactive dashboard design.

---

## 📊 Project Overview

| Detail | Value |
|---|---|
| **Dataset** | Pizza restaurant sales — full year 2015 |
| **Total Revenue** | $817,860 |
| **Total Orders** | 21,350 unique orders |
| **Total Pizzas Sold** | 49,574 |
| **Average Order Value** | $38.31 |
| **Average Pizzas Per Order** | 2.32 |
| **SQL Queries Written** | 11 (Sections A–L) |
| **Dashboard Pages** | 2 interactive Power BI pages |
| **Tools Used** | SQL Server Management Studio, Power BI Desktop |

---

## 🏗️ Pipeline Architecture

```
Raw CSV File
     ↓
SQL Server Database (pizza_DB)
     ↓
11 SQL Queries (KPIs, trends, category/size breakdown, best/worst sellers)
     ↓
Power BI — Power Query (data cleaning)
     ↓
2-Page Interactive Dashboard
```

This is the same pipeline used in production business intelligence environments — not a simplified tutorial version.

---

## 🗄️ Database & SQL Work

### Data Type Bug — Caught Before It Corrupted Everything

When SQL Server imported the CSV using the Import Flat File wizard, it auto-detected `order_id` as **tinyint** — a data type that can only store values from 0 to 255. Our dataset had **21,350 unique orders**.

If left uncaught, every KPI built on order count would have been silently wrong. The fix: immediately verified all column types in Table Design after import, changed `order_id` from `tinyint` to `int` (supports up to 2.1 billion values), and confirmed 21,350 rows loaded correctly before writing a single query.

**Lesson: Never trust auto-detected data types on import. Always verify.**

### SQL Query Structure (Sections A–L)

| Section | Topic | Key Function Used |
|---|---|---|
| A | Core KPIs (Revenue, AOV, Total Orders, Qty, Avg) | `SUM`, `COUNT DISTINCT`, `CAST` |
| B | Orders by Day of Week | `DATENAME(DW, order_date)` |
| C | Orders by Month | `DATENAME(MONTH, order_date)` |
| D | Revenue % by Category | Subquery inside `SELECT` |
| E | Revenue % by Size | Subquery inside `SELECT` |
| F | Quantity by Category | `SUM(quantity)` + `GROUP BY` |
| G–H | Top 5 Pizzas by Revenue, Quantity, Orders | `ORDER BY ... DESC`, `TOP 5` |
| I–L | Bottom 5 Pizzas by Revenue, Quantity, Orders | `ORDER BY ... ASC`, `TOP 5` |

### Key SQL Engineering Decisions

**COUNT vs COUNT DISTINCT:** One order generates multiple rows (one per pizza). Using plain `COUNT(order_id)` counted rows — not orders. Switched to `COUNT(DISTINCT order_id)` throughout. The difference in output was significant.

**CAST for decimal division:** SQL Server performs integer division by default. `2 / 3` returns `0`, not `0.67`. Wrapped both values in `CAST(... AS DECIMAL(10,2))` before dividing to force decimal output. Without this, Average Pizzas Per Order would have been wrong.

**Percentage subquery:** Revenue percentage by category required a nested `SELECT SUM(total_price)` inside the main `SELECT` clause to get the grand total as the denominator. Took several attempts to get the clause placement right — understanding SQL execution order was critical.

---

## 📈 Key Findings

### Revenue & Volume
- **$817,860** total annual revenue from **21,350 orders**
- **$38.31** average order value
- **49,574** total pizzas sold

### Peak Periods
- **Busiest day:** Friday — **3,538 orders** (35% more than slowest day)
- **Slowest day:** Sunday — 2,624 orders
- **Peak month:** July — 1,935 orders (summer dining effect)
- **Slowest month:** October — 1,646 orders

### Category Breakdown
| Category | Revenue Share |
|---|---|
| Classic | 26.91% |
| Supreme | 25.46% |
| Chicken | 23.96% |
| Veggie | 23.68% |

All four categories within 3 percentage points — a remarkably balanced menu.

### Size Breakdown — The Most Important Finding
| Size | Revenue Share |
|---|---|
| **Large** | **45.89%** |
| Medium | 30.49% |
| Small | 21.77% |
| XL / XXL | 1.85% |

**Large pizzas generate nearly half of all revenue.** A simple upsell prompt ("Upgrade to Large for $2 more") would have significant revenue impact.

### Best Performers
| Metric | #1 Pizza | Value |
|---|---|---|
| Revenue | Thai Chicken Pizza | $43,434 |
| Quantity | Classic Deluxe Pizza | 2,453 sold |
| Orders | Classic Deluxe Pizza | 2,329 orders |

Note: Thai Chicken leads in revenue but ranks lower in quantity — it earns more per pizza. Classic Deluxe is the most reliable volume performer.

### Worst Performer — Unanimous
The **Brie Carre Pizza** ranked last on all three metrics:
- Last in revenue: $11,588 (27% of what Thai Chicken earns)
- Last in quantity: 490 pizzas sold
- Last in orders: 480 orders

Three different metrics. Three different analytical approaches. Same answer every time — across a full year of 21,000+ orders. The data makes the menu decision straightforward.

---

## 📊 Dashboard — 2 Interactive Pages

### Page 1: Home Dashboard
- **5 KPI cards** at the top (Revenue, AOV, Total Pizzas, Total Orders, Avg Pizzas/Order)
- **Bar chart** — Orders by day of week (Sunday → Saturday calendar order)
- **Line chart** — Monthly trend across all 12 months
- **Doughnut charts** — Revenue % by category and size
- **Horizontal bar chart** — Quantity by category
- **Text summary panel** — Written insights in plain English so the dashboard explains itself without a presenter
- **Category slicer + date range slider** — cross-page filters that update every visual simultaneously

### Page 2: Best & Worst Sellers
- **6 horizontal bar charts** in a 2-row × 3-column layout
- Top 5 and Bottom 5 across: Revenue, Quantity, Total Orders
- Written summary panel with plain-English product recommendations

---

## 🔧 Data Cleaning (Power Query)

**Challenge — pizza_size abbreviations:** The raw data stored sizes as `S`, `M`, `L`, `XL`, `XXL`. These would render as single letters on dashboard charts — unreadable to any stakeholder.

Used Replace Values in Power Query to expand all labels: S → Small, M → Medium, L → Large, XL → Extra Large, XXL → XX-Large.

**Gotcha — case sensitivity:** Power Query's Replace Values is case-sensitive. `XL` and `xl` are different strings. Initial replacement only caught the uppercase version. Required a second separate replacement pass for the lowercase variant. After catching all variants, every size label in every chart reads as a full word.

**Other cleaning steps:**
- Verified `order_date` was recognized as Date type (not text — time-based charts break on text dates)
- Checked for null values in `total_price`
- Verified for duplicate order rows that could cause SUM queries to overcount

---

## 💼 Business Recommendations

| Finding | Recommendation |
|---|---|
| Friday has 35% more orders than Sunday | Schedule additional staff every Friday and Saturday |
| October is the slowest month | Run promotional campaigns in September and October |
| Large = 45.89% of revenue | Add upsell prompt at point of sale ("Upgrade to Large?") |
| Brie Carre is last on all 3 metrics | Evaluate removal from menu — data case is clear |
| Thai Chicken leads revenue | Feature prominently on menu and in marketing |
| Classic Deluxe leads volume | Include in combo deals to maximize order attachment |

---

## 🛠️ Tech Stack

- **Database:** Microsoft SQL Server (SQL Server Management Studio)
- **Queries:** T-SQL — 11 queries across KPIs, trends, category, size, and product performance
- **BI Tool:** Power BI Desktop (Import mode, Power Query Editor)
- **Data Source:** pizza_sales.csv — 48,620 rows, 8 columns

---

## 👤 Author

**Ujjwal Dhakal** — Responsible for:
- Power BI connection to SQL Server database
- Data cleaning in Power Query (size labels, null checks, date types)
- Best & Worst Sellers dashboard page (6 interactive charts)
- Interactive slicer and date filter implementation
- Cross-page filter synchronization

Built in collaboration with a partner who handled database setup, SQL query architecture, and the Home dashboard page.

- LinkedIn: [linkedin.com/in/ujjwal-dhakal-67bb763a4](https://www.linkedin.com/in/ujjwal-dhakal-67bb763a4)
- Email: dhakalujjwal16@gmail.com
