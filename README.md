# Supply Chain Visibility & Optimization — Milestone 1: Data Modelling

## Objective
Build a clean, reproducible star-schema data model in Power BI for the DataCo supply chain dataset — covering data import, preprocessing (missing values, duplicates, data types, renaming), star schema design (fact/dimension tables with defined relationships), and a set of reusable DAX measures. This model forms the foundation for downstream inventory, delivery, supplier, and transportation analytics modules.

## Dataset Source
**DataCo Smart Supply Chain Dataset** (Kaggle): `DataCoSupplyChainDataset.csv`
- 180,519 rows × 53 columns
- Covers order-level and order-item-level transactional data: customers, products, categories, departments, shipping, delivery status, sales, discounts, and profit — spanning Jan 2015 – Feb 2018.
- Source: https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis

## Data Cleaning and Transformation Steps
Performed in Power Query (see `PowerQuery_M_Script.txt` for the exact M code used):

1. **Removed unusable/PII columns**: `Product Description` (100% null across all 180,519 rows — dropped entirely), plus `Customer Email`, `Customer Password`, `Customer Fname`, `Customer Lname`, `Customer Street`, `Product Image` (not needed for analytics and reduce PII exposure in a public repo).
2. **Duplicates**: Checked on the natural grain key `Order Item Id` — 0 duplicates found; `Table.Distinct` applied defensively.
3. **Missing values**:
   - `Customer Zipcode` — 3 nulls → replaced with `0`.
   - `Order Zipcode` — 155,679 nulls (~86% of rows, largely international orders without a postal code) → replaced with `0` rather than dropped, to preserve row-level fact data.
4. **Data type corrections**: Dates (`order date (DateOrders)`, `shipping date (DateOrders)`) → Date/Time; monetary/ratio fields (`Sales`, `Order Item Total`, `Benefit per order`, `Order Item Discount Rate`, etc.) → Decimal Number; IDs and counts → Whole Number.
5. **Renamed columns** to consistent PascalCase, meaningful names (e.g. `order date (DateOrders)` → `OrderDate`, `Order Item Quantity` → `Quantity`, `Late_delivery_risk` → `LateDeliveryRisk`). Full mapping in the M script.
6. **Surrogate date keys** (`OrderDateKey`, `ShippingDateKey`, format `yyyyMMdd`) added to the base query to relate transactions to `Dim_Date`.
7. **Star schema split**: The cleaned base query was referenced (not re-imported) to build each fact/dimension table, keeping a single source of truth and avoiding redundant CSV reads.

## Data Model Overview
Star schema, grain = **one row per Order Item** (line item within an order).

**Fact_OrderDetails** (180,519 rows)
OrderItemID (PK), OrderID, CustomerID, ProductID, DepartmentID, OrderDateKey, ShippingDateKey, Quantity, UnitPrice, Sales, OrderItemDiscount, DiscountRate, OrderItemTotal, BenefitPerOrder, OrderProfitPerOrder, ProfitRatio, SalesPerCustomer

**Dimension tables:**
| Table | Rows | Key Columns |
|---|---|---|
| Dim_Customer | 20,652 | CustomerID (PK), CustomerCity, CustomerState, CustomerCountry, CustomerSegment, CustomerZipcode |
| Dim_Product | 118 | ProductID (PK), ProductName, ProductPrice, ProductStatus, CategoryID, CategoryName |
| Dim_Department | 11 | DepartmentID (PK), DepartmentName |
| Dim_Order | 65,752 | OrderID (PK), OrderStatus, ShippingMode, DeliveryStatus, Market, OrderRegion, OrderCountry, OrderState, OrderCity, PaymentType, DaysForShippingReal, DaysForShipmentScheduled, LateDeliveryRisk |
| Dim_Date | 1,133 | DateKey (PK), Date, Day, Month, MonthName, Quarter, Year, Weekday, WeekdayName, IsWeekend |

**Relationships** (all 1-to-many, dimension → fact, single direction except where noted):
- Dim_Customer[CustomerID] → Fact_OrderDetails[CustomerID]
- Dim_Product[ProductID] → Fact_OrderDetails[ProductID]
- Dim_Department[DepartmentID] → Fact_OrderDetails[DepartmentID]
- Dim_Order[OrderID] → Fact_OrderDetails[OrderID]
- Dim_Date[DateKey] → Fact_OrderDetails[OrderDateKey] *(active)*
- Dim_Date[DateKey] → Fact_OrderDetails[ShippingDateKey] *(inactive — activated in DAX via `USERELATIONSHIP` for shipping-date-based measures)*

See `screenshots/Data_model.png` for the Model View layout.

### Reusable DAX Measures (see `DAX_Measures.txt`)
Total Sales, Total Orders, Average Order Value, Profit Margin %, Late Delivery %, Avg Shipping Days (Real/Scheduled), Shipping Delay (Days), Sales Per Customer, Sales Growth % (YoY), Shipping Date Sales (alternate date relationship) — full list and definitions in the file.

## Tools Used
- Power BI Desktop (data model, Power Query, DAX, relationships)
- Power Query (M language) for ETL
- DAX for reusable measures
- Python/pandas — used during preparation to profile the raw dataset (null counts, duplicate checks, cardinality) ahead of building the Power Query steps


## How to Reproduce
1. Open Power BI Desktop → `Get Data` → `Text/CSV` → `data/SupplyChain.csv` → `Transform Data`.
2. In Power Query Advanced Editor, paste `PowerQuery_M_Script.txt` for the base cleaning query, then create the 5 referenced star-schema tables as described in the comments at the end of that file.
3. Add a new Blank Query, paste `PowerQuery_DimDate_Script.txt`, name it `Dim_Date`.
4. Close & Apply. In Model View, build the relationships listed above (mark the shipping-date one as inactive).
5. Add a `_Measures` table and paste in the measures from `DAX_Measures.txt`.
6. Save as `SupplyChain_Milestone1.pbix`, screenshot Model View → save as `screenshots/Data_model.png`.
