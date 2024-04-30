# 🧬 Business Insights 360 Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Data Wrangling](#phase-1-data-wrangling)
- [Phase 2: ETL with Power Query](#phase-2-etl-with-power-query)
- [Phase 3: Data Modelling & Calculated Columns](#phase-3-data-modelling--calculated-columns)
- [Phase 4: Finance View](#phase-4-finance-view)

---

## Phase 1: Data Wrangling

`Step 1: Loading Data to MySQL Workbench`

1. Download the extract the .sql data files.
2. New Connection → Rename Business Insights 360 → Test the connection
3. Server → Data Import → Import from self-contained file → Select the .sql files from directory → Start Import → Once import complete → Check database in the Navigator Schema

`Step 2: Connecting MySQL Database to Power BI`

- Get Data → MySQL Database → Server: localhost & Database: gdb041 & gdb056 → Load all the tables

---

## Phase 2: ETL with Power Query

`Step 1: Creating custom Date Table`

1. New Source → Blank Query → Custom M Code: = {Number.From(#date(2017,9,1))..Number.From(#date(2022,12,31))}→ Will create a list of date values → Convert to Table → Set Data type as Date → Rename column as Date
2. Add columns → Day, Day Name, Start of Week, Start of Month & Month Name.
3. Add Custom fiscal_year column using M code: Date.Year(Date.AddMonths([month],4))

   NOTE: This basically adds 4 months to the Date Month and then extracts the Year out of the result. Our financial year starts in Sep hence we add 4 months.
4. Filter out the fiscal_year column for 2023 values since we have incomplete data for it.

`Step 2: Creating Last Sales Month Reference Table`

1. Create a reference table from the fact_sales_monthly table. Extract the date column by editing the source step code to: #"gdb041 fact_sales_monthly"[date]
2. Calculate the Last Sales Month table by getting the maximum date value from the column using: = List.Max(#"gdb041 fact_sales_monthly"[date])

`Step 3: Creating Remaining Forecast Reference Table`

1. Create a reference table from the fact_forecast_monthly table. Sort the table ascending by date values.
2. Filter for all the data after the Last Sales Month value using M code: Table.SelectRows(Source, each ([date] > last_sales_month))

`Step 4: Creating a New Table with both Actual & Forecast Data`

1. Append the fact_sales monthly table and remaining_forecast table as a new table fact_actuals&estimates.
2. Rename sold_quantity (from fact_sales_monthly) and forecast_quantity (from remaining_forecast) as qty so the columns get merged into one column in the append operation.

`Step 5: Calculating Net Invoice Sales based on FY varying Gross Price & Pre-invoice Deductions`

Logic: Gross Price - Pre-invoice Deductions = Net Invoice Sales → Net Invoice Sales - Post-invoice Deductions = Net Sales (will be done later)

1. Add a custom calculated column fiscal_year to the fact_actuals&estimates table to calculate the FY based on the date column using M code: Date.Year(Date.AddMonths([date],4))
2. Merge the fact_actuals&estimates table and the gross_price table based on 2 columns: product_code and fiscal_year with a Left Outer Join. Expand the merged columns to show only gross_price without the reference name. Set the data type as Currency. 
3. Add a custom calulated column gross_sales that is the product of gross_price and the qty sold columns. Set the data type as Currency.
4. Now Merge the fact_actuals&estimates table and the pre_invoice_deductions table based on 2 columns: customer_code and fiscal_year with a Left Outer Join. Expand the merged columns to show only pre_invoice_discount_pct without the reference name. Set the data type as Percentage.
5. Add a custom calculated column pre_invoice_discount that is the product of pre_invoice_discount_pct and gross_sales columns. Set the data type as Currency.
6. Add a custom calculated column net_invoice_sales that is the difference between gross_sales and pre_invoice_discount columns. Set the data type as Currency.

---

## Phase 3: Data Modelling & Calculated Columns

`Step 1: Normalizing Data in Tables`

- When the Data Engineer provided us the raw data, it was de-normalized and hence the fact tables contain redundant data like customer name and product name.
- This redundant data increases the data refresh time and hence needs to be handled by Power Query:
   fact_sales_monthly Table: Choose Columns → Uncheck customer_name, channel, platform, market, product, category & division columns.
   fact_forecast_monthly Table: Choose Columns → Uncheck customer_name, channel, platform, market, product, category & division columns.

`Step 2: Creating Table Relationships`

|Primary Key (Dimension table) (1)||Secondary Key (Fact table) (*)|
|-|-|-|
|date (dim_date)|→|date(fact_sales_monthly)|
|date (dim_date)|→|date(fact_forecast_monthly)|
|date (dim_date)|→|date(fact_actuals&estimates)|
|date (dim_date)|→|date(post_invoice_deductions)|
|customer_code(dim_customer)|→|customer_code(fact_sales_monthly)|
|customer_code(dim_customer)|→|customer_code(fact_forecast_monthly)|
|customer_code(dim_customer)|→|customer_code(fact_actuals&estimates)|
|customer_code(dim_customer)|→|customer_code(post_invoice_deductions)|
|product_code(dim_product)|→|product_code(fact_sales_monthly)|
|product_code(dim_product)|→|product_code(fact_forecast_monthly)|
|product_code(dim_product)|→|product_code(fact_actuals&estimates)|
|product_code(dim_product)|→|product_code(post_invoice_deductions)|
|product_code(dim_product)|→|product_code(manufacturing_cost)|
|market(dim_market)|→|market(dim_customer)|
|market(dim_market)|→|market(freight_cost)|

`Step 3: Creating fiscal_year table using DAX`

- We want to create table relationships for fiscal year values, fiscal_year(dim_date) → fiscal_year(freight_cost) however this leads to a many-to-many relationship which is not advised.
- Hence we need a separate fiscal year table that would contain unique values. Data View → New Table → DAX: fiscal_year = ALLNOBLANKROW(dim_date[fiscal_year])
- Table Relationships: fiscal_year(fiscal_year) → fiscal_year(dim_date)     &     fiscal_year(fiscal_year) → fiscal_year(freight_cost)     &     fiscal_year(fiscal_year) → cost_year(manufacturing_cost)

`Step 4: Calculated Columns for post_invoice Calculations`

1. We now need to get the Post invoice deductions data into the fact_actuals&estimates table. 

   For this DAX Calculated Column: post_invoice_deductions_pct = CALCULATE(MAX(post_invoice_deductions[discounts_pct]), RELATEDTABLE(post_invoice_deductions))

   NOTE: This could have also be done simply by merging both tables based on product_code, customer_code and date as merging parameters.
2. To the get the numeric value instead of pct we’ll multiple it with the net_invoice_sales value to get the updated DAX Calculated Column. Set the data type as Currency.

   post_invoice_deductions = var pct = CALCULATE(MAX(post_invoice_deductions[discounts_pct]), RELATEDTABLE(post_invoice_deductions))
   return pct*'fact_actuals&estimates'[net_invoice_sales]
3. We’ll create similar DAX Calculated Column for the other deductions amount. Set the data type as Currency.

   post_invoice_other_deductions = var pct = CALCULATE(MAX(post_invoice_deductions[other_deductions_pct]), RELATEDTABLE(post_invoice_deductions))
   return pct*'fact_actuals&estimates'[net_invoice_sales]
4. Now Net Sales will be all Post Invoice Deductions subtracted from Net Invoice Sales. Set the data type as Currency.

   DAX: net_sales = 'fact_actuals&estimates'[net_invoice_sales]-'fact_actuals&estimates'[post_invoice_deductions]-'fact_actuals&estimates'[post_invoice_other_deductions]

`Step 5: Calculated Columns for COGS & Gross Margin Calculations`

1. We’ll calculate the Manufacturing Cost based on Qty of each Product. Set the data type as Currency.

   DAX Calculated Column: manufacturing_cost = var res = CALCULATE(MAX(manufacturing_cost[manufacturing_cost]), RELATEDTABLE(manufacturing_cost))
   return res*'fact_actuals&estimates'[qty]
2. We’ll calculate the Freight Cost on the Net Sales value. Set the data type as Currency.

   DAX Calculated Column: freight_cost = var pct = CALCULATE(MAX(freight_cost[freight_pct]), RELATEDTABLE(freight_cost))
   return pct*'fact_actuals&estimates'[net_sales]
3. We’ll calculate the Other Cost on the Net Sales value. Set the data type as Currency.
   
   DAX Calculated Column: other_cost = var pct = CALCULATE(MAX(freight_cost[other_cost_pct]), RELATEDTABLE(freight_cost))
   return pct*'fact_actuals&estimates'[net_sales]
4. We’ll calculate Cost of Goods Sold (COGS) = Manufacturing Cost + Freight (Transportation) + Other Costs. Set the data type as Currency.

   DAX Calculated Column: cogs = 'fact_actuals&estimates'[manufacturing_cost]+'fact_actuals&estimates'[freight_cost]+'fact_actuals&estimates'[other_cost]
5. Finally Gross Margin is the difference between Net Sales and COGS. Set the data type as Currency.

   DAX Calculated Column: gross_margin = 'fact_actuals&estimates'[net_sales]-'fact_actuals&estimates'[cogs]
6. Gross Margin % will be calculated over the Net Sales value. Set the data type as Percentage.

   DAX Calculated Column: gross_margin% = DIVIDE('fact_actuals&estimates'[gross_margin], 'fact_actuals&estimates'[net_sales], 0)

`Step 6: Optimizing Report File Size`

- Since the fact_actuals&estimates table is taking up to 73.5% of the file space, we need to remove some columns that are taking up cache space in the pbix file such that they can be dynamically calculated from the MySQL server as and when required.
    - Remove gross_price, pre_invoice_discount_pct, pre_invoice_discount in Power Query Editor.
- Columns created in PBI Frontend using DAX take up more space that those created in Power Query since it has a efficient data compression engine. Hence to save significant space we’ll remove some DAX columns that can be calculated later adhoc when required.
    - Remove cogs (since this is basically the sum of 3 columns), gross_margin and gross_margin% columns from the Data Model.

---

## Phase 4: Finance View

`Step 1: Creating Measures Table`

- To collect all report measures in a single place we’ll create a measures table to store all the measures together.
- Data View → Enter Data → Rename as measure.

`Step 2: Creating P & L Measures`

- Gross Sales Measure: GS $ = SUM('fact_actuals&estimates'[gross_sales])
- Net Invoice Sales Measure: NIS $ = SUM('fact_actuals&estimates'[net_invoice_sales])
- Pre-Invoice Deduction Measure: Pre-Invoice Deduction $ = [GS $ ] - [NIS $ ]
- Post-Invoice Deduction Measure: Post-Invoice Deduction $ = SUM('fact_actuals&estimates'[post_invoice_deductions])
- Post-Invoice Other Deduction Measure: Post-Invoice Other Deduction $ = SUM('fact_actuals&estimates'[post_invoice_other_deductions])
- Total Post-Invoice Deduction Measure: Total Post-Invoice Deduction $ = [Post-Invoice Deduction $ ] + [Post-Invoice Other Deduction $ ]
- Net Sales Measure: NS $ = SUM('fact_actuals&estimates'[net_sales])
- Manufacturing Cost Measure: Manufacturing Cost $ = SUM('fact_actuals&estimates'[manufacturing_cost])
- Freight Cost Measure: Freight Cost $ = SUM('fact_actuals&estimates'[freight_cost])
- Other Cost Measure: Other Cost $ = SUM('fact_actuals&estimates'[other_cost])
- Cost of Goods Sold Measure: COGS $ = [Manufacturing Cost $ ] + [Freight Cost $ ] + [Other Cost $ ]
- Gross Margin Measure: GM $ = [NS $ ] - [COGS $ ]
- Gross Margin % Measure: GM % = DIVIDE([GM $ ], [NS $ ], 0)
- Quantity/Units Measure: Quantity = SUM('fact_actuals&estimates'[qty])
- Gross Margin per Unit Measure: GM / Unit = DIVIDE([GM $ ], [Quantity], 0)

`Step 3: Creating P & L Rows Table`

1. Copy the P & L Table Structure from the excel file.
2. Data view →Enter Data → Paste → Rename as P & L Rows.

`Step 4: Building P&L Matrix visual`

1. Create a Matrix visual with ‘P & L Rows’[Line Item] as Row Parameter.
2. Now we need the corresponding values of line items in the next column, we can do this by using the SWITCH Fn to create a measure and binding the values to be shown based on the Order value of the Line Item.
3. P & L Values DAX Measure: P&L Values = SWITCH(TRUE( ),

   MAX('P & L Rows'[Order])=1,  [GS $ ]/1000000,
   
   MAX('P & L Rows'[Order])=2,  [Pre-Invoice Deduction $ ]/1000000,
   
   MAX('P & L Rows'[Order])=3,  [NIS $ ]/1000000,

   MAX('P & L Rows'[Order])=4,  [Post-Invoice Deduction $ ]/1000000,

   MAX('P & L Rows'[Order])=5,  [Post-Invoice Other Deduction $ ]/1000000,

   MAX('P & L Rows'[Order])=6,  [Total Post-Invoice Deduction $ ]/1000000,

   MAX('P & L Rows'[Order])=7,  [NS $ ]/1000000,

   MAX('P & L Rows'[Order])=8,  [Manufacturing Cost $ ]/1000000,

   MAX('P & L Rows'[Order])=9,  [Freight Cost $ ]/1000000,

   MAX('P & L Rows'[Order])=10,  [Other Cost $ ]/1000000,

   MAX('P & L Rows'[Order])=11,  [COGS $ ]/1000000,

   MAX('P & L Rows'[Order])=12,  [GM $ ]/1000000,

   MAX('P & L Rows'[Order])=13,  [GM %]*100,

   MAX('P & L Rows'[Order])=14,  [GM / Unit])
4. In the Data View → P & L Rows Table → Sort by Column → Order → Fixes the Line Item Order in the visual. Remove Row Subtotals.
5. Create Last Year Measure using the SAMEPERIODLASTYEAR Fn in the Filter context: P&L LY = CALCULATE([P & L Values], SAMEPERIODLASTYEAR(dim_date[date]))
6. Since our sales_actuals&estimates data has both actual and estimate data we want to show the fiscal years for which estimate is shown with a “Est” suffix. For this we can create a calculated column in the fiscal_year table that will dynamically concatenate the suffix to the latest date year. DAX Calculated Column:
   
   fy_desc = var MAXDATE = CALCULATE(MAX(fiscal_year[fiscal_year]), ALL(fiscal_year[fiscal_year]))
   RETURN IF(fiscal_year[fiscal_year] = MAXDATE, MAXDATE & " Est", fiscal_year[fiscal_year])
   
   Configure the Calculated Column as a Tile Slicer with Single Select option.
7. To compare the Last Year performance with Current year we would require a measure that calculates the difference in values. DAX Measure: P&L YoY Chg = [P&L Values] - [P&L LY]
   To compare the performance change in percentage. DAX Measure: P&L YoY Chg % = DIVIDE([P&L YoY Chg], [P&L LY], 0) * 100
8. We want the P&L Values Column to show dynamic year based on the year selected in the slicer, for this we need a dynamic table with a column that contains all these values.
   We will create table P&L Columns with column Col Header that will contain all the fiscal years as well as the LY, YoY Chg & Yoy Chg % values.
   
   DAX Code: P&L Columns = var fy = ALLNOBLANKROW(fiscal_year[fy_desc])
   RETURN UNION( ROW("Col Header", "LY"), ROW("Col Header", "YoY Chg"), ROW("Col Header", "YoY Chg %"), fy)

   This table has all the possible column header values however we still need to dynamically display the fiscal year based on the fy selected in the slicer. For this we’ll have to use the       SELECTEDVALUE Fn in a Measure to get the fy selected in the slicer and fetch the corresponding column header from the P&L Columns table.

   DAX Measure: P&L Final Value = SWITCH(TRUE(), SELECTEDVALUE(fiscal_year[fy_desc]) = MAX('P&L Columns'[Col Header]), [P&L Values])

   We also need to show to corresponding LY, YoY Chg and YoY Chg % values based on the FY selected based on the Filter context using their individual measures.

   Updated DAX Measure: P&L Final Value = SWITCH(TRUE( ), SELECTEDVALUE(fiscal_year[fy_desc])=MAX('P&L Columns'[Col Header]), [P&L Values],
   MAX('P&L Columns'[Col Header])="LY", [P&L LY],
   MAX('P&L Columns'[Col Header])="YoY Chg",[P&L YoY Chg],
   MAX('P&L Columns'[Col Header])="YoY Chg %",[P&L YoY Chg %])
