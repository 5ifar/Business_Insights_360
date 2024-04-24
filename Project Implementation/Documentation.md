# 🧬 Business Insights 360 Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Data Wrangling](#phase-1-data-wrangling)
- [Phase 2: ETL with Power Query](#phase-2-etl-with-power-query)
- [Phase 3: Data Modelling & Calculated Columns](#phase-3-data-modelling--calculated-columns)

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
