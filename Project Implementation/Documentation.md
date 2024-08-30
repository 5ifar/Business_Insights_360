# 🧬 Business Insights 360 Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Data Wrangling](#phase-1-data-wrangling)
- [Phase 2: ETL with Power Query](#phase-2-etl-with-power-query)
- [Phase 3: Data Modelling & Calculated Columns](#phase-3-data-modelling--calculated-columns)
- [Phase 4: Finance View](#phase-4-finance-view)
- [Phase 5: Sales View](#phase-5-sales-view)
- [Phase 6: Marketing View](#phase-6-marketing-view)
- [Phase 7: Supply Chain View](#phase-7-supply-chain-view)
- [Phase 8: Designing Effective Dashboard](#phase-8-designing-effective-dashboard)

---

## Phase 1: Data Wrangling

### Step 1: Loading Data to MySQL Workbench

1. Download the extract the .sql data files.
2. New Connection → Rename Business Insights 360 → Test the connection
3. Server → Data Import → Import from self-contained file → Select the .sql files from directory → Start Import → Once import complete → Check database in the Navigator Schema

### Step 2: Connecting MySQL Database to Power BI

- Get Data → MySQL Database → Server: localhost & Database: gdb041 & gdb056 → Load all the tables

---

## Phase 2: ETL with Power Query

### Step 1: Creating custom Date Table

1. New Source → Blank Query → `Custom M Code: = {Number.From(#date(2017,9,1))..Number.From(#date(2022,12,31))}` → Will create a list of date values → Convert to Table → Set Data type as Date → Rename column as Date
2. Add columns → Day, Day Name, Start of Week, Start of Month & Month Name.
3. Add Custom fiscal_year column using M code: `Date.Year(Date.AddMonths([month],4))`

   NOTE: This basically adds 4 months to the Date Month and then extracts the Year out of the result. Our financial year starts in Sep hence we add 4 months.
4. Filter out the fiscal_year column for 2023 values since we have incomplete data for it.

### Step 2: Creating Last Sales Month Reference Table

1. Create a reference table from the fact_sales_monthly table. Extract the date column by editing the source step code to: `#"gdb041 fact_sales_monthly"[date]`
2. Calculate the Last Sales Month table by getting the maximum date value from the column using: `= List.Max(#"gdb041 fact_sales_monthly"[date])`

### Step 3: Creating Remaining Forecast Reference Table

1. Create a reference table from the fact_forecast_monthly table. Sort the table ascending by date values.
2. Filter for all the data after the Last Sales Month value using M code: `Table.SelectRows(Source, each ([date] > last_sales_month))`

### Step 4: Creating a New Table with both Actual & Forecast Data

1. Append the fact_sales monthly table and remaining_forecast table as a new table fact_actuals&estimates.
2. Rename sold_quantity (from fact_sales_monthly) and forecast_quantity (from remaining_forecast) as qty so the columns get merged into one column in the append operation.

### Step 5: Calculating Net Invoice Sales based on FY varying Gross Price & Pre-invoice Deductions

Logic: Gross Price - Pre-invoice Deductions = Net Invoice Sales → Net Invoice Sales - Post-invoice Deductions = Net Sales (will be done later)

1. Add a custom calculated column fiscal_year to the fact_actuals&estimates table to calculate the FY based on the date column using M code: `Date.Year(Date.AddMonths([date],4))`
2. Merge the fact_actuals&estimates table and the gross_price table based on 2 columns: product_code and fiscal_year with a Left Outer Join. Expand the merged columns to show only gross_price without the reference name. Set the data type as Currency. 
3. Add a custom calulated column gross_sales that is the product of gross_price and the qty sold columns. Set the data type as Currency.
4. Now Merge the fact_actuals&estimates table and the pre_invoice_deductions table based on 2 columns: customer_code and fiscal_year with a Left Outer Join. Expand the merged columns to show only pre_invoice_discount_pct without the reference name. Set the data type as Percentage.
5. Add a custom calculated column pre_invoice_discount that is the product of pre_invoice_discount_pct and gross_sales columns. Set the data type as Currency.
6. Add a custom calculated column net_invoice_sales that is the difference between gross_sales and pre_invoice_discount columns. Set the data type as Currency.

---

## Phase 3: Data Modelling & Calculated Columns

### Step 1: Normalizing Data in Tables

- When the Data Engineer provided us the raw data, it was de-normalized and hence the fact tables contain redundant data like customer name and product name.
- This redundant data increases the data refresh time and hence needs to be handled by Power Query:
  
  fact_sales_monthly Table: Choose Columns → Uncheck customer_name, channel, platform, market, product, category & division columns.
  
  fact_forecast_monthly Table: Choose Columns → Uncheck customer_name, channel, platform, market, product, category & division columns.

### Step 2: Creating Table Relationships

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

### Step 3: Creating fiscal_year table using DAX

- We want to create table relationships for fiscal year values, fiscal_year(dim_date) → fiscal_year(freight_cost) however this leads to a many-to-many relationship which is not advised.
- Hence we need a separate fiscal year table that would contain unique values. Data View → New Table → DAX: `fiscal_year = ALLNOBLANKROW(dim_date[fiscal_year])`
- Table Relationships: fiscal_year(fiscal_year) → fiscal_year(dim_date)     &     fiscal_year(fiscal_year) → fiscal_year(freight_cost)     &     fiscal_year(fiscal_year) → cost_year(manufacturing_cost)

### Step 4: Calculated Columns for post_invoice Calculations

1. We now need to get the Post invoice deductions data into the fact_actuals&estimates table. 

   For this DAX Calculated Column: `post_invoice_deductions_pct = CALCULATE(MAX(post_invoice_deductions[discounts_pct]), RELATEDTABLE(post_invoice_deductions))`

   NOTE: This could have also be done simply by merging both tables based on product_code, customer_code and date as merging parameters.
2. To the get the numeric value instead of pct we’ll multiple it with the net_invoice_sales value to get the updated DAX Calculated Column. Set the data type as Currency.

   `post_invoice_deductions = var pct = CALCULATE(MAX(post_invoice_deductions[discounts_pct]), RELATEDTABLE(post_invoice_deductions))`
   
   `return pct*'fact_actuals&estimates'[net_invoice_sales]`
3. We’ll create similar DAX Calculated Column for the other deductions amount. Set the data type as Currency.

   `post_invoice_other_deductions = var pct = CALCULATE(MAX(post_invoice_deductions[other_deductions_pct]), RELATEDTABLE(post_invoice_deductions))`
   
   `return pct*'fact_actuals&estimates'[net_invoice_sales]`
4. Now Net Sales will be all Post Invoice Deductions subtracted from Net Invoice Sales. Set the data type as Currency.

   DAX: `net_sales = 'fact_actuals&estimates'[net_invoice_sales]-'fact_actuals&estimates'[post_invoice_deductions]-'fact_actuals&estimates'[post_invoice_other_deductions]`

### Step 5: Calculated Columns for COGS & Gross Margin Calculations

1. We’ll calculate the Manufacturing Cost based on Qty of each Product. Set the data type as Currency.

   DAX Calculated Column: `manufacturing_cost = var res = CALCULATE(MAX(manufacturing_cost[manufacturing_cost]), RELATEDTABLE(manufacturing_cost))`
   
   `return res*'fact_actuals&estimates'[qty]`
2. We’ll calculate the Freight Cost on the Net Sales value. Set the data type as Currency.

   DAX Calculated Column: `freight_cost = var pct = CALCULATE(MAX(freight_cost[freight_pct]), RELATEDTABLE(freight_cost))`
   
   `return pct*'fact_actuals&estimates'[net_sales]`
3. We’ll calculate the Other Cost on the Net Sales value. Set the data type as Currency.
   
   DAX Calculated Column: `other_cost = var pct = CALCULATE(MAX(freight_cost[other_cost_pct]), RELATEDTABLE(freight_cost))`
   
   `return pct*'fact_actuals&estimates'[net_sales]`
4. We’ll calculate Cost of Goods Sold (COGS) = Manufacturing Cost + Freight (Transportation) + Other Costs. Set the data type as Currency.

   DAX Calculated Column: `cogs = 'fact_actuals&estimates'[manufacturing_cost]+'fact_actuals&estimates'[freight_cost]+'fact_actuals&estimates'[other_cost]`
5. Finally Gross Margin is the difference between Net Sales and COGS. Set the data type as Currency.

   DAX Calculated Column: `gross_margin = 'fact_actuals&estimates'[net_sales]-'fact_actuals&estimates'[cogs]`
6. Gross Margin % will be calculated over the Net Sales value. Set the data type as Percentage.

   DAX Calculated Column: `gross_margin% = DIVIDE('fact_actuals&estimates'[gross_margin], 'fact_actuals&estimates'[net_sales], 0)`

### Step 6: Optimizing Report File Size

- Since the fact_actuals&estimates table is taking up to 73.5% of the file space, we need to remove some columns that are taking up cache space in the pbix file such that they can be dynamically calculated from the MySQL server as and when required.
    - Remove gross_price, pre_invoice_discount_pct, pre_invoice_discount in Power Query Editor.
- Columns created in PBI Frontend using DAX take up more space that those created in Power Query since it has a efficient data compression engine. Hence to save significant space we’ll remove some DAX columns that can be calculated later adhoc when required.
    - Remove cogs (since this is basically the sum of 3 columns), gross_margin and gross_margin% columns from the Data Model.

---

## Phase 4: Finance View

### Step 1: Creating Measures Table

- To collect all report measures in a single place we’ll create a measures table to store all the measures together.
- Data View → Enter Data → Rename as measure.

### Step 2: Creating P & L Measures

- Gross Sales Measure: `GS $ = SUM('fact_actuals&estimates'[gross_sales])`
- Net Invoice Sales Measure: `NIS $ = SUM('fact_actuals&estimates'[net_invoice_sales])`
- Pre-Invoice Deduction Measure: `Pre-Invoice Deduction $ = [GS $ ] - [NIS $ ]`
- Post-Invoice Deduction Measure: `Post-Invoice Deduction $ = SUM('fact_actuals&estimates'[post_invoice_deductions])`
- Post-Invoice Other Deduction Measure: `Post-Invoice Other Deduction $ = SUM('fact_actuals&estimates'[post_invoice_other_deductions])`
- Total Post-Invoice Deduction Measure: `Total Post-Invoice Deduction $ = [Post-Invoice Deduction $ ] + [Post-Invoice Other Deduction $ ]`
- Net Sales Measure: `NS $ = SUM('fact_actuals&estimates'[net_sales])`
- Manufacturing Cost Measure: `Manufacturing Cost $ = SUM('fact_actuals&estimates'[manufacturing_cost])`
- Freight Cost Measure: `Freight Cost $ = SUM('fact_actuals&estimates'[freight_cost])`
- Other Cost Measure: `Other Cost $ = SUM('fact_actuals&estimates'[other_cost])`
- Cost of Goods Sold Measure: `COGS $ = [Manufacturing Cost $ ] + [Freight Cost $ ] + [Other Cost $ ]`
- Gross Margin Measure: `GM $ = [NS $ ] - [COGS $ ]`
- Gross Margin % Measure: `GM % = DIVIDE([GM $ ], [NS $ ], 0)`
- Quantity/Units Measure: `Quantity = SUM('fact_actuals&estimates'[qty])`
- Gross Margin per Unit Measure: `GM / Unit = DIVIDE([GM $ ], [Quantity], 0)`

### Step 3: Creating P & L Rows Table

1. Copy the P & L Table Structure from the excel file.
2. Data view → Enter Data → Paste → Rename as P & L Rows.

### Step 4: Building P&L Matrix visual

1. Create a Matrix visual with ‘P & L Rows’[Line Item] as Row Parameter.
2. Now we need the corresponding values of line items in the next column, we can do this by using the SWITCH Fn to create a measure and binding the values to be shown based on the Order value of the Line Item.
3. P & L Values DAX Measure: `P&L Values = SWITCH(TRUE( ),`

   `MAX('P & L Rows'[Order])=1,  [GS $ ]/1000000,`
   
   `MAX('P & L Rows'[Order])=2,  [Pre-Invoice Deduction $ ]/1000000,`
   
   `MAX('P & L Rows'[Order])=3,  [NIS $ ]/1000000,`

   `MAX('P & L Rows'[Order])=4,  [Post-Invoice Deduction $ ]/1000000,`

   `MAX('P & L Rows'[Order])=5,  [Post-Invoice Other Deduction $ ]/1000000,`

   `MAX('P & L Rows'[Order])=6,  [Total Post-Invoice Deduction $ ]/1000000,`

   `MAX('P & L Rows'[Order])=7,  [NS $ ]/1000000,`

   `MAX('P & L Rows'[Order])=8,  [Manufacturing Cost $ ]/1000000,`

   `MAX('P & L Rows'[Order])=9,  [Freight Cost $ ]/1000000,`

   `MAX('P & L Rows'[Order])=10,  [Other Cost $ ]/1000000,`

   `MAX('P & L Rows'[Order])=11,  [COGS $ ]/1000000,`

   `MAX('P & L Rows'[Order])=12,  [GM $ ]/1000000,`

   `MAX('P & L Rows'[Order])=13,  [GM %]*100,`

   `MAX('P & L Rows'[Order])=14,  [GM / Unit])`
4. In the Data View → P & L Rows Table → Sort by Column → Order → Fixes the Line Item Order in the visual. Remove Row Subtotals.
5. Create Last Year Measure using the SAMEPERIODLASTYEAR Fn in the Filter context: `P&L LY = CALCULATE([P & L Values], SAMEPERIODLASTYEAR(dim_date[date]))`
6. Since our sales_actuals&estimates data has both actual and estimate data we want to show the fiscal years for which estimate is shown with a “Est” suffix. For this we can create a calculated column in the fiscal_year table that will dynamically concatenate the suffix to the latest date year. DAX Calculated Column:
   
   `fy_desc = var MAXDATE = CALCULATE(MAX(fiscal_year[fiscal_year]), ALL(fiscal_year[fiscal_year]))`
   
   `RETURN IF(fiscal_year[fiscal_year] = MAXDATE, MAXDATE & " Est", fiscal_year[fiscal_year])`
   
   Configure the Calculated Column as a Tile Slicer with Single Select option.
7. To compare the Last Year performance with Current year we would require a measure that calculates the difference in values. DAX Measure: `P&L YoY Chg = [P&L Values] - [P&L LY]`
   To compare the performance change in percentage. DAX Measure: `P&L YoY Chg % = DIVIDE([P&L YoY Chg], [P&L LY], 0) * 100`
8. We want the P&L Values Column to show dynamic year based on the year selected in the slicer, for this we need a dynamic table with a column that contains all these values.
   We will create table P&L Columns with column Col Header that will contain all the fiscal years as well as the LY, YoY Chg & Yoy Chg % values.
   
   DAX Code: `P&L Columns = var fy = ALLNOBLANKROW(fiscal_year[fy_desc])`
   
   `RETURN UNION( ROW("Col Header", "LY"), ROW("Col Header", "YoY Chg"), ROW("Col Header", "YoY Chg %"), fy)`

   This table has all the possible column header values however we still need to dynamically display the fiscal year based on the fy selected in the slicer. For this we’ll have to use the       SELECTEDVALUE Fn in a Measure to get the fy selected in the slicer and fetch the corresponding column header from the P&L Columns table.

   DAX Measure: `P&L Final Value = SWITCH(TRUE(), SELECTEDVALUE(fiscal_year[fy_desc]) = MAX('P&L Columns'[Col Header]), [P&L Values])`

   We also need to show to corresponding LY, YoY Chg and YoY Chg % values based on the FY selected based on the Filter context using their individual measures.

   Updated DAX Measure: `P&L Final Value = SWITCH(TRUE( ), SELECTEDVALUE(fiscal_year[fy_desc])=MAX('P&L Columns'[Col Header]), [P&L Values],`
   
   `MAX('P&L Columns'[Col Header])="LY", [P&L LY],`
   
   `MAX('P&L Columns'[Col Header])="YoY Chg",[P&L YoY Chg],`
   
   `MAX('P&L Columns'[Col Header])="YoY Chg %",[P&L YoY Chg %])`

### Step 5: Configuring Quarters & YTD/YTG Slicers

AtliQ’s Financial Year starts is from Sep to Aug.

1. We’ll create a new column in the dim_date table which will contain the fiscal month number obtained by adding 4 months to the dates and extracting the month part of it.

   DAX Calculated Column: `fy_month_num = MONTH(DATE(YEAR(dim_date[date]), MONTH(dim_date[date])+4, 1))`
2. To calculate the quarters we need to divide the fiscal month nbr by 3 and round it up to the next integer. 
   
   DAX Calculated Column: `quarters = “Q” & ROUNDUP(dim_date[fy_month_num]/3, 0)`

   Add this Calculated Column as a Tile Slicer to the Finance View.
3. Now Sep 1st to the last sales date will be YTD, and the rest of the days until 31st August should be YTG. 
   Hence, the logic for generating YTD and YTG is when a date in the date column is greater than the last sales then it is YTG, otherwise it is YTD.
4. However, if you compare the date column with the last sales date you will get the YTD and YTG only for the current fiscal year which is 2022. Since you need it for all fiscal years – a column that works for all fiscal years is needed i.e. a column that is not attributed to the year but just the months. You can see that it has the fiscal year month number which is the same for all fiscal years. i.e fy_month_num for Dec 21 and Dec 20 is the same as it takes the month into the account and not the year. So, we will compare the fy_month_num of the date column vs fy_month_num of the last sales date to find if a given date is YTD or YTG.
5. Now we need to use the if condition to check whether the fy_month_num of a date is greater than FYMONTHNUM of last sales date in order to assign it as YTG or else YTD.

   DAX Calculated Column: `ytd_ytg = `
   
   `var LASTSALESDATE = MAX(fact_sales_monthly[date])`
   
   `var FYMONTHNUM = MONTH(DATE(YEAR(LASTSALESDATE), MONTH(LASTSALESDATE)+4, 1))`
   
   `RETURN IF(dim_date[fy_month_num] > FYMONTHNUM, "YTG", "YTD")`
   
   Add this Calculated Column as a Tile Slicer to the Finance View.

### Step 6: Building P&L Performance over Time visual

1. Add a clustered column chart visual with P&L Values Measure on the Y Axis and date on the X Axis. Format the X Axis type as Categorical to see distinct columns for all months.
2. Format the date as mmm yy format to show shortened dates in the visual.
3. Change the report setting to select Change default visual interaction from cross highlighting to cross filtering to avoid highlighting when filtering based on a single line item of P&L       Values.
4. Currently if we dont select any line item it shows a total of all P&L Values by default which is not very useful. We want the default to be Net Sales, for this we need to edit the P&L Values measure and add a condition to ensure Net Sales is selected when no line item is selected. Updated P&L Values Measure: `P&L Values = `
   
   `var res = SWITCH(TRUE( ),`

   `MAX('P&L Rows'[Order])=1,  [GS $]/1000000,`

   `MAX('P&L Rows'[Order])=2,  [Pre-Invoice Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=3,  [NIS $]/1000000,`

   `MAX('P&L Rows'[Order])=4,  [Post-Invoice Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=5,  [Post-Invoice Other Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=6,  [Total Post-Invoice Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=7,  [NS $]/1000000,`

   `MAX('P&L Rows'[Order])=8,  [Manufacturing Cost $]/1000000,`

   `MAX('P&L Rows'[Order])=9,  [Freight Cost $]/1000000,`

   `MAX('P&L Rows'[Order])=10,  [Other Cost $]/1000000,`

   `MAX('P&L Rows'[Order])=11,  [COGS $]/1000000,`

   `MAX('P&L Rows'[Order])=12,  [GM $]/1000000,`

   `MAX('P&L Rows'[Order])=13,  [GM %]*100,`

   `MAX('P&L Rows'[Order])=14,  [GM / Unit])`

   `RETURN IF(HASONEVALUE('P&L Rows'[Description]), res, [NS $]/1000000)`
5. Now the visual shows the data for the line item selected but the title does not get updated automatically. For this we need to create another measure which will provide title as per          selection that is made out of the line items.

   DAX Measure: `P&L Selected Row = IF(HASONEVALUE('P&L Rows'[Description]), SELECTEDVALUE('P&L Rows'[Description]), "Net Sales")`

   We’ll then create a supporting Measure to customize the title for specific visual: `Performance Visual Title = [P&L Selected Row] & " Performance over Time”`
6. Now we’ll update this Performance Visual Title measure as the parameter to base the Title on using the Title fx Conditional Formatting.
7. We also need to compare performance figures from last year so we’ll add the P&L LY Measure before the P&L Values measure on Y Axis.
8. Since our objective behind this visual was to compare the performance over time, using an Area Chart here would be better since its good for comparing values over time as well as total value using the plot area.
9. Formatting for Area chart: Move Legend position to Top Center. Remove X  & Y Axis Titles.

### Step 7: Building Top Market & Product visuals

1. Add a Matrix visual with market & customer fields as Rows and P&L Values & P&L YoY Chg % Measures as Values.
2. Formatting for Matrix visual: Disable Row & Column Subtotals. 
3. Add a Matrix visual with segment, category & product fields as Rows and P&L Values & P&L YoY Chg % Measures as Values.
4. Formatting for Matrix visual: Disable Row & Column Subtotals.

### Step 8: Importing Operating Expenses Data

- Get Data → Excel Workbook → Select the Operational Expenses Excel File → Transform → Rename to operational_expense → Any transformation if required → Load
- Data Modelling:

|Primary Key (Dimension table) (1)||Secondary Key (Fact table) (*)|
|-|-|-|
|fiscal_year (fiscal_year)|→|fiscal_year (operational_expense)|
|market (dim_market)|→|market (operational_expense)|

### Step 9: Calculated Columns & Measures for Operational Expenses & Net Profit Calculations

1. We’ll calculate the Ads & Promotions Cost on the Net Sales value. Set the data type as Currency.
   
   DAX Calculated Column: `ads_promotions = var pct = CALCULATE(MAX(operational_expense[ads_promotions_pct]), RELATEDTABLE(operational_expense))`
   
   `return pct*'fact_actuals&estimates'[net_sales]`
2. We’ll calculate the Other Operational Expenses on the Net Sales value. Set the data type as Currency.
   
   DAX Calculated Column: `other_operational_expense = var pct = CALCULATE(MAX(operational_expense[other_operational_expense_pct]), RELATEDTABLE(operational_expense))`
   
   `return pct*'fact_actuals&estimates'[net_sales]`
3. Ads & Promotions Expense Measure: `Ads & Promotions $ = SUM('fact_actuals&estimates'[ads_promotions])`
4. Other Operational Expense Measure: `Other Operational Expense $ = SUM('fact_actuals&estimates'[other_operational_expense])`
5. Operational Expenses Measure: `Operational Expense $ = ([Ads & Promotions $ ] + [Other Operational Expense $ ]) * -1`

   We’ll multiply with -1 to ensure this is treated as unfavorable expense.
6. Net Profit Measure: `Net Profit $ = [GM $ ] + [Operational Expense $ ]`

   Here we want to subtract the Operational Expense but since it already has negative value we can add it.
7. Net Profit % Measure: `Net Profit % = DIVIDE([Net Profit $ ], [NS $ ], 0)`

### Step 10: Updating the P&L visual with Operating Expenses and Net Profit

1. Since we want to add the 3 fields to the P&L Line items, they need to be appended to the P&L Rows tables that is being used in visual rows field.
2. In Transform Power Query, go to P&L Rows and in the Source step edit the table to add 3 new rows: Operational Expense (Order: 15), Net Profit (Order: 16) & Net Profit % (Order: 17).
3. Now we edit the P&L Values Measure to include these 3 new fields. Updated Measure: `P&L Values =`

   `var res = SWITCH(TRUE( ),`

   `MAX('P&L Rows'[Order])=1,  [GS $]/1000000,`

   `MAX('P&L Rows'[Order])=2,  [Pre-Invoice Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=3,  [NIS $]/1000000,`

   `MAX('P&L Rows'[Order])=4,  [Post-Invoice Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=5,  [Post-Invoice Other Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=6,  [Total Post-Invoice Deduction $]/1000000,`

   `MAX('P&L Rows'[Order])=7,  [NS $]/1000000,`

   `MAX('P&L Rows'[Order])=8,  [Manufacturing Cost $]/1000000,`

   `MAX('P&L Rows'[Order])=9,  [Freight Cost $]/1000000,`

   `MAX('P&L Rows'[Order])=10,  [Other Cost $]/1000000,`

   `MAX('P&L Rows'[Order])=11,  [COGS $]/1000000,`

   `MAX('P&L Rows'[Order])=12,  [GM $]/1000000,`

   `MAX('P&L Rows'[Order])=13,  [GM %]*100,`

   `MAX('P&L Rows'[Order])=14,  [GM / Unit],`

   `MAX('P&L Rows'[Order])=15,  [Operational Expense $]/1000000,`

   `MAX('P&L Rows'[Order])=16,  [Net Profit $]/1000000,`

   `MAX('P&L Rows'[Order])=17,  [Net Profit %]*100)`

   `RETURN`

   `IF(HASONEVALUE('P&L Rows'[Description]), res, [NS $]/1000000)`

---

## Phase 5: Sales View

### Step 1: Building Customer Performance visual

1. Add a Matrix visual with customer field as Rows and Net Sales, Gross Margin & Gross Margin % Measures as values.
2. Change the display units for NS $ & GM $ to Millions. Format → Specific Column → Select Column → Value Display Unit to Millions with 2 decimal points.

### Step 2: Building Customers GM & NS Plot visual

1. Copy the Top Customers visual and convert it to Scatter Plot. Add market field before customer field in values to aid drill down and fix congestion. Enable Category labels and Zoom Sliders.
2. We need Net Sales values as the X Axis and the Gross Margin % values as the Y Axis and the Gross Margin $ values as the Bubble size. Add region field as the Legend.
3. Copy the FY, Quarters & YTD-YTG Slicers from Finance View to Sales View and Sync them across pages.
4. Add an additional Region, Market & Customer fields Dropdown Slicers for country-specific business users.

### Step 3: Building Product Performance visual

- Copy the Customer Performance visual and replace the customer field in Rows by segment, category & product fields to aid drill down on product level.

### Step 4: Building Unit Economics visual

1. Add a Donut chart with P&L Values measure as the Values and Description field as Legend. Now in the Filters pane, filter the Description field to only show Net Sales, Pre Invoice Deduction & Total Post Invoice Deduction.
2. Set Detail Labels Display Units as None. Align the legend to top center. Disable legend title.
3. Copy the above visual and change Description field filter to only show Total COGS & Gross Margin. This is a breakdown of Net Sales from above visual.

---

## Phase 6: Marketing View

Duplicate the Sales View. Remove the Customer Performance Matrix visual.

### Step 1: Building Product Performance visual

- Add Net Profit and Net Profit % values to the Product Performance visual copied from Sales View.

### Step 2: Building Products GM & NS Plot visual

- Replace the existing values field by segment, category and product fields. Add division field to the legend.

### Step 3: Building Unit Economics visual

- Replace the Net Sales and Deduction Donut chart by a Waterfall chart with Gross Margin, Operation Cost & Net Profit fields to show the flow of expenses.
- Sort the waterfall by P&L Rows Order column such that the Gross Margin is on the left and Net Profit is on right.

### Step 4: Building Market Performance visual

- Copy the Product Performance visual and change the Rows to region and market fields.

---

## Phase 7: Supply Chain View

### Step 1: Understanding Key Metrics

- We already have a quantity measure but it is not the actual sales quantity because it has been calculated based on the fact_actuals&estimates that takes into consideration the estimate/forecast data as well. We only need the actual data sales qty.
- To extract the actual sales qty from the fact_actuals&estimates table we need to ignore the data after the last sales date. To get this date we’ll enable the load of the last_sales_month table from Power Query.
- Now since we have both fact_actuals&estimates & last_sales_month tables we don't really need the fact_sales_monthly table and can disable its load to reduce the file size. But first we need to replace its usage from the ytd_ytg calculated column. So we’ll update to: `var LASTSALESDATE = MAX(last_sales_month[last_sales_month])` in the DAX code.
- Now we can disable the fact_sales_monthly table loading from Power Query.
- We’ll create a new measure to calculate the actual sales quantity for dates before the last_sales_month date.

  Sales Qty Measure: `Sales Qty = CALCULATE([Quantity], 'fact_actuals&estimates'[date] <= MAX(last_sales_month[last_sales_month]))`
- Similarly we create a measure to calculate the forecast sales quantity without using an filters based on entire fact_forecast_monthly table.

  Forecast Qty Measure: `Forecast Qty = SUM(fact_forecast_monthly[forecast_quantity])`

### Step 2: Creating Supply Chain Measures

- Net Error is the difference between the Forecast and Actual Sales figures. Net Error Measure: `Net Error = [Forecast Qty] - [Sales Qty]`
- Net Error % is calculated over the Forecast figure. Net Error % Measure: `Net Error % = DIVIDE([Net Error], [Forecast Qty], 0)`
- We’ll modify Forecast Qty Measure to restrict it till last sales date since we want to compare it with the Actual Sales.

  Updated Forecast Qty Measure: `Forecast Qty = `

  `var lsalesdate = MAX(last_sales_month[last_sales_month])`

  `RETURN CALCULATE(SUM(fact_forecast_monthly[forecast_quantity]), fact_forecast_monthly[date] <= lsalesdate)`
- For the Absolute Error Measure, in AtliQ as per the supply chain team’s requirement - the ABS Error needs to be measured for each product at the monthly level.  In other words, we need to convert the Net error to ABS Error at the product and month level granularity.

  To apply the formula to Products and Months, we need a list of products and Months. Hence `DISTINCT(dim_product[product_code])` and `DISTINCT(dim_date[month])` are used.

  Now, you need to iterate the `ABS([Net Error])` for each product and sum them up. Hence, SUMX is required.
- Absolute Error Measure: `Abs Error = SUMX(DISTINCT(dim_date[date]), SUMX(DISTINCT(dim_product[product_code]), ABS([Net Error])))`
- Absolute Error % is also calculated over the Forecast figure.Absolute Error Measure: `Abs Error % = DIVIDE([Abs Error], [Forecast Qty], 0)`
- Forecast Accuracy is logical opposite of Absolute Error. However doing simply, `Forecast Accuracy % = 1 - [Abs Error %]` gives us some blank rows in visualizations and the Forecast Accuracy % as 1 or 100%. This happens due to the fact that products without sales or forecast is included in this calculation. Because for products without forecast or sales, the ABS Error % is blank. By that logic, Forecast Accuracy = 1- Blank( ) which returns 1. Hence we’ll add an IF condition such that if the `[ABS Error %]` is blank the formula will also return blank and hence it won’t be displayed in the table.

  Updated Forecast Accuracy % Measure: `Forecast Accuracy % = IF([Abs Error %] <> BLANK( ), 1 - [Abs Error %], BLANK( ))`

### Step 3: Building Supply Chain visuals

1. Add Forecast Accuracy %, Net Error & Abs Error KPI Cards to the top left Supply Chain View page area.
2. Copy the Customer Performance visual from Sales View. Replace the values fields by Forecast Accuracy %, Net Error and Net Error %.
3. Copy the Product Performance visual from Sales View. Replace the values fields by Forecast Accuracy %, Net Error and Net Error %.
4. Add a Line & Clustered Column chart with Months field on X Axis (Set Value type as Categorical), Net Error on Column Y Axis and Forecast Accuracy % on Line Y Axis.

   We also need the Forecast Accuracy Year Ago % value on the Line Y Axis, for this we need to create a new measure. Forecast Accuracy % LY Measure:

   `Forecast Accuracy % LY = CALCULATE([Forecast Accuracy %], SAMEPERIODLASTYEAR(dim_date[date]))`

   Add this measure to both the Customer and Product Performance visuals.
5. We’ll create a new Risk measure to display if the item is in excess inventory or out of stock based on the Net Error measure value. Risk Measure:

   `Risk = IF([Net Error]>0, "EI", IF([Net Error]<0, "OOS", BLANK()))`

   Add this Risk Measure to both the Customer and Product Performance visuals.
6. Add Footer as: BM: Benchmark | LY: Last Year | EI: Excess Inventory | OOS: Out of Stock

---

## Phase 8: Designing Effective Dashboard

### Step 1: Building Home View landing page

- Set Custom View background. Page Formatting → Canvas Background → Select Image → Adjust transparency to 50%
- Now we’ll build the navigation setup. This needs to be done since we’ll be configuring Nav button actions to navigate to each of the pages and if we build this on anyone of those pages we won’t have the option to configure the button to navigate to same page it was built on. So building it on a new page gives us the option to configure navigation to all the other pages.
- Insert a Blank Button, in button formatting change state to Default and set Icon Type as Custom and Browse to select custom icon. Setup similarly custom icons for On Hover and On Press state. Follow this process for Info, Economy, Acquisition, Advertising, Executive, Supply Chain & Help Icons to the view. Size all to Height 100 & Width 120. Add all the Icon View Descriptions.
- Add AtliQ Logo to the top left page corner. Add Business Insights 360 Report title. Format the color and size as required.
- Configure Page Navigation by enabling Button Action as Type: Page Navigation and set the Destination as Required Page.

  Economy Icon → Finance View, Acquisition Icon → Sales View, Advertising Icon → Marketing View, Supply Chain Icon → Supply Chain View
- Add a horizontal line shape to demarcate the footer.
- Add Last Sales Month value to the page footer using the measure `Last Sales Month Footer = "Sales data loaded until: " & FORMAT(MAX(last_sales_month[last_sales_month]), "MMM YY")` as a Simple Card visual.

  Upgraded Design Dashboard measure: `Last Sales Month Footer = FORMAT(MAX(last_sales_month[last_sales_month]), "MMM YYYY")`
- Add “Values are in Millions. Currency is USD.” text to the footer.
- To add the Date of Report Refresh create a Blank query in the Power Query Editor. In the Advanced Editor add M - code:
    
  `let Source = #table(type table[Report Last Refreshed=datetime], {{DateTime.LocalNow()}})
  in Source`
    
  Add the values as a Simple Card visual.

### Step 2: Upgrading Finance View

- Set Custom View background. Page Formatting → Canvas Background → Select Image → Adjust transparency to 50%
- Make space for Left Nav Bar and 3 KPI Cards by formatting existing visuals.
- Change P&L Matrix Style Preset to Default/Transparent and adjust the padding to 2 to avoid any scroll bars. Add visual shadow.
- Resize Top Markets & Top Products visuals to equal size and use format painter to copy the formatting from P&L Matrix visual. Add Region Field to Top Market Rows.
- For Net Sales Performance visual, add visual shadow. Set line and shade area color according to the background color palette.

### Step 3: Adding Key Elements to Finance Dashboard

- Add markers to the Performance over Time Area chart visual.
- Add custom titles for Top Customers/Products visuals based on the line item selected in the P&L Matrix visual. For this we’ll create 2 new measures and configure them as conditionally formatted tiles for the corresponding visuals.

  Top / Bottom Customers Visual Title Measure: `Top / Bottom Customers Visual Title = "Customers by " & [P&L Selected Row]`

  Top / Bottom Products Visual Title Measure: `Top / Bottom Products Visual Title = "Products by " & [P&L Selected Row]`
- Update the P&L Matrix title to P&L Statement.
- Update Slicers field grouping. Add Market field to Region Slicer. Replace Market Slicer by Product Slicer. Add Segment and Category fields to the Product Slicer. Remove background and format Value Background color to White.
- Remove the background from FY-Q-YTX Slicers. Edit Value text to Bold and Background color to White.
- We’ll add 3 Key Performance Indicators:

1. Net Sales:

   Set NS $ Measure as the Value field and fiscal_year in the Trend axis field. In the target field we need Net Sales from last year so for this we’ll create a new measure:

   `NS $ LY = CALCULATE([NS $], SAMEPERIODLASTYEAR(dim_date[date]))`

   Change the Target label to LY.

2. Gross Margin %:

   Set GM % Measure as the Value field and fiscal_year in the Trend axis field. In the target field we need Gross Margin % from last year so for this we’ll create a new measure:

   `GM % LY = CALCULATE([GM %], SAMEPERIODLASTYEAR(dim_date[date]))`

   Change the Target label to LY.

3. Net Profit %:

   Set Net Profit % Measure as the Value field and fiscal_year in the Trend axis field. In the target field we need Net Profit % from last year so for this we’ll create a new measure:

   `Net Profit % LY = CALCULATE([Net Profit %], SAMEPERIODLASTYEAR(dim_date[date]))`

   Change the Target label to LY.

### Step 4: Configuring Navigation Bar

- Add a Rounded rectangle shape the left edge of the view with shape rounded corners set to 20%. Set Fill color to white with 60% transparency and Border color to Dark Red.
- Add the Home, Finance, Sales, Marketing, Supply Chain & Executive Icon Buttons to the Nav Bar and set Alignment and distribution. Set appropriate custom color icons for the hover and press states. Set the default state also as the custom color icon according to the view you're on.
- Copy the Nav Bar to all the views. Set the Home Icon button navigation to the Home view.

### Step 5: Upgrading Supply Chain View

- Update the Supply Chain Card to KPI Cards. Configure Forecast Accuracy KPI Card with Forecast Accuracy % measure as the value field and Forecast Accuracy % LY measure as the target field.
- Configure Net Error KPI Card with Net Error measure as the value field and new measure: `Net Error LY = CALCULATE([Net Error], SAMEPERIODLASTYEAR(dim_date[date]))` as the target field.

  Also since less Net Error is good we’ll configure the Trend Axis direction as Low is good so that lower values are shown as green and the Distance to goal direction as Increasing is
  positive such that the percentage change is shown with correct +ve/-ve sign.
- Configure Abs Error KPI Card with Abs Error measure as the value field and new measure: `Abs Error LY = CALCULATE([Abs Error], SAMEPERIODLASTYEAR(dim_date[date]))` as the target field.
  Also since less Abs Error is good we’ll configure the Trend Axis direction as Low is good so that higher values are shown as red and the Distance to goal direction as Increasing is
  positive such that the percentage change is shown with correct +ve/-ve sign.

### Step 6: Upgrading Sales View

- Change Customer and Product Performance visual columns to only show 2 decimal values.
- Change legend location to top center for Unit Economics visuals and add Net Sales Breakdown Arrow.
- Update the Sales View Nav Bar icon to color state and setup navigation.
- Set the current Canvas background and copy the same formatting across all visuals using Format Painter.

### Step 7: Upgrading Marketing View

- Change legend location to top center for Unit Economics visuals and add Gross Margin Breakdown Arrow.
- Update the Marketing View Nav Bar icon to color state and setup navigation.
- Set the current Canvas background and copy the same formatting across all visuals using Format Painter.

### Step 8: Incorporating Country level NS, GM & NP Target FY 2022 Data in Finance View

- Import the Targets FY 2022 file to the Power Query Editor and load it to the front end after making suitable changes.
- In the Modelling view, create relationships: market (dim_market) → market (ns_gm_target) & date (dim_date) → month (ns_gm_target)
- Create measure for Net Sales Target: `NS $ Target = SUM(ns_gm_target[ns_target])` with Currency data type and 2 decimals.
- Create measure for Gross Margin Target: `GM $ Target = SUM(ns_gm_target[gm_target])` with Currency data type and 2 decimals.
- Create measure for Net Profit target: `Net Profit $ Target = SUM(ns_gm_target[np_target])` with Currency data type and 2 decimals.
- The NS $ KPI Card can be filtered based on Region since we have regional data for target however it cannot be further filtered based on Customer since then the NS $ value will be shown for that particular customer but the target value will be shown for the entire region level which will result in incorrect Change % calculation.  E.g. Regional target of 30M$ does not imply 30M$/equally divided target for all the customers in the region.

  However the GM % and Net Profit % KPI Cards can be filtered based on Region and further on Customer since e.g. a Regional target of 30% can also be seen as 30% target for all the
  customers in the region.
- Create measure for Gross Margin % target: `GM % Target = DIVIDE([GM $ Target], [NS $ Target], 0)` with Percentage 2 decimal values data type.
- Create measure for Net Profit % target: `Net Profit % Target = DIVIDE([Net Profit $ Target], [NS $ Target], 0)` with Percentage 2 decimal values data type.

### Step 9: Updating Finance View visuals

- Update the KPI Cards target label from LY to BM (Benchmark) since it can be used to represent both LY and Target values.
- Add Footer text BM: Benchmark LY: Last Year
- Update the KPI Cards Targets to NS $ Target, GM % Target & Net Profit % Target Measures respectively.

  Now when we filter based on Region and Customers the Formatting and Distance to goal will be shown correctly as per target values for both Gross Margin % and Net Profit % Cards but it will   be incorrect for Net Sales Card as explained above at Point 6.

### Step 10: Creating a Dynamic Switch to toggle between LY and Target benchmarks

1. Home → Enter Data → Set table name as Set BM → Column names: benchmarks and ID → Benchmark: vs LY, ID: 1 & Benchmark: vs Target, ID: 2 → Load
2. Add the Benchmarks column to the page as a Tile slicer with Single select enabled.
3. Now we need to create new measure that will toogle values between Target and LY based on the slicer value selected out of Set BM [ID] column.
4. Create Net Sales Benchmark measure: `NS $ BM = SWITCH(TRUE( ), SELECTEDVALUE('Set BM'[ID]) = 1, [NS $ LY], SELECTEDVALUE('Set BM'[ID]) = 2, [NS $ Target])`

   Add the measure to the Net Sales KPI Card Target parameter.
5. However when we filter based on customer or product the Net Sales KPI card shows inaccurate value. To fix this we need a measure that check if any customer direct filter or product direct/indirect filter is being applied. Here we’ll allow indirect filter on customer since filtering region leads to filtered customers.

   `Customer / Product Filter Check = ISFILTERED(dim_customer[customer]) || ISCROSSFILTERED(dim_product[product])`
6. We’ll use this Filter Check in the NS $ Target measure so that when there is a filter applied then blank/no value will be shown other use the regular formula.

   `Updated measure for Net Sales Target: NS $ Target = `

   `var target = SUM(ns_gm_target[ns_target])`

   `return IF([Customer / Product Filter Check], BLANK( ), target)`
7. However with the above update formula the NS $ Target measure value will become BLANK whenever there is any customer/product specific filters applied this will in turn break our GM %         Target and Net Profit % Target measures and they will show target as 0 %. To fix this we’ll replace NS $ Target in these  measures with its base formula which is not affected by filters.

   Updated measure for Gross Margin % target: `GM % Target = DIVIDE([GM $ Target], SUM(ns_gm_target[ns_target]), 0)`

   Updated measure for Net Profit % target: `Net Profit % Target = DIVIDE([Net Profit $ Target], SUM(ns_gm_target[ns_target]), 0)`
8. Create Gross Margin % Benchmark measure: `GM % BM = SWITCH(TRUE( ), SELECTEDVALUE('Set BM'[ID]) = 1, [GM % LY], SELECTEDVALUE('Set BM'[ID]) = 2, [GM % Target])`

   Add the measure to the Gross Margin % KPI Card Target parameter.
9. Create Net Profit % Benchmark measure: `Net Profit % BM = SWITCH(TRUE( ), SELECTEDVALUE('Set BM'[ID]) = 1, [Net Profit % LY], SELECTEDVALUE('Set BM'[ID]) = 2, [Net Profit % Target])`

   Add the measure to the Net Profit % KPI Card Target parameter.
10. When we change the FY slicer to any year other than 2022 Est the Target benchmarks are not available and the KPI Cards show Blank value in Target and the Distance to goal is shown as         infinity %. This can be confusing and can be clarified by showing a text msg warning when the target values are blank using a new BM Message measure:

    `BM Message = IF([NS $ BM] = BLANK( ) || [GM % BM] = BLANK() || [Net Profit % BM] = BLANK(), "BM Target(s) is not available for the selected filters", "")`

### Step 11: Configuring BM instead of LY for other Finance View visuals

P&L Statement, Customers/products by Net Sales visuals:

1. For the P&L Statement visual the key measure used currently is P & L Values. We’ll create a corresponding Target measure for all metrics for which we have target data:

   `P&L Target =`

   `var res = SWITCH(TRUE( ),`

   `MAX('P&L Rows'[Order])=7,  [NS $ Target]/1000000,`

   `MAX('P&L Rows'[Order])=12,  [GM $ Target]/1000000,`

   `MAX('P&L Rows'[Order])=13,  [GM % Target]*100,`

   `MAX('P&L Rows'[Order])=16,  [Net Profit $ Target]/1000000,`

   `MAX('P&L Rows'[Order])=17,  [Net Profit % Target]*100)`

   `RETURN IF(HASONEVALUE('P&L Rows'[Description]), res, [NS $ Target]/1000000)`
2. Now we’ll create the P&L BM measure: `P&L BM = SWITCH(TRUE( ), SELECTEDVALUE('Set BM'[ID]) = 1, [P&L LY], SELECTEDVALUE('Set BM'[ID]) = 2, [P&L Target])`
3. We’ll update the P&L Final Value measure that is being used in the visual with column header based on BM instead of LY. 

   `P&L Final Value = SWITCH(TRUE( ),`

   `SELECTEDVALUE(fiscal_year[fy_desc])=MAX('P&L Columns'[Col Header]), [P&L Values],`

   `MAX('P&L Columns'[Col Header]) = "BM", [P&L BM],`

   `MAX('P&L Columns'[Col Header]) = "Chg",[P&L YoY Chg],`

   `MAX('P&L Columns'[Col Header]) = "Chg %",[P&L YoY Chg %])`
4. We’ll also update the P&L Columns table since the headers for P&L Final Value is coming from there.

   Measure `P&L Columns =`

   `var fy = ALLNOBLANKROW(fiscal_year[fy_desc])`

   `RETURN UNION(ROW("Col Header", "BM"), ROW("Col Header", "Chg"), ROW("Col Header", "Chg %"), fy)`
5. Also update the P&L YoY Chg and P&L YoY Chg % measures used in P&L Final value to refelect the new BM measure and update the name to not show YoY. Also both the measures as BLANK if the      P&L Values or P&L BM measure is BLANK. 

   A. `P&L Chg =`

      `var res = [P&L Values] - [P&L BM]`

      `RETURN IF(ISBLANK([P&L Values]) || ISBLANK([P&L BM]), BLANK( ), res)`

   B. `P&L Chg % =`

      `var res = DIVIDE([P&L Chg], ABS([P&L BM]), 0) * 100`

      `RETURN IF(ISBLANK([P&L Values]) || ISBLANK([P&L BM]), BLANK(), res)`

Net Sales Performance visual:

1. Replace the P&L LY measure in values field by P&L BM measure. Change the label to “vs BM”.

### Step 12: Setup Dynamic GM% Parameter Slicer

1. Create Numeric range parameter “Target Gap Tolerance” with values from 0 (0%) to 0.21 (20%) in increments of 0.01 (1%). Add the slicer to page.
2. Auto generated Target Gap Tolerance paramter DAX code:

   `Target gap Tolerance = GENERATESERIES(0, 0.21, 0.01)` → Change to Percentage data type with 0 decimals.

   `Target gap Tolerance Value = SELECTEDVALUE('Target gap Tolerance'[Target gap Tolerance], 0)`
3. Create a new measure for GM% variance between benchmark and actual value:  `GM % Variance = [GM % BM] - [GM %]`
4. Now we’ll create a measure to help us filter when the GM % Variance is more than the Target gap Tolerance Value. We’ll set the values as 1 if TRUE or 0 if FALSE.

   // `GM % Filter = IF([GM % Variance] >= SELECTEDVALUE('Target gap Tolerance'[Target gap Tolerance]), 1, 0)`

   `GM % Filter = IF([GM % Variance] >= [Target gap Tolerance Value], 1, 0)`
5. Apply GM % Filter measure as a visual level filter for the GM %, NS $ and GM $ Performance Plot visual by adding the measure and show items when value is 1.
6. Now the Performance Plot will only show customer with higher GM % Variance than the Slicer value so that stakeholders can eventually fix underlying finance issue and move them to the top right quadrant with high NS $ and GM % values.

### Step 13: Configure Toggle Button to switch GM % and NP % in Marketing View Performance Plot visual

1. Add a rounded rectangle button to the Marketing View Performance Plot visual and add text Show NP %.
2. Duplicate the Plot and change the Y Axis GM % field to Net Profit % and Size GM $ field to Net Profit $. Duplicate the button and change text to Show GM %.
3. Align the Plots and Buttons on top of each other. Assign lables and create Plot and Button pair groups in selection tool.
4. Create  Bookmarks - `GM % Shown` and `NP % Shown`. Unselect Data option from settings so that the static data when bookmark was created is not shown and instead the data gets updated dynamically.
5. Now hide/unhide the groups and Update the Bookmarks.
6. Configure Button Action to display corresponding Bookmarks. `Show NP % Button → NP % Shown Bookmark` & `Show GM % Button → GM % Shown Bookmark`

### Step 14: Implement custom Tooltip to show NS $ and GM % Trend in Sales View

1. Create a new page Sales Trend Tooltip and set the Page Type and Canvas Settings Type as Tooltip.
2. Add a Line chart visual with NS $ measure on the Y Axis, GM % measure on the secondary Y Axis and months on the X Axis.
3. Disable X and Y Axis Titles. Set X Axis type as Categorical. Add Data Markers.
4.  Since we see a scroll bar on the Tooltip we can change the size to custom Height 300px and Width 400px.
5. Configure the tooltip for Customer Performance & Perfromance Plot visuals since they involve customers from Visual Properties → Tooltips and hide the tooltip page from the report.
6. Now we’ll create a custom title to reflect the customer for whom the tooltip is being shown. `Sales Trend Tooltip Title = "NS $ & GM % Trend for " & SELECTEDVALUE(dim_customer[customer])`

   Configure this custom title for the tooltip using conditional title option.

### Step 15: Fix Data Quality Issues

- In dim_customer, replace AltiQ Exclusive → AtliQ Exclusive and change its channel Retail → Direct using M Code: `= Table.ReplaceValue(#"Replaced to AtliQ Exclusive India Value", each [channel], each if [customer_code]="90002011" then "Direct" else [channel], Replacer.ReplaceText,{"channel"})`
- In dim_customer, we we 2 entries in customer for Amazon due to trailing space to deal with this select the column and use Trim from Transform Tab.
- In Measures List, since we have a lot of them we can group similar ones by selecting → Right click → Display Folder. Create Folders: Abs Error, Forecast, GM, Net Error, Net Profit, NS, P&L.

### Step 16: Create Support View

- Create 5 new buttons to address the following support issues:
1. Get an Issue resolved: Action: Navigate to Web URL & Tooltip: Navigate to the BI 360 Report - Ticket Management System to log a new ticket.
2. Provide feedback: Action: Navigate to Web URL & Tooltip: Navigate to the BI 360 Report - Feedback Form to provide your valuable feedback.
3. Add New requests: Action: Navigate to Web URL & Tooltip: Navigate to the BI 360 Report - Change Request Tool to commission new requests.
4. Check out Contingency Plan: Action: Navigate to Web URL & Tooltip: Navigate to the BI 360 - Contingency Plan Documentation to review issue specific POCs.
5. Power BI Documentation: Action: Navigate to Web URL & Tooltip: Learn how to consume insights better from Power BI.
- Setup Nav Bar navigation from Home view to Support view and back.

### Step 17: Create Info View

- Create a Text Box with following info:
1. All the system data in tool is refreshed every month on 5th working day.
2. System data such as Forecast, Actuals and Historical forecast are received from Global database.
3. Non system data such as Target, Operational Expense and Market Share are refreshed on request.
4. For FAQs click here.
5. Download live excel version here.
- Setup Nav Bar navigation from Home view to Info view and back.

### Step 18: Save DAX Studio Metrics File

- Power BI → External Tools → DAX Studio → Advanced → View Metrics → You can see metrics and statistics related to your report and data.
- Save Metrics → As CSV → Can be reloaded to DAX Studio later for analysis when needed.

---

## Phase 9: Executive View

### Step 1: Importing Market Share Data

- Import the file marketshare-v2022.xlsx into the Power BI Backend i.e Power Query.
- Market Share % will be the share of one of the 6 top players out of the total market share. Instead of calculating this manually as 6 individual columns:
- We can select all the 6 players and unpivot the columns to merge them all as 6 new individual rows.
- Change the new attribute column name to manufacturer and the values column name to sales_$.
- Remove the additional sales_$ from all the rows in manufacturer column using Extract before delimiter tool.
- Change fy_dec column name to fy and set data type as Text. Load the data to PBI Frontend.

Designing Market Share View: Create new Market Share View and hide it from the report.

### Step 2: Configure the Data Model

- sub_zone is a column in both market_share and dim_market however it is a many to many relationship. To tackle this we’ll create a new table that will contain unique values of sub_zone that can be used as a pathway to connect both the tables. We’ll do the same for category field as well since directly connecting it leads to many to many relationship.
- Report View → Modelling → New table →  DAX Code: `sub_zone = ALLNOBLANKROW(dim_market[sub_zone])`
- Report View → Modelling → New table →  DAX Code: `category = ALLNOBLANKROW(dim_product[category])`

Table Relationships to be setup:

|Primary Key (Dimension table) (1)||Secondary Key (Fact table) (*)|
|-|-|-|
|fiscal_year (fiscal_year)|→|fy (market_share)|
|sub_zone (sub_zone)|→|sub_zone (dim_market)|
|sub_zone (sub_zone)|→|sub_zone (market_share)|
|category (category)|→|category (dim_product)|
|category (category)|→|category (market_share)|

### Step 3: Building Market Share View Visuals

- Create a Matrix visual with category field as Rows, manufacturer as Columns and Sum of sales_$ as Values field.
- Create a new measure to calculate the Market Share %. Format as Percentage value with 1 decimal.

  `Market Share % = DIVIDE(SUM(market_share[sales_$]), SUM(market_share[total_market_sales_$]), 0)`
- Replace the Matrix visual Values field with Market Share % measure. Add fy_desc to the Columns field before manufacturer. Add sub_zone field as a Tile slicer to enable further drilling.
- Now that we have all the data aligned convert the matrix visual into a ribbon visual with fy_desc field as X Axis, Market Share % as Y Axis and manufacturer field as the legend.
- Enable data lables for the Ribbon visual. Sort Axis the visual by fy_desc in ascending order.
- Also since Other Manufacturers takes up a lot of space and since it is not very helpful for comparision we can remove it by adding a visual level filter to unselect it from the manufacturers values.

### Step 4: Creating the Executive View

- Duplicate the Supply Chain view and remove all the visuals except the Slicers and Nav Bar. Add the BM Slicer from the Sales View containing comparision with LY or target.
- Update the Nav Bar Icons to display the Executive Icon as selected instead of Supply Chain.

### Step 5: Creating Executive KPI Cards

- Create a KPI Card Visual with NS $ measure as the Value field, fiscal_year as the Trend Axis field and NS $ BM Measure as the Target field. Set the Title as Net Sales and Target Label as BM.
- Copy the above KPI Card and edit to GM % measure as the Value field and GM % BM measure as the Target field. Set the Title as GM %.
- Copy the above KPI Card and edit to Net Profit % measure as the Value field and Net Profit % BM measure as the Target field. Set the Title as Net Profit %.
- Copy the above KPI Card and edit to Forecast Accuracy % measure as the Value field and Forecast Accuracy % LY measure as the Target field. Set the Title as Forecast Accuracy.

### Step 6: Creating Revenue (NS) by Division & Channel Donut Charts

- Add a Donut chart with NS $ Measure as the Vlaues field and divison as legend field. Align Legend to Top center. Change title to Revenue by Division. Set Detail label contents to only show Percentage values.
- Duplicate the above chart and replace the divison by channel as legend field. Change title to Revenue by Channel.

### Step 7: Creating Key Insights by Sub Zone Matrix visual

- Create a Matrix visual with sub_zone as Rows and NS $, GM %, Net Profit %, Net Error %, Risk as Columns.
- Apply formatting to display values in Millions and with 1 decimal. Add Title as Key Insights by Sub Zone.
- We need AtliQ’s Market Share as a Column. For this we’ll create a new measure to calculate the Market Share % measure with a filter for AtliQ.

  `AtliQ MS % = CALCULATE([Market Share %], market_share[manufacturer] = "atliq")`
- For adding Revenue (Net Sales) Contribution as a column we’ll need to create a new measure that will calculate the Percentage of NS foir that sub_zone across the total of all sub_zone.

  `RC % = DIVIDE([NS $], CALCULATE([NS $], ALL(sub_zone[sub_zone])), 0)`
- Add Footer as: BM: Benchmark | LY: Last Year | MS: Market Share | RC %: Revenue Contribution % | EI: Excess Inventory | OOS: Out of Stock
- We need Conditional Formatting for GM % column whenever there is decline compared to BM. For this we’ll use the GM % Variance measure.

  Visual → Cell Elements → Based on GM % Variance → Icons → If value > 0 and ≤ 100 then Red Down Arrow, If value = 0 then Yellow Side Arrow, If value ≥ Min and < 0 then Green Up Arrow.

### Step 8: Creating Yearly Trend Chart

- Create a Line and Cluster Column Chart with fy_desc field on X Axis, NS $ field on Y Axis Column and GM %, Net Profit % & AtliQ MS % on Y Axis Line.
- At this moment the chart will only show the data selected in the FY Slicer but we want the data to be shown for all the years. So we’ll change the FY Slicer Interaction to the Line Chart as None to avoid filtering.
- Add Title as Yearly Trend by Revenue, GM %, Net Profit % and PC Market Share %.

### Step 9: Creating Market Share Ribbon Chart Visual

- Move the Market Share % by Manufacturer Ribbon Chart visual that was created in the Market Share View to the Executive View.
- At this moment the chart will only show the data selected in the FY Slicer but we want the data to be shown for all the years. So we’ll change the FY Slicer Interaction to the Ribbon Chart as None to avoid filtering.

### Step 10: Creating Top 5 Customers & Products by Revenue Visuals

- Create a matrix visual with customers in Rows field and RC % and GM % in Values field.
- The RC % column throws 100 % values in all Rows since in its formula we have used ALL to ignore only sub_zone filter. We’ll extend this to ignore all filters from market, customer or product.

  `Updated RC % Measure: RC % = DIVIDE([NS $], CALCULATE([NS $], ALL(dim_market), ALL(dim_customer), ALL(dim_product)), 0)`
- Since we only need the Top 5 Customers we’ll configure a Top N visual filter to display top 5 customers by Revenue i.e NS $. Set Tile to Top 5 Customers by Revenue.
- Add Conditional Formatting on GM % Column whenever there is decline compared to BM. For this we’ll use the GM % Variance measure.

  Visual → Cell Elements → Based on GM % Variance → Icons → If value > 0 and ≤ 100 then Red Down Arrow, If value = 0 then Yellow Side Arrow, If value ≥ Min and < 0 then Green Up Arrow.
- Duplicate the above visual and replace customer in Rows field by products. Reconfigure the Top N visual filter to display top 5 products by Revenue i.e NS $. Set Tile to Top 5 Products by Revenue.

---
