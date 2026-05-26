# DAX Queries — E-commerce Power BI Report

> All measures are stored in the `_Measures` table unless noted otherwise.  
> Calculated columns are created directly on their respective tables (noted in each section).

---

## Table of Contents

1. [Core Measures — used across all pages](#1-core-measures--used-across-all-pages)
2. [Page 2 — Marketing & Ads](#2-page-2--marketing--ads)
3. [Page 3 — Customer Analytics](#3-page-3--customer-analytics)
4. [Page 4 — Inventory & Supply](#4-page-4--inventory--supply)
5. [Page 5 — Product & Returns](#5-page-5--product--returns)
6. [Calculated Columns](#6-calculated-columns)

---

## 1. Core Measures — used across all pages

### Total Revenue
Sums the net order value across all orders.
```dax
Total Revenue = SUM(fact_orders[order_value_net])
```

### Total Orders
Counts distinct orders — avoids double counting multi-item orders.
```dax
Total Orders = DISTINCTCOUNT(fact_orders[Order ID])
```

### AOV (Average Order Value)
Divides total revenue by total order count. Uses DIVIDE to avoid division-by-zero errors.
```dax
AOV = DIVIDE([Total Revenue], [Total Orders])
```

### Return Rate
Proportion of order items that were returned.
```dax
Return Rate =
  DIVIDE(
    CALCULATE(
      COUNTROWS(fact_order_items),
      fact_order_items[Returned? (Y/N)] = "Y"
    ),
    COUNTROWS(fact_order_items)
  )
```

### Repeat Rate
Proportion of customers who made more than one purchase.
```dax
Repeat Rate =
  DIVIDE(
    CALCULATE(
      COUNTROWS(dim_customers),
      dim_customers[RePurchased] = "Y"
    ),
    COUNTROWS(dim_customers)
  )
```

---

## 2. Page 2 — Marketing & Ads

### Total Ad Spend
Total amount spent across all Meta ad campaigns.
```dax
Total Ad Spend = SUM(fact_meta_ads[Amount spent (INR)])
```

### Blended ROAS
Return on Ad Spend — total conversion value divided by total spend.
```dax
Blended ROAS =
  DIVIDE(
    SUM(fact_meta_ads[purchase_conversion_value]),
    SUM(fact_meta_ads[Amount spent (INR)])
  )
```

### Avg CAC
Average Customer Acquisition Cost across all campaigns.
```dax
Avg CAC = AVERAGE(fact_meta_ads[cac])
```

### Ad Purchases
Total number of purchases attributed to Meta campaigns.
```dax
Ad Purchases = SUM(fact_meta_ads[purchases])
```

### Avg CTR
Average click-through rate on ad links across all campaigns.
```dax
Avg CTR = AVERAGE(fact_meta_ads[ctr_link])
```

### Avg Hook Rate
Average hook rate — proportion of people who watched past the first few seconds of a video ad.
```dax
Avg Hook Rate = AVERAGE(fact_meta_ads[Hook Rate])
```

---

## 3. Page 3 — Customer Analytics

### Total Customers
Total number of customer records.
```dax
Total Customers = COUNTROWS(dim_customers)
```

### Repeat Customers
Number of customers who made a second purchase.
```dax
Repeat Customers =
  CALCULATE(
    COUNTROWS(dim_customers),
    dim_customers[RePurchased] = "Y"
  )
```

### Avg LTV
Average total revenue generated per customer over their lifetime.
```dax
Avg LTV = AVERAGE(dim_customers[Total revenue])
```

### Avg Orders per Customer
Average number of orders placed per customer.
```dax
Avg Orders = AVERAGE(dim_customers[Total orders])
```

### Avg Days to 2nd Purchase
Average number of days between first and second purchase, across customers who repurchased.
```dax
Avg Days to 2nd = AVERAGE(dim_customers[Time to 2nd purchase])
```

### Loyalty Score (Calculated Column on dim_customers)
Composite score combining order frequency, total spend, and repurchase behaviour.  
Weights: Orders (40%) + Revenue per ₹1000 (40%) + Repurchase bonus (20%).
```dax
Loyalty_Score =
  dim_customers[Total orders] * 0.4
  + (dim_customers[Total revenue] / 1000) * 0.4
  + IF(dim_customers[RePurchased] = "Y", 20, 0) * 0.2
```

---

## 4. Page 4 — Inventory & Supply

### Gross Margin %
Margin after subtracting cost per unit from selling price at the item level.
```dax
Gross Margin % =
  DIVIDE(
    SUMX(
      fact_order_items,
      fact_order_items[Selling price] - RELATED(dim_sku[Cost_per_unit])
    ),
    SUM(fact_order_items[Selling price])
  )
```

### Low Stock Count
Number of SKUs with fewer than 30 days of inventory remaining.
```dax
Low Stock Count =
  CALCULATE(
    COUNTROWS(dim_inventory),
    dim_inventory[Days of inventory left] < 30
  )
```

### Dead Stock Count
Number of SKUs flagged as dead stock.
```dax
Dead Stock Count =
  CALCULATE(
    COUNTROWS(dim_inventory),
    dim_inventory[Dead stock flag] = "Y"
  )
```

### Delivery Delay
Average number of days actual delivery exceeded expected delivery across all procurement orders.
```dax
Delivery Delay =
  AVERAGEX(
    fact_procurement,
    DATEDIFF(
      fact_procurement[Expected delivery],
      fact_procurement[Actual delivery],
      DAY
    )
  )
```

### Stock Status (Calculated Column on dim_inventory)
Classifies each SKU's stock health into three risk levels based on days of inventory remaining.
```dax
Stock Status =
  IF(dim_inventory[Days of inventory left] < 30,  "Critical",
  IF(dim_inventory[Days of inventory left] < 60,  "Low",
  "Healthy"))
```

---

## 5. Page 5 — Product & Returns

### Items Sold
Total count of order item rows (each row = one item sold).
```dax
Items Sold = COUNTROWS(fact_order_items)
```

### Items Returned
Count of items where the returned flag is Y.
```dax
Items Returned =
  CALCULATE(
    COUNTROWS(fact_order_items),
    fact_order_items[Returned? (Y/N)] = "Y"
  )
```

### Item Return Rate
Proportion of items sold that were subsequently returned.
```dax
Item Return Rate = DIVIDE([Items Returned], [Items Sold])
```

### Avg Discount %
Average discount percentage applied across all order items.
```dax
Avg Discount % = AVERAGE(fact_order_items[Discount %])
```

### Revenue per SKU
Sum of selling price per SKU — used in the Top 15 SKUs visual and scatter chart.
```dax
Revenue per SKU = SUMX(fact_order_items, fact_order_items[Selling price])
```

### Discount Impact
Total revenue given away through discounting — difference between MRP and actual selling price.
```dax
Discount Impact =
  SUM(fact_order_items[MRP]) - SUM(fact_order_items[Selling price])
```

---

## 6. Calculated Columns

Calculated columns are added directly to tables (right-click table → New column), not to the _Measures table.

### dim_customers[Days_Bucket]
Buckets the time-to-second-purchase field into readable ranges for histogram visualization.  
Blank values (customers who never repurchased) are explicitly handled as "No 2nd purchase".
```dax
Days_Bucket =
  IF(
    ISBLANK(dim_customers[Time to 2nd purchase]),
    "No 2nd purchase",
    SWITCH(TRUE(),
      dim_customers[Time to 2nd purchase] <= 30,  "1 — 0 to 30 days",
      dim_customers[Time to 2nd purchase] <= 60,  "2 — 31 to 60 days",
      dim_customers[Time to 2nd purchase] <= 90,  "3 — 61 to 90 days",
      dim_customers[Time to 2nd purchase] <= 180, "4 — 91 to 180 days",
      "5 — 180+ days"
    )
  )
```
> **Note:** Number prefixes (1, 2, 3...) force correct sort order in visuals automatically without needing a separate sort column.

### dim_customers[Loyalty_Score]
Composite loyalty score. See [Section 3](#3-page-3--customer-analytics) for full explanation.
```dax
Loyalty_Score =
  dim_customers[Total orders] * 0.4
  + (dim_customers[Total revenue] / 1000) * 0.4
  + IF(dim_customers[RePurchased] = "Y", 20, 0) * 0.2
```

### dim_inventory[Stock_Status]
Three-level stock health classification. See [Section 4](#4-page-4--inventory--supply) for full explanation.
```dax
Stock Status =
  IF(dim_inventory[Days of inventory left] < 30,  "Critical",
  IF(dim_inventory[Days of inventory left] < 60,  "Low",
  "Healthy"))
```

### fact_order_items[Category_SKU]
Pulls the Category field from dim_sku into fact_order_items using the active SKU relationship.  
Used as a Legend field in bar charts to color-code bars by product category.
```dax
Category_SKU = RELATED(dim_sku[Category])
```
> **Prerequisite:** Requires an active relationship between `fact_order_items[SKU ID]` and `dim_sku[SKU]` in the data model.

---

## Notes

- All measures use `DIVIDE()` instead of `/` to gracefully handle division-by-zero scenarios
- `RELATED()` only works when there is an active relationship between the two tables in the model view
- Calculated columns using `SWITCH(TRUE(), ...)` evaluate conditions top-to-bottom and return on the first match — order of conditions matters
- `AVERAGEX` and `SUMX` are iterator functions — they evaluate an expression row-by-row across a table, which is why they are used instead of `AVERAGE` or `SUM` when the calculation requires row-level logic
