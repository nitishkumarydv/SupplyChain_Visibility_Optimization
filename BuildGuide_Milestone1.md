# Milestone 1 — Build Guide (Power BI Desktop)

This guide assumes the **DataCo Smart Supply Chain Dataset** (Kaggle, `DataCoSupplyChainDataset.csv`, ~180,519 rows). If your CSV has different column names, just swap them in — the logic stays the same.

Typical raw columns include: `Type, Days for shipping (real), Days for shipment (scheduled), Delivery Status, Late_delivery_risk, Category Id, Category Name, Customer City, Customer Country, Customer Email, Customer Fname, Customer Id, Customer Lname, Customer Segment, Customer State, Customer Street, Customer Zipcode, Department Id, Department Name, Latitude, Longitude, Market, Order City, Order Country, Order Customer Id, order date (DateOrders), Order Id, Order Item Cardprod Id, Order Item Discount, Order Item Discount Rate, Order Item Id, Order Item Product Price, Order Item Profit Ratio, Order Item Quantity, Sales, Order Item Total, Order Profit Per Order, Order Region, Order State, Order Status, Order Zipcode, Product Card Id, Product Category Id, Product Description, Product Image, Product Name, Product Price, Product Status, shipping date (DateOrders), Shipping Mode`.

---

## Step 1 — Import into Power BI

1. Open **Power BI Desktop** → `Get Data` → `Text/CSV` → select `SupplyChain.csv`.
2. Click **Transform Data** (not Load) — this opens Power Query Editor. Always transform before loading.

---

## Step 2 — Data Cleaning & Transformation (Power Query)

Do these in the Power Query Editor UI (each becomes an "Applied Step"). Below is the equivalent M code you can paste into **Advanced Editor** if you want to move faster (adjust column names to match your file exactly).

### 2.1 Remove duplicates
`Home → Remove Rows → Remove Duplicates` (select `Order Item Id` as it's usually the grain-unique key).

### 2.2 Handle missing values
- `Product Description` and `Order Zipcode` are usually mostly blank → either drop the column (if not needed for analysis) or replace nulls with `"Unknown"` / `0`.
- For numeric fields like `Sales`, `Order Item Quantity` → replace nulls with `0` only if it's safe to assume "no transaction"; otherwise remove the row.
- Use `Transform → Replace Values` or `Fill Down` where appropriate.

### 2.3 Correct data types
- `order date (DateOrders)`, `shipping date (DateOrders)` → **Date/Time**
- `Sales`, `Order Item Total`, `Order Item Profit Ratio`, `Order Profit Per Order`, `Order Item Discount`, `Order Item Discount Rate`, `Product Price` → **Decimal Number**
- `Order Item Quantity`, `Late_delivery_risk`, `Days for shipping (real)`, `Days for shipment (scheduled)` → **Whole Number**
- IDs (`Order Id`, `Customer Id`, `Product Card Id`, `Category Id`, `Department Id`) → **Whole Number** (used as keys)
- Everything else text-like → **Text**

### 2.4 Rename columns (meaningful, consistent naming)
| Original | Renamed |
|---|---|
| `order date (DateOrders)` | `OrderDate` |
| `shipping date (DateOrders)` | `ShippingDate` |
| `Order Item Quantity` | `Quantity` |
| `Order Item Total` | `OrderItemTotal` |
| `Order Item Product Price` | `UnitPrice` |
| `Order Item Discount Rate` | `DiscountRate` |
| `Order Item Profit Ratio` | `ProfitRatio` |
| `Late_delivery_risk` | `LateDeliveryRisk` |
| `Order Item Id` | `OrderItemID` |
| `Order Id` | `OrderID` |
| `Customer Id` | `CustomerID` |
| `Product Card Id` | `ProductID` |
| `Category Id` | `CategoryID` |
| `Department Id` | `DepartmentID` |

Use `Transform → Rename Column` or right-click header → Rename.

### 2.5 M code snippet (example — paste in Advanced Editor, adjust as needed)

```m
let
    Source = Csv.Document(File.Contents("SupplyChain.csv"),[Delimiter=",", Columns=53, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    PromotedHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    RemovedDuplicates = Table.Distinct(PromotedHeaders, {"Order Item Id"}),
    ChangedType = Table.TransformColumnTypes(RemovedDuplicates,{
        {"order date (DateOrders)", type date},
        {"shipping date (DateOrders)", type date},
        {"Sales", type number},
        {"Order Item Quantity", Int64.Type},
        {"Order Id", Int64.Type},
        {"Customer Id", Int64.Type}
    }),
    RenamedColumns = Table.RenameColumns(ChangedType,{
        {"order date (DateOrders)", "OrderDate"},
        {"shipping date (DateOrders)", "ShippingDate"},
        {"Order Item Quantity", "Quantity"},
        {"Order Id", "OrderID"},
        {"Customer Id", "CustomerID"}
    }),
    RemovedBlankRows = Table.SelectRows(RenamedColumns, each [OrderID] <> null)
in
    RemovedBlankRows
```

### 2.6 Split into star schema tables (do this in Power Query, one query per table)
Use `Reference` (right-click the cleaned base query → **Reference**) to spin off each dimension without re-importing the CSV, then `Remove Duplicates` and `Choose Columns` on each reference.

---

## Step 3 — Star Schema Design

**Grain of fact table:** one row per Order Item (line item within an order).

### Fact_OrderDetails (Fact Table)
| Column | Type | Notes |
|---|---|---|
| OrderItemID | Whole Number | Primary Key |
| OrderID | Whole Number | FK → Dim_Order |
| CustomerID | Whole Number | FK → Dim_Customer |
| ProductID | Whole Number | FK → Dim_Product |
| DepartmentID | Whole Number | FK → Dim_Department |
| OrderDateKey | Whole Number/Date | FK → Dim_Date |
| ShippingDateKey | Whole Number/Date | FK → Dim_Date |
| Quantity | Whole Number | |
| UnitPrice | Decimal | |
| Sales | Decimal | |
| DiscountRate | Decimal | |
| OrderItemTotal | Decimal | |
| ProfitRatio | Decimal | |
| LateDeliveryRisk | Whole Number (0/1) | |

### Dim_Customer
CustomerID (PK), CustomerFirstName, CustomerLastName, Segment, City, State, Country, Zipcode

### Dim_Product
ProductID (PK), ProductName, CategoryID, CategoryName, ProductPrice, ProductStatus

### Dim_Department
DepartmentID (PK), DepartmentName

### Dim_Order
OrderID (PK), OrderStatus, ShippingMode, DeliveryStatus, Market, OrderRegion, OrderCountry, OrderState, OrderCity

### Dim_Date (build with DAX or Power Query, covering min→max of OrderDate/ShippingDate)
DateKey (PK), Date, Day, Month, MonthName, Quarter, Year, Weekday, WeekdayName, IsWeekend

**Relationships (all 1-to-many, single direction, dimension → fact):**
- Dim_Customer[CustomerID] → Fact_OrderDetails[CustomerID]
- Dim_Product[ProductID] → Fact_OrderDetails[ProductID]
- Dim_Department[DepartmentID] → Fact_OrderDetails[DepartmentID]
- Dim_Order[OrderID] → Fact_OrderDetails[OrderID]
- Dim_Date[DateKey] → Fact_OrderDetails[OrderDateKey] (active)
- Dim_Date[DateKey] → Fact_OrderDetails[ShippingDateKey] (inactive — use `USERELATIONSHIP` in DAX when needed)

Arrange in Power BI's Model View with fact table in the center and dimensions radiating outward — classic star shape (screenshot this for `Data_model.png`).

---

## Step 4 — DAX Measures (create in a dedicated "_Measures" table for cleanliness)

```dax
Total Sales = SUM(Fact_OrderDetails[Sales])

Total Orders = DISTINCTCOUNT(Fact_OrderDetails[OrderID])

Total Quantity Sold = SUM(Fact_OrderDetails[Quantity])

Average Order Value = DIVIDE([Total Sales], [Total Orders])

Total Profit = SUM(Fact_OrderDetails[OrderItemTotal]) - SUM(Fact_OrderDetails[Sales])

Profit Margin % = DIVIDE([Total Profit], [Total Sales])

Late Delivery Count = SUM(Fact_OrderDetails[LateDeliveryRisk])

Late Delivery % =
DIVIDE(
    [Late Delivery Count],
    COUNTROWS(Fact_OrderDetails)
)

Avg Shipping Days =
AVERAGEX(
    Fact_OrderDetails,
    DATEDIFF(RELATED(Dim_Order[OrderDate]), RELATED(Dim_Order[ShippingDate]), DAY)
)

Sales LY =
CALCULATE([Total Sales], SAMEPERIODLASTYEAR(Dim_Date[Date]))

Sales Growth % =
DIVIDE([Total Sales] - [Sales LY], [Sales LY])

Shipping Date Sales =
CALCULATE(
    [Total Sales],
    USERELATIONSHIP(Dim_Date[DateKey], Fact_OrderDetails[ShippingDateKey])
)
```

---

## Step 5 — Screenshot for `Data_model.png`
Go to **Model View** in Power BI Desktop, arrange tables star-style, then use `File → Export → ... ` or just a screenshot tool (Win+Shift+S) and save as `screenshots/Data_model.png`.

---

## Step 6 — GitHub Repo Setup

```bash
mkdir SupplyChain_Visibility_Optimization
cd SupplyChain_Visibility_Optimization
mkdir -p Milestone1/data Milestone1/screenshots
# copy SupplyChain_Milestone1.pbix, data/SupplyChain.csv, screenshots/Data_model.png, README.md into place
git init
git add .
git commit -m "Milestone 1: Data modelling, cleaning, star schema, DAX measures"
git branch -M main
git remote add origin https://github.com/<your-username>/SupplyChain_Visibility_Optimization.git
git push -u origin main
```

Make sure the repo is set to **Public** in GitHub settings before sharing the link.
