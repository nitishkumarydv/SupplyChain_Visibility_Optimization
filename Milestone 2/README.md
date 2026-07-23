# Milestone 2 – Inventory Analytics & Delivery Performance Dashboards

## Project: Supply Chain Visibility & Optimization

This milestone extends the Milestone 1 Power BI data model with two new
analytical dashboards: **Inventory Analytics** and **Delivery Performance**.

---

## 1. Inventory Turnover Calculation Approach

Inventory turnover measures how efficiently stock is being sold and replaced
over a period.

- **Inventory Turnover Ratio (Value)** = Total Sales ÷ Average Inventory Value
- **Inventory Turnover Ratio (Units)** = Units Sold ÷ Average Stock Quantity

A higher ratio indicates faster-moving inventory and efficient stock
management, while a lower ratio signals overstocking or weak demand for
specific products/categories.

## 2. Slow-Moving and Fast-Moving Inventory Identification Logic

Products are classified using a **Stock Status** measure based on:
- **Days Since Last Sale** (days elapsed since a product's most recent order)
- **Current Stock Quantity**

Classification logic:

| Condition | Status |
|---|---|
| Stock Qty = 0 | Out of Stock |
| Days Idle > 90 and Stock Qty > 0 | Dead Stock |
| Days Idle between 30–90 | Slow-Moving |
| Otherwise | Active (Fast-Moving) |

This lets the business quickly identify which products need clearance,
promotion, or reordering attention.

## 3. Delivery Performance Analysis Methodology

Delivery performance is evaluated using the `delivery_status` field and
shipping date fields:

- **On-Time Delivery %** = Orders with status "Shipping on time" ÷ Total Orders
- **Late Delivery %** = Orders with status "Late delivery" ÷ Total Orders
- **Advance Shipping %** = Orders with status "Advance shipping" ÷ Total Orders
- **Lead Time Variance (Days)** = Avg Actual Shipping Days − Avg Scheduled Shipping Days
- **Regional Late Rate** = Late Delivery % calculated per order_region
- **Late Delivery % Trend** = Late Delivery % tracked across order_date

These measures were visualized by region, shipping mode, and time to spot
recurring delay patterns, with drill-down enabled at the regional and
product level.

## 4. Key Insights & Business Recommendations

- A significant portion of inventory value is concentrated in a few
  warehouses/categories — capacity planning should prioritize these.
- A large share of stock falls under "Dead Stock" or "Slow-Moving" status,
  suggesting the need for discounting, bundling, or reduced future
  procurement for these SKUs.
- Late delivery rates vary meaningfully by region, indicating logistics or
  carrier performance issues concentrated in specific geographies rather
  than a uniform network-wide problem.
- Lead Time Variance shows scheduled vs. actual shipping gaps, which can be
  used to recalibrate customer-facing delivery estimates.
- Recommend setting automated reorder alerts using the Reorder Flag measure
  to reduce stockout risk for high-turnover products.

---

## Dashboards
- `screenshots/Inventory_Analytics.png`
- `screenshots/Delivery_Performance.png`

## File
- `Milestone2_PowerBI.pbix`
