# E-commerce Marketing & Sales — Power BI Dashboard Report



---

## Project Overview

A 5-page interactive Power BI report built on an Indian e-commerce company's marketing and sales data. The report provides end-to-end visibility across sales performance, digital marketing effectiveness, customer behaviour, inventory health, and product-level analysis — covering 30,000+ orders, 42,000+ order items, and 17,000+ customers across 2022–2025.

---

## Dashboard Pages

### Page 1 — Sales Overview

**KPIs:** Total Revenue (₹18M) · Total Orders (8,155) · AOV (₹2.21K) · Return Rate (0.35) · Repeat Rate (0.46)

**Visuals:**
- Revenue & order volume over time (dual-axis line chart with drill-down Year → Month)
- Revenue by channel source (horizontal bar — Meta dominates)
- Revenue by category (donut chart — all 6 categories at equal 16.67%)
- Revenue by city and tier (bubble map — Tier 1 vs Tier 2 cities)
- Orders by delivery status (bar — Delivered / Returned / RTO)

**Filters:** Year · Month · Channel source · Delivery status

---

### Page 2 — Marketing & Ads

**KPIs:** Sum of Purchases (6,654) · Blended ROAS (11.13) · Total Ad Spend (₹7M) · Avg CAC (₹1.12K) · Avg CTR (0.01)

**Visuals:**
- Ad spend vs ROAS over time (combo chart — columns for spend, line for ROAS)
- Campaign funnel (Reach → Link Clicks → Add to Cart → Checkout → Purchases)
- Device sessions breakdown (mobile vs desktop vs tablet)
- Campaign performance matrix (drill-down campaign → adset, conditional formatting on ROAS and CAC)
- CAC vs ROAS by adset (scatter — bubble size = spend)
- Average ROAS by creative ad type (Studio-Reel performs best)

**Filters:** Time period · Campaign name · Creative type · Adset name

---

### Page 3 — Customer Analytics

**KPIs:** Total Customers (17K) · Repeat Customers (7,748) · Repeat Rate (0.46) · Avg LTV (₹4.51K) · Avg Total Orders (1.78) · Avg Days to 2nd Purchase (410)

**Visuals:**
- Top 10 loyal customers by Loyalty Score (horizontal bar — custom DAX composite score)
- Revenue by city & repurchasal status (bubble map)
- AOV table by payment mode (matrix — UPI, Card, COD, NetBanking comparison)
- Time to 2nd purchase distribution (horizontal bar using Days_Bucket calculated column)
- New vs repeat revenue over time (area chart by month)
- Orders by acquisition channel (column chart — Meta leads)
- Orders by payment mode (donut — UPI 37.69%, Card 31.9%, COD 18.24%)

**Filters:** Time period · City tier · Acquisition channel · Repurchased status

---

### Page 4 — Inventory & Supply

**KPIs:** Gross Margin % (51%) · Dead Stock Count (68) · Count of SKU (55) · Low Stock Count (7,170) · Delivery Delay (1.87 days)

**Visuals:**
- Average inventory left by SKU (horizontal bar — sorted by days remaining)
- Lead time and delivery delay by vendor (combo chart — all 5 vendors)
- Stock units by category and size (matrix heatmap — color scale showing over/understocked combinations)
- Most common product return reasons (bar — Size issue is #1 reason)
- Gross margin % by category (bar — Shorts highest, Pants lowest)

**Filters:** Category · Vendor · Dead stock flag · Size

---

### Page 5 — Product & Returns

**KPIs:** Avg Discount % (26.15%) · Items Returned (15K) · Items Sold (42K) · Item Return Rate (0.35) · Discount Impact (₹21M)

**Visuals:**
- Top products by revenue (horizontal bar — Top 15 SKUs, color-coded by category)
- Return rate by category (treemap — Shorts and Jackets have highest return rates)
- Most colors sold by category (stacked horizontal bar — Olive, White, Beige top performers)
- Discount % vs revenue per SKU (scatter — discount does not reliably drive revenue)
- Most ordered colors by category and size (matrix heatmap)

**Filters:** Category · Size · Color · Returned status

---

## Data Model

Star schema with 9 tables:

```
fact_orders ──────────── dim_customers
     │
     ├── fact_order_items ── dim_sku ── dim_inventory
     │                                      │
     │                               fact_procurement
     │
     └── dim_date

fact_meta_ads   (standalone — no order-level join key)
fact_sessions   (standalone — pre-aggregated traffic data)
fact_session_detail
```

### Datasets

| Table | Description | Key columns |
|-------|-------------|-------------|
| `fact_orders` | Order-level transactions | Order ID, Customer ID, Order date, Revenue, Payment mode, Channel |
| `fact_order_items` | Item-level data per order | Order ID, SKU ID, Category, Size, Color, MRP, Selling price, Returned |
| `fact_meta_ads` | Meta advertising campaign data | Campaign, Spend, ROAS, CAC, Purchases, Hook Rate, CTR |
| `fact_sessions` | Daily web traffic aggregated | Date, Traffic source, Device, Sessions, Conversion rate |
| `fact_session_detail` | Individual session records | Session ID, Order ID, Customer ID, Revenue |
| `dim_customers` | Customer master data | Customer ID, Name, City, Tier, Acquisition channel, LTV, Repurchased |
| `dim_inventory` | Stock levels per SKU | SKU, Category, Size, Units in stock, Days of inventory left, Dead stock flag |
| `dim_sku` | Product master | SKU, Category, Vendor, MRP, Cost per unit |
| `fact_procurement` | Purchase orders from vendors | SKU, Vendor, Order qty, Cost, Lead time, Expected/Actual delivery |

---

## DAX Measures & Calculated Columns

All DAX queries are documented separately in [`DAX_Queries.md`](DAX_Queries.md).

---

## Key Insights

| Area | Insight |
|------|---------|
| Sales | Meta is the dominant revenue channel — far ahead of Google and Influencer |
| Sales | Strong October 2023 revenue spike (~₹2.9M) aligns with India's festive season |
| Marketing | Blended ROAS of 11.13x exceeds the D2C apparel benchmark of 3–4x |
| Marketing | Biggest funnel drop-off: Link Clicks (317K) → Add to Cart (18K) — 94% drop |
| Marketing | Studio-Reel creatives outperform all other ad formats in ROAS |
| Customers | 46% repeat rate is strong; average time to 2nd purchase is 410 days (too long) |
| Customers | LTV:CAC ratio is ~4:1 (₹4,510 LTV vs ₹1,120 CAC) — healthy |
| Inventory | Size issue is the #1 return reason — a sizing guide could reduce 15K returns |
| Inventory | 68 dead stock SKUs require markdown or bundling to recover capital |
| Products | SKU-SHI-0001 alone generates ₹19.6M — 2x the next SKU (concentration risk) |
| Products | Higher discounts do not reliably produce higher per-SKU revenue |

---

## Tools & Technologies

- **Power BI Desktop** — report design, DAX, data modelling
- **Power Query (M)** — data cleaning and transformation
- **DAX** — measures, calculated columns, KPI logic
- **Star Schema** — fact-dimension data model design

---

## Repo Structure

```
/
├── README.md                        ← this file
├── DAX_Queries.md                   ← all DAX measures and calculated columns
├── PowerBI
    ├── ecommerce_sales_and_marketing.pbix         ← Power BI report file
├── data/
│   ├── customers.csv
│   ├── orders.csv
│   ├── order_line_items.csv
│   ├── meta_ads_campaigns.csv
│   ├── website_sessions.csv
│   ├── website_daily.csv
│   ├── inventory_snapshots.csv
│   ├── sku_catalog.csv
│   └── purchase_orders.csv
└── screenshots/
    ├── 1 - Sales & Overview.png
    ├── 2 - Marketing & Ads.png
    ├── 3 - Customer Analytics.png
    ├── 4 - Inventory & Supply.png
    └── 5 - Product & Returns.png
```
