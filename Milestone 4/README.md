# Milestone 4 – Warehouse Efficiency Dashboard & Executive Dashboard
**Project:** Supply Chain Visibility & Optimization
**Tool:** Power BI (extends `Milestone3_PowerBI.pbix`)

---

## 1. Overview

This milestone extends the Milestone 3 Power BI model with two new report pages:

1. **Warehouse Efficiency Dashboard** – operational-level view of utilization, throughput, storage efficiency, and productivity across warehouses, regions, and product categories.
2. **Executive Dashboard** – consolidated, drill-through-enabled KPI view for leadership, combining metrics from all prior modules with trend and forecast visuals.

---

## 2. Warehouse Utilization Calculation Methodology

Utilization measures how much of a warehouse's available capacity is actively being used for storage at a given point in time.

```
Warehouse Utilization % =
DIVIDE(
    SUM(Inventory[OccupiedSpaceUnits]),
    SUM(Warehouse[TotalCapacityUnits]),
    0
)
```

- **OccupiedSpaceUnits**: current stored inventory volume/pallets/units per warehouse (snapshot fact table).
- **TotalCapacityUnits**: static warehouse dimension attribute (max storage capacity).
- Utilization is evaluated both as a **point-in-time snapshot** (latest inventory date) and as a **trend over time** using a date-filtered version:

```
Avg Utilization % (Period) =
AVERAGEX(
    VALUES('Date'[Date]),
    [Warehouse Utilization %]
)
```

- Warehouses are bucketed into **Under-utilized (<60%)**, **Optimal (60–85%)**, and **Over-utilized (>85%)** bands using a calculated column/measure, visualized via conditional formatting.

---

## 3. Throughput and Productivity KPI Calculations

**Throughput** – volume of goods moved (received + shipped) per warehouse per time period:

```
Total Throughput (Units) =
CALCULATE(SUM(Orders[UnitsReceived])) + CALCULATE(SUM(Orders[UnitsShipped]))

Throughput per Day =
DIVIDE([Total Throughput (Units)], DISTINCTCOUNT('Date'[Date]), 0)
```

**Productivity** – output relative to labor/resource input:

```
Units Processed per Labor Hour =
DIVIDE(SUM(Orders[UnitsProcessed]), SUM(Labor[HoursWorked]), 0)

Order Fulfillment Rate % =
DIVIDE(
    CALCULATE(COUNTROWS(Orders), Orders[Status] = "Fulfilled"),
    COUNTROWS(Orders),
    0
)

Average Dock-to-Stock Time (Hrs) =
AVERAGEX(Orders, DATEDIFF(Orders[ReceivedDateTime], Orders[PutAwayDateTime], HOUR))
```

**Inventory Storage Efficiency**:

```
Storage Efficiency % =
DIVIDE(SUM(Inventory[UnitsStored]), SUM(Warehouse[StorageSlotsAvailable]), 0)

Inventory Turnover Ratio =
DIVIDE(SUM(Orders[UnitsShipped]), AVERAGE(Inventory[AvgInventoryOnHand]), 0)
```

All KPIs are sliceable by **Warehouse, Region, and Product Category** via the shared dimension model (star schema from Milestone 3), using standard `CALCULATE`/`ALLSELECTED` patterns for cross-filtering consistency.

---

## 4. Executive Dashboard Design Methodology

- **Single-page consolidated view**: top KPI cards (Revenue, Order Fulfillment %, Avg Utilization %, Throughput, On-Time Delivery %, Inventory Turnover) pulled from all prior milestone modules (Sales/Demand, Inventory, Logistics, Warehouse).
- **Design principles applied**:
  - F-pattern layout — KPI cards top row, trend charts middle, detail/drill-through table bottom.
  - Consistent color theme and conditional formatting (red/amber/green) for KPI health.
  - Card visuals use `SELECTEDVALUE` + `SWITCH` measures to dynamically change titles based on slicer context.
- **Interactivity**:
  - Slicers for Region, Warehouse, Product Category, and Date Range synced across all visual pages.
  - **Drill-through** from Executive Dashboard KPI cards/charts → Warehouse Efficiency Dashboard (right-click → "Drill through to Warehouse Detail"), passing filter context (Warehouse/Region) automatically.
  - Bookmarks + buttons for toggling between "Summary" and "Detailed" views.

---

## 5. Forecasting Implementation Approach

- Power BI's built-in **Analytics pane forecasting** (exponential smoothing) applied to the Throughput and Demand trend line charts, with a forecast horizon of 3 months and 95% confidence interval bands.
- For a more controlled forecast, a DAX-based moving average is also provided as an alternative measure:

```
3-Month Moving Avg Throughput =
AVERAGEX(
    DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, MONTH),
    [Total Throughput (Units)]
)
```

- Forecast visuals are placed on the Executive Dashboard trend charts, clearly labeled "Forecast" vs "Actual" using a legend.

---

## 6. Dashboard Optimization Techniques

- Reduced visual count per page and used **aggregated tables** (pre-summarized at Warehouse/Region/Month grain) instead of row-level detail visuals where possible.
- Removed unused columns from the data model; disabled auto date/time hierarchies not in use.
- Used **variables (`VAR`)** in complex DAX measures to avoid repeated context evaluation.
- Applied `ALLSELECTED` instead of `ALL` where cross-filter interactions needed to stay respectful of slicers.
- Set up **incremental refresh** (if using Power BI Service) / limited historical data load in Power Query to reduce model size.
- Disabled visual interactions where not needed (Format → Edit Interactions) to reduce unnecessary recalculation.
- Used **Performance Analyzer** to identify and optimize slow visuals/DAX queries before final export.

---

## 7. Key Insights & Business Recommendations

*(Replace with your actual data-driven findings before submission — sample structure below)*

- **Utilization imbalance**: Warehouses in [Region X] are operating above 90% utilization while [Region Y] sits below 55% — recommend redistributing inventory or reallocating capacity.
- **Throughput bottlenecks**: [Warehouse A] shows the lowest units-processed-per-labor-hour, suggesting a staffing or process review.
- **Storage efficiency**: Product Category [Z] has the lowest inventory turnover, indicating potential overstocking — recommend demand-driven replenishment.
- **Forecast outlook**: Throughput trend indicates a [X%] increase over the next quarter, suggesting proactive labor/capacity planning.
- **Overall recommendation**: Prioritize capacity rebalancing and process standardization at underperforming sites to improve network-wide efficiency.

---


