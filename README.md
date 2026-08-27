# 🛒 RETAILVISION — Global E-Commerce Analytics Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Measures-blue?style=flat)
![Power Query](https://img.shields.io/badge/Power%20Query-M-green?style=flat)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat)

> A four-page interactive Power BI report that turns raw global e-commerce data into decisions — where revenue comes from, which products actually earn margin, who the valuable customers are, and where fulfilment slows down.

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Business Questions Answered](#-business-questions-answered)
- [Dashboard Pages](#-dashboard-pages)
- [Data Model](#-data-model)
- [Key DAX Measures](#-key-dax-measures)
- [Interactivity](#-interactivity)
- [Design System](#-design-system)
- [Getting Started](#-getting-started)
- [Repository Structure](#-repository-structure)
- [Design Decisions & Known Trade-offs](#-design-decisions--known-trade-offs)
- [Roadmap](#-roadmap)
- [License](#-license)

---

## 🔍 Overview

**RETAILVISION** consolidates sales, product, customer, geographic, store, and delivery data into a single analytical report. Instead of scattered exports and one-off pivot tables, decision-makers get one place to move from *"how did we do?"* to *"why, and where do we act?"*

The report is built around a **decision funnel**:

| Layer | Page | Question it answers |
|---|---|---|
| **What happened** | Executive Overview | Are we growing? Where is the money coming from? |
| **What sells** | Sales & Product Analytics | Which products, brands, and categories earn revenue vs. margin? |
| **Who buys** | Customer & Geography | Which segments and countries are worth the most? |
| **How we deliver** | Operations & Delivery | Which stores perform, and where is fulfilment slow? |

---

## ❓ Business Questions Answered

- Is revenue growing month-over-month and year-over-year, and is profit keeping pace?
- Which categories carry high revenue but **thin margins** (volume traps)?
- Which 10 products drive units, and are they the same 10 that drive profit?
- Which brands are worth expanding, and which are dead weight?
- What is the average revenue per customer by segment, and which segment compounds?
- Which countries over-index on orders but under-index on revenue?
- Which stores are profitable relative to their size?
- Where does average delivery time exceed acceptable thresholds?

---

## 📑 Dashboard Pages

### 1. Executive Overview
High-level health of the business in a single screen.

**KPI cards:** Total Revenue · Total Profit · Total Orders · Total Customers · Average Order Value

**Visuals**
- Monthly Revenue & Profit trend (combo)
- Yearly Revenue & Profit
- Orders & Units by Category
- Profit Margin ranking by Category
- Revenue by Country (map / bar)
- Revenue by Category + Revenue Share (donut)
- Revenue vs Profit by Category (scatter)

**Slicers:** Date Range · Category

---

### 2. Sales & Product Analytics
Product, brand, and category performance — revenue *and* margin side by side.

**Visuals**
- Top 10 Products by Units Sold
- Top 10 Products by Revenue
- Top 10 Products by Profit
- Monthly Revenue & Profit
- Yearly Revenue & Profit trend
- Category Revenue & Profit
- Category Revenue vs Profit
- Brand Revenue & Profit performance
- Brand Profit Margin

**Slicer:** Category

---

### 3. Customer & Geography Analytics
Segment value and geographic concentration.

**Visuals**
- Revenue by Customer Segment
- Units Purchased by Customer Segment
- Customer count by Segment
- Average Revenue per Customer by Segment
- Customer Value by Segment
- Customer Orders vs Revenue (scatter)
- Revenue / Units / Orders by Country

**Slicers:** Customer Segment · Country

---

### 4. Operations & Delivery
Store economics and fulfilment reliability.

**Visuals**
- Average Delivery Time by Country
- Store Size vs Revenue (scatter)
- Store Revenue Performance
- Store Profit Performance
- Store Revenue vs Units Sold
- Top 10 Stores by Profit Margin
- Revenue by Store Country
- Store count by Country

**Slicers:** Country · Store Country

---

## 🗂️ Data Model

The report consumes **nine pre-aggregated analytical tables**, each shaped for a specific analytical grain.

| Table | Grain | Key fields |
|---|---|---|
| `executive_kpis` | Whole business (1 row) | total_revenue, total_profit, total_orders, total_customers, avg_order_value |
| `monthly_sales` | Year–Month | date, year, month, revenue, profit, orders, units_sold |
| `product_performance` | Product | product_id, product_name, brand, category, revenue, profit, units_sold |
| `brand_performance` | Brand | brand, revenue, profit, units_sold, profit_margin |
| `category_performance` | Category | category, revenue, profit, orders, units_sold, profit_margin |
| `customer_segments` | Segment | segment, customers, revenue, orders, units_purchased, avg_revenue_per_customer |
| `country_performance` | Country | country, revenue, profit, orders, units_purchased, customers |
| `store_performance` | Store | store_id, store_name, store_country, store_size, revenue, profit, units_sold |
| `delivery_performance` | Country | country, avg_delivery_days, orders_delivered, on_time_rate |

> 📌 **Grain matters.** Each table answers questions at its own level. Mixing grains without care is the fastest way to double-count revenue, so aggregations are kept deliberately separate rather than force-joined.

---

## 🧮 Key DAX Measures

<details>
<summary><b>Core KPIs</b></summary>

```dax
Total Revenue =
SUM ( monthly_sales[revenue] )

Total Profit =
SUM ( monthly_sales[profit] )

Total Orders =
SUM ( monthly_sales[orders] )

Total Units Sold =
SUM ( monthly_sales[units_sold] )

Total Customers =
SUM ( customer_segments[customers] )

Average Order Value =
DIVIDE ( [Total Revenue], [Total Orders], 0 )

Profit Margin % =
DIVIDE ( [Total Profit], [Total Revenue], 0 )
```
</details>

<details>
<summary><b>Time intelligence</b></summary>

```dax
Revenue YTD =
TOTALYTD ( [Total Revenue], 'monthly_sales'[date] )

Revenue PY =
CALCULATE ( [Total Revenue], SAMEPERIODLASTYEAR ( 'monthly_sales'[date] ) )

Revenue YoY % =
VAR Current = [Total Revenue]
VAR Prior   = [Revenue PY]
RETURN DIVIDE ( Current - Prior, Prior )

Revenue MoM % =
VAR Current = [Total Revenue]
VAR Prior   =
    CALCULATE ( [Total Revenue], DATEADD ( 'monthly_sales'[date], -1, MONTH ) )
RETURN DIVIDE ( Current - Prior, Prior )

Profit YoY % =
VAR Current = [Total Profit]
VAR Prior   = CALCULATE ( [Total Profit], SAMEPERIODLASTYEAR ( 'monthly_sales'[date] ) )
RETURN DIVIDE ( Current - Prior, Prior )
```
</details>

<details>
<summary><b>Product, brand & category</b></summary>

```dax
Product Revenue =
SUM ( product_performance[revenue] )

Product Profit =
SUM ( product_performance[profit] )

Product Margin % =
DIVIDE ( [Product Profit], [Product Revenue], 0 )

Revenue Rank (Product) =
RANKX (
    ALLSELECTED ( product_performance[product_name] ),
    [Product Revenue],
    ,
    DESC,
    DENSE
)

Top 10 Product Revenue =
CALCULATE ( [Product Revenue], KEEPFILTERS ( [Revenue Rank (Product)] <= 10 ) )

Category Revenue Share % =
DIVIDE (
    SUM ( category_performance[revenue] ),
    CALCULATE ( SUM ( category_performance[revenue] ), ALLSELECTED ( category_performance ) ),
    0
)

Brand Margin % =
DIVIDE ( SUM ( brand_performance[profit] ), SUM ( brand_performance[revenue] ), 0 )
```
</details>

<details>
<summary><b>Customer & geography</b></summary>

```dax
Segment Revenue =
SUM ( customer_segments[revenue] )

Avg Revenue per Customer =
DIVIDE ( [Segment Revenue], SUM ( customer_segments[customers] ), 0 )

Avg Orders per Customer =
DIVIDE ( SUM ( customer_segments[orders] ), SUM ( customer_segments[customers] ), 0 )

Country Revenue =
SUM ( country_performance[revenue] )

Country Revenue Share % =
DIVIDE (
    [Country Revenue],
    CALCULATE ( [Country Revenue], ALLSELECTED ( country_performance ) ),
    0
)
```
</details>

<details>
<summary><b>Store & delivery</b></summary>

```dax
Store Revenue =
SUM ( store_performance[revenue] )

Store Profit =
SUM ( store_performance[profit] )

Store Margin % =
DIVIDE ( [Store Profit], [Store Revenue], 0 )

Revenue per Sq Ft =
DIVIDE ( [Store Revenue], SUM ( store_performance[store_size] ), 0 )

Avg Delivery Days =
AVERAGE ( delivery_performance[avg_delivery_days] )

On-Time Delivery % =
AVERAGE ( delivery_performance[on_time_rate] )

Delivery Status =
SWITCH (
    TRUE (),
    [Avg Delivery Days] <= 3, "Fast",
    [Avg Delivery Days] <= 6, "Acceptable",
    "Delayed"
)
```
</details>

<details>
<summary><b>Dynamic titles & no-data handling</b></summary>

```dax
Selected Category Title =
VAR Sel = SELECTEDVALUE ( category_performance[category], "All Categories" )
RETURN "Revenue & Profit — " & Sel

No Data Message =
IF ( ISBLANK ( [Total Revenue] ), "No data for the current filter selection", "" )
```
</details>

---

## 🎛️ Interactivity

| Feature | Implementation |
|---|---|
| Page navigation | Built-in **Page Navigator** on every page |
| Filtering | Category, Date Range, Customer Segment, Country, Store Country slicers |
| Reset | **Reset Filters** button per page, driven by bookmarks |
| Cross-filtering | Enabled between visuals sharing a grain |
| Tooltips | Native hover tooltips with formatted values |

---

## 🎨 Design System

Consistency was treated as a feature, not decoration:

- **Hierarchy** — KPI row on top, trends in the middle, breakdowns below
- **Placement** — slicers in the same position on every page, so muscle memory works
- **Titles** — every visual states the metric *and* the dimension
- **Sorting** — descending by the metric being ranked; no arbitrary alphabetical ordering
- **Labels** — data labels only where they add precision, not on every bar
- **Colour** — one accent for revenue, one for profit, used identically across all four pages
- **Whitespace** — aligned grid, consistent gutters, no crowded corners

---

## 🚀 Getting Started

### Prerequisites
- Power BI Desktop (latest version recommended)
- Windows 10/11

### Steps

1. Clone the repository
```bash
   git clone https://github.com/<your-username>/RETAILVISION.git
   cd RETAILVISION
```
2. Open `RETAILVISION_Final.pbix` in Power BI Desktop.
3. If prompted, update the data source path under **Transform data → Data source settings**.
4. Click **Refresh**.
5. Start on **Executive Overview**, then use the Page Navigator to explore.
6. Use slicers to drill into a category, segment, country, or period — **Reset Filters** returns the page to its default state.

---

## 📁 Repository Structure

```
RETAILVISION/
│
├── README.md
├── RETAILVISION_Final.pbix
│
├── dax/
│   └── measures.dax                  # All DAX measures documented above
│
├── screenshots/
│   ├── executive-overview.png
│   ├── sales-product-analytics.png
│   ├── customer-geography-analytics.png
│   └── operations-delivery.png
│
└── data/
    ├── dataset-description.md        # Data dictionary & source notes
    └── sample/                       # Small, non-sensitive sample extracts only
```

> ⚠️ Do not commit confidential, proprietary, or restricted datasets to a public repository. Ship a sample or a schema description instead.

---

## 🧠 Design Decisions & Known Trade-offs

Being explicit about these is part of the analysis, not an apology for it:

1. **Pre-aggregated tables over a single fact table.** Each analytical table is loaded at its own grain. This keeps visuals fast and the DAX readable.
2. **The Date Range slicer applies to `monthly_sales`.** Time-based visuals respond to it; grain-specific aggregates (brand, store, segment) do not, because forcing artificial relationships between differently-grained tables would produce silently wrong totals. Filtering correctness was prioritised over apparent interactivity.
3. **Revenue and profit are always shown together.** A revenue-only ranking hides volume traps, so every ranking visual carries a margin counterpart.
4. **No cosmetic relationships.** Relationships exist only where they are analytically valid.

---

## 🔮 Roadmap

- [ ] Drill-through pages for product- and store-level investigation
- [ ] Custom tooltip pages with mini trend charts
- [ ] A proper `Date` dimension table to extend time filtering across all grains
- [ ] Additional KPIs: repeat-purchase rate, customer lifetime value, return rate
- [ ] Row-level security for country- or store-scoped access
- [ ] Publish to Power BI Service with scheduled refresh
- [ ] Mobile-optimised page layouts

---

## 📌 Project Status

**Completed** — four interactive analytical pages with navigation, slicers, bookmarks, and a consistent design system.

---

## 📄 License

Released under the MIT License. See `LICENSE` for details.

---

<p align="center">Built with Power BI · Power Query · DAX</p>
