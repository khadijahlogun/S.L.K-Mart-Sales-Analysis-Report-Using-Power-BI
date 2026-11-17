# S.L.K Mart Sales Analysis Report

Using Power BI, I analysed 2023 retail sales data for S.L.K Mart, a Nigerian multi-category retailer, to uncover insights on product category performance, regional sales volume, product costs and sales by category. I prepared four related tables in Power Query, built a star-schema data model, and designed an interactive dashboard with slicers for order date, manager and state headquarters.

## Table of Contents
- [Dashboard Preview](#dashboard-preview)
- [Introduction](#introduction)
- [Data Overview and Dataset Structure](#data-overview-and-dataset-structure)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Data Model and DAX Measures](#data-model-and-dax-measures)
- [Dashboard Analysis](#dashboard-analysis)
- [Conclusion](#conclusion)
- [Recommendations](#recommendations)
- [Assumptions and Limitations](#assumptions-and-limitations)
- [Tools and Repository Structure](#tools-and-repository-structure)

## Dashboard Preview

![S.L.K Mart Sales Analysis Dashboard](images/SLK%20Power%20BI%20Project.png)
## Introduction

A retailer selling groceries, electronics, fashion and home goods across five regions needs to know which categories move the most stock, which regions sell the most, and where its product spend is concentrated. Without this view, stocking decisions rely on guesswork and regional managers cannot be compared fairly.

This project analyses S.L.K Mart's 2023 sales data using Microsoft Power BI to answer practical questions about sales and cost. The analysis aims to:

- Compare quantity sold across the four product categories.
- Show each category's share of total sales.
- Compare quantity sold across the five regions.
- Identify the products with the highest total cost.
- Report headline totals for orders, cost, sales and quantity sold.
- Let managers filter the report by order date, manager and state headquarters.

## Data Overview and Dataset Structure

The analysis uses four related tables from `SALES_AD.xlsx`.

**sales_data_3 (fact table)**: 1,550 rows covering 1 January to 31 December 2023.

| Column | Description |
|---|---|
| orderdate | Date the order was placed |
| ordernumber | Order identifier (for example SO2023117) |
| productkey | Links to the product_lookup table |
| customerkey | Links to the customer_details table |
| territorykey | Links to the territory_lookup table |
| orderquantity | Number of units ordered |
| Product_Name, Product_Categpry | Product name and category, repeated from the lookup |
| Product_Cost, product_price | Unit cost and unit selling price |
| Region, State_headquaters, Manager | Territory details repeated from the lookup |
| First_Name, Last_Name, Gender | Customer details repeated from the lookup |

**product_lookup (dimension table)**: 32 products with `productkey`, `productname`, `productcategory`, `productcost` and `productprice`. Four categories: Groceries & Essentials, Electronics & Tech, Fashion & Apparel and Home & Living.

**territory_lookup (dimension table)**: 5 territories with `territorykey`, `region`, `state_headquarters` and `manager`.

| Region | State headquarters | Manager |
|---|---|---|
| South West | Lagos | Oluwaseun A. |
| North Central | Abuja | Aminu B. |
| South East | Enugu | Nneka O. |
| North West | Kano | Ibrahim K. |
| South South | Port Harcourt | Tamuno W. |

**customer_details (dimension table)**: 125 customers with `customerkey`, name, `dateofbirth`, `gender`, `maritalstatus` and `occupation`.

This is a practice dataset, not real company data.

## Data Cleaning and Preparation

The raw sales table has several quality issues. All cleaning is done in Power Query so every step is recorded and repeatable and the source file is left untouched.

| Issue found in `sales_data_3` | Treatment |
|---|---|
| 50 fully duplicated rows (same order number and content) | Remove duplicates, reducing 1,550 rows to 1,500 unique orders |
| `orderquantity` stored as text, with some values carrying a leading apostrophe (for example `'12`) | Strip the apostrophe and convert the column to a whole number |
| 30 blank `orderquantity` values and 1 invalid value (`z`) | Exclude from quantity and revenue calculations instead of treating them as zero |
| 1 row where `Product_Name` holds the literal text `productname` | Remove, since it is a stray header value |
| Misspelled column names (`Product_Categpry`, `State_headquaters`) | Rename to `Product_Category` and `State_Headquarters` |
| Unnamed or empty columns, and an unlabelled `Column1` | Remove empty columns and drop `Column1` from the model |

## Data Model and DAX Measures

The model is a star schema. `sales_data_3` is the fact table, linked one-to-many from `product_lookup` (productkey), `customer_details` (customerkey) and `territory_lookup` (territorykey).

Transaction-level measures I recommend for the cleaned fact table:

```DAX
Total Orders = DISTINCTCOUNT(sales_data_3[ordernumber])

Total Quantity Sold = SUM(sales_data_3[orderquantity])

Total Sales = SUMX(sales_data_3, sales_data_3[orderquantity] * sales_data_3[product_price])

Total Cost = SUMX(sales_data_3, sales_data_3[orderquantity] * sales_data_3[Product_Cost])

Profit = [Total Sales] - [Total Cost]

Profit Margin = DIVIDE([Profit], [Total Sales])
```

## Dashboard Analysis

This section explains each part of the dashboard, with the method and the insight it gives.

### 1. Headline KPIs
**Method:** Four cards show Total Order, Total Cost, Total Sales and Total Quantity Sold.

The dashboard reports 125 total orders, 681K total cost, 908K total sales and about 2K total quantity sold. See the validation note under Assumptions and Limitations, because these cards need to be rebuilt on the transaction table before they are used for decisions.

### 2. Total Quantity by Category
**Method:** A horizontal bar chart counts quantity for each product category.

Groceries & Essentials leads with 541, followed by Electronics & Tech (444), Fashion & Apparel (289) and Home & Living (276). Groceries alone accounts for about 35% of the 1,550 units shown, which fits a retailer whose customers buy everyday items most often.

### 3. Total Sales by Category
**Method:** A pie chart shows each category's share of total sales.

Fashion & Apparel holds the largest share at 30.12%, followed by Groceries & Essentials (26.24%), Electronics & Tech (21.87%) and Home & Living (21.77%). Fashion sells fewer units than Groceries and Electronics but earns the biggest sales share, which points to higher-priced items in that category.

### 4. Total Quantity by Region
**Method:** A column chart compares quantity across the five regions.

South South leads with 320, followed by South West (317), North Central (307), North West (305) and South East (301). The gap between first and last is only 19 units, so regional demand is very even.

### 5. Total Cost by Product
**Method:** A column chart ranks the top ten products by total cost.

Bluetooth Speaker (69K) and Rechargeable Fan (68K) carry the highest cost, followed by Smart Watch and Lace Material (59K each), Peak Milk Pack and Casual Polo (55K each), Indomie Noodles Case (52K), Ankara Print Fabric (42K), Bedding Set (39K) and Electric Kettle (35K). Electronics and home appliances dominate the top of the list.

### 6. Slicers
**Method:** Dropdown slicers filter every visual by Order Date, Manager and State Headquarters.

This lets each regional manager view their own territory, and lets the business compare months or states without building separate reports.

## Conclusion

The dashboard gives a clear first view of S.L.K Mart's sales:

- **Category demand:** Groceries & Essentials sells the most units (541), ahead of Electronics & Tech (444).
- **Sales share:** Fashion & Apparel leads sales at 30.12% despite lower unit volume.
- **Regions:** All five regions sell between 301 and 320 units, so no region is a clear laggard.
- **Cost concentration:** Bluetooth Speaker and Rechargeable Fan are the two highest-cost products.
- **Filtering:** Slicers by date, manager and state support territory-level review.

## Recommendations

1. **Rebuild the KPI cards on the transaction table.** Calculate sales and cost from quantity multiplied by unit price and unit cost, so the totals reflect what was actually sold.
2. **Review pricing and stock for Fashion & Apparel.** It earns the largest sales share on fewer units, so protect its stock levels.
3. **Keep Groceries & Essentials well stocked.** It drives the most unit volume.
4. **Watch the high-cost electronics and appliances.** Bluetooth Speaker, Rechargeable Fan and Smart Watch tie up the most cost, so track their margins closely.
5. **Add profit and monthly trend visuals.** A profit margin card and a sales-by-month line chart would show seasonality and profitability.
6. **Improve data capture.** Make quantity a required numeric field and add checks against duplicate order numbers.

## Assumptions and Limitations

- **The headline KPIs do not match the transaction data.** Total Cost (681K) and Total Sales (908K) equal the sum of the unit cost and unit price columns in `product_lookup` (680,700 and 908,027), which is a price list and not a record of sales. Total Order (125) equals the number of rows in `customer_details`. Total quantity (about 2K) matches the 1,550 row count of the raw sales table and not the sum of `orderquantity`.
- **Validation figures from the cleaned transaction table.** After removing duplicates, the stray header row and blank or invalid quantities, 1,470 rows remain. They show 9,577 units sold, about 270.4M in sales (quantity multiplied by unit price), about 203.7M in cost and about 66.8M in profit. Groceries & Essentials then leads sales with a 31.3% share.
- The pie chart percentages do not exactly match either source table, so check the measure behind that visual.
- The quantity-by-category and quantity-by-region charts count rows. In the raw data these counts include the 50 duplicate rows.
- The same product name appears under several product keys with different costs and prices in `product_lookup` (for example Peak Milk Pack has three keys), so product-level results depend on whether you group by name or by key.
- This is a practice dataset covering one year (2023), so findings describe patterns and are not forecasts.

## Tools and Repository Structure

**Tools:** Power BI Desktop, Power Query (M), DAX, Excel.

```
.
├── README.md
├── powerbi/    # .pbix dashboard file (add your file here)
├── data/       # SALES_AD.xlsx raw source tables
└── images/     # dashboard screenshot
```

## Author

**S.L.K Studio** | AI Creator | AI Video & Digital Content Specialist
Instagram: [@s.l.k_studio](https://instagram.com/s.l.k_studio) | YouTube: [slk_studio1](https://youtube.com/@slk_studio1)
