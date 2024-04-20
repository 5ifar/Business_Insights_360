# 🧬 Business Insights 360 Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Data Wrangling](#phase-1-data-wrangling)
- [Phase 2: ETL with Power Query](#phase-2-etl-with-power-query)

---

## Phase 1: Data Wrangling

`Step 1: Loading Data to MySQL Workbench`

1. Download the extract the .sql data files.
2. New Connection → Rename Business Insights 360 → Test the connection
3. Server → Data Import → Import from self-contained file → Select the .sql files from directory → Start Import → Once import complete → Check database in the Navigator Schema

`Step 2: Connecting MySQL Database to Power BI`

- Get Data → MySQL Database → Server: localhost & Database: gdb041 & gdb056 → Load all the tables

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
