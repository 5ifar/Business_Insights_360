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
2. Data view → Enter Data → Paste → Rename as P & L Rows.

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

`Step 5: Configuring Quarters & YTD/YTG Slicers`

AtliQ’s Financial Year starts is from Sep to Aug.

1. We’ll create a new column in the dim_date table which will contain the fiscal month number obtained by adding 4 months to the dates and extracting the month part of it.

   DAX Calculated Column: fy_month_num = MONTH(DATE(YEAR(dim_date[date]), MONTH(dim_date[date])+4, 1))
2. To calculate the quarters we need to divide the fiscal month nbr by 3 and round it up to the next integer. 
   
   DAX Calculated Column: quarters = “Q” & ROUNDUP(dim_date[fy_month_num]/3, 0)

   Add this Calculated Column as a Tile Slicer to the Finance View.
3. Now Sep 1st to the last sales date will be YTD, and the rest of the days until 31st August should be YTG. 
   Hence, the logic for generating YTD and YTG is when a date in the date column is greater than the last sales then it is YTG, otherwise it is YTD.
4. However, if you compare the date column with the last sales date you will get the YTD and YTG only for the current fiscal year which is 2022. Since you need it for all fiscal years – a column that works for all fiscal years is needed i.e. a column that is not attributed to the year but just the months. You can see that it has the fiscal year month number which is the same for all fiscal years. i.e fy_month_num for Dec 21 and Dec 20 is the same as it takes the month into the account and not the year. So, we will compare the fy_month_num of the date column vs fy_month_num of the last sales date to find if a given date is YTD or YTG.
5. Now we need to use the if condition to check whether the fy_month_num of a date is greater than FYMONTHNUM of last sales date in order to assign it as YTG or else YTD.

   DAX Calculated Column: ytd_ytg = 
   
   var LASTSALESDATE = MAX(fact_sales_monthly[date])
   
   var FYMONTHNUM = MONTH(DATE(YEAR(LASTSALESDATE), MONTH(LASTSALESDATE)+4, 1))
   
   RETURN IF(dim_date[fy_month_num] > FYMONTHNUM, "YTG", "YTD")
   
   Add this Calculated Column as a Tile Slicer to the Finance View.

`Step 6: Building P&L Performance over Time visual`

1. Add a clustered column chart visual with P&L Values Measure on the Y Axis and date on the X Axis. Format the X Axis type as Categorical to see distinct columns for all months.
2. Format the date as mmm yy format to show shortened dates in the visual.
3. Change the report setting to select Change default visual interaction from cross highlighting to cross filtering to avoid highlighting when filtering based on a single line item of P&L       Values.
4. Currently if we dont select any line item it shows a total of all P&L Values by default which is not very useful. We want the default to be Net Sales, for this we need to edit the P&L Values measure and add a condition to ensure Net Sales is selected when no line item is selected. Updated P&L Values Measure: P&L Values = 
   
   var res = SWITCH(TRUE( ),

   MAX('P&L Rows'[Order])=1,  [GS $]/1000000,

   MAX('P&L Rows'[Order])=2,  [Pre-Invoice Deduction $]/1000000,

   MAX('P&L Rows'[Order])=3,  [NIS $]/1000000,

   MAX('P&L Rows'[Order])=4,  [Post-Invoice Deduction $]/1000000,

   MAX('P&L Rows'[Order])=5,  [Post-Invoice Other Deduction $]/1000000,

   MAX('P&L Rows'[Order])=6,  [Total Post-Invoice Deduction $]/1000000,

   MAX('P&L Rows'[Order])=7,  [NS $]/1000000,

   MAX('P&L Rows'[Order])=8,  [Manufacturing Cost $]/1000000,

   MAX('P&L Rows'[Order])=9,  [Freight Cost $]/1000000,

   MAX('P&L Rows'[Order])=10,  [Other Cost $]/1000000,

   MAX('P&L Rows'[Order])=11,  [COGS $]/1000000,

   MAX('P&L Rows'[Order])=12,  [GM $]/1000000,

   MAX('P&L Rows'[Order])=13,  [GM %]*100,

   MAX('P&L Rows'[Order])=14,  [GM / Unit])

   RETURN IF(HASONEVALUE('P&L Rows'[Description]), res, [NS $]/1000000)
5. Now the visual shows the data for the line item selected but the title does not get updated automatically. For this we need to create another measure which will provide title as per          selection that is made out of the line items.

   DAX Measure: P&L Selected Row = IF(HASONEVALUE('P&L Rows'[Description]), SELECTEDVALUE('P&L Rows'[Description]), "Net Sales")

   We’ll then create a supporting Measure to customize the title for specific visual: Performance Visual Title = [P&L Selected Row] & " Performance over Time”
6. Now we’ll update this Performance Visual Title measure as the parameter to base the Title on using the Title fx Conditional Formatting.
7. We also need t compare performance figures from last year so we’ll add the P&L LY Measure before the P&L Values measure on Y Axis.
8. Since our objective behind this visual was to compare the performance over time, using an Area Chart here would be better since its good for comparing values over time as well as total value using the plot area.
9. Formatting for Area chart: Move Legend position to Top Center. Remove X  & Y Axis Titles.

`Step 7: Building Top Market & Product visuals`

1. Add a Matrix visual with market & customer fields as Rows and P&L Values & P&L YoY Chg % Measures as Values.
2. Formatting for Matrix visual: Disable Row & Column Subtotals. 
3. Add a Matrix visual with segment, category & product fields as Rows and P&L Values & P&L YoY Chg % Measures as Values.
4. Formatting for Matrix visual: Disable Row & Column Subtotals.

`Step 8: Importing Operating Expenses Data`

- Get Data → Excel Workbook → Select the Operational Expenses Excel File → Transform → Rename to operational_expense → Any transformation if required → Load
- Data Modelling:

|Primary Key (Dimension table) (1)||Secondary Key (Fact table) (*)|
|-|-|-|
|fiscal_year (fiscal_year)|→|fiscal_year (operational_expense)|
|market (dim_market)|→|market (operational_expense)|

`Step 9: Calculated Columns & Measures for Operational Expenses & Net Profit Calculations`

1. We’ll calculate the Ads & Promotions Cost on the Net Sales value. Set the data type as Currency.
   
   DAX Calculated Column: ads_promotions = var pct = CALCULATE(MAX(operational_expense[ads_promotions_pct]), RELATEDTABLE(operational_expense))
   
   return pct*'fact_actuals&estimates'[net_sales]
2. We’ll calculate the Other Operational Expenses on the Net Sales value. Set the data type as Currency.
   
   DAX Calculated Column: other_operational_expense = var pct = CALCULATE(MAX(operational_expense[other_operational_expense_pct]), RELATEDTABLE(operational_expense))
   
   return pct*'fact_actuals&estimates'[net_sales]
3. Ads & Promotions Expense Measure: Ads & Promotions $ = SUM('fact_actuals&estimates'[ads_promotions])
4. Other Operational Expense Measure: Other Operational Expense $ = SUM('fact_actuals&estimates'[other_operational_expense])
5. Operational Expenses Measure: Operational Expense $ = ([Ads & Promotions $ ] + [Other Operational Expense $ ]) * -1

   We’ll multiply with -1 to ensure this is treated as unfavorable expense.
6. Net Profit Measure: Net Profit $ = [GM $ ] + [Operational Expense $ ]

   Here we want to subtract the Operational Expense but since it already has negative value we can add it.
7. Net Profit % Measure: Net Profit % = DIVIDE([Net Profit $ ], [NS $ ], 0)

`Step 10: Updating the P&L visual with Operating Expenses and Net Profit`

1. Since we want to add the 3 fields to the P&L Line items, they need to be appended to the P&L Rows tables that is being used in visual rows field.
2. In Transform Power Query, go to P&L Rows and in the Source step edit the table to add 3 new rows: Operational Expense (Order: 15), Net Profit (Order: 16) & Net Profit % (Order: 17).
3. Now we edit the P&L Values Measure to include these 3 new fields. Updated Measure: P&L Values =

   var res = SWITCH(TRUE( ),

   MAX('P&L Rows'[Order])=1,  [GS $]/1000000,

   MAX('P&L Rows'[Order])=2,  [Pre-Invoice Deduction $]/1000000,

   MAX('P&L Rows'[Order])=3,  [NIS $]/1000000,

   MAX('P&L Rows'[Order])=4,  [Post-Invoice Deduction $]/1000000,

   MAX('P&L Rows'[Order])=5,  [Post-Invoice Other Deduction $]/1000000,

   MAX('P&L Rows'[Order])=6,  [Total Post-Invoice Deduction $]/1000000,

   MAX('P&L Rows'[Order])=7,  [NS $]/1000000,

   MAX('P&L Rows'[Order])=8,  [Manufacturing Cost $]/1000000,

   MAX('P&L Rows'[Order])=9,  [Freight Cost $]/1000000,

   MAX('P&L Rows'[Order])=10,  [Other Cost $]/1000000,

   MAX('P&L Rows'[Order])=11,  [COGS $]/1000000,

   MAX('P&L Rows'[Order])=12,  [GM $]/1000000,

   MAX('P&L Rows'[Order])=13,  [GM %]*100,

   MAX('P&L Rows'[Order])=14,  [GM / Unit],

   MAX('P&L Rows'[Order])=15,  [Operational Expense $]/1000000,

   MAX('P&L Rows'[Order])=16,  [Net Profit $]/1000000,

   MAX('P&L Rows'[Order])=17,  [Net Profit %]*100)

   RETURN

   IF(HASONEVALUE('P&L Rows'[Description]), res, [NS $]/1000000)

---

## Phase 5: Sales View

`Step 1: Building Customer Performance visual`

1. Add a Matrix visual with customer field as Rows and Net Sales, Gross Margin & Gross Margin % Measures as values.
2. Change the display units for NS $ & GM $ to Millions. Format → Specific Column → Select Column → Value Display Unit to Millions with 2 decimal points.

`Step 2: Building Customers GM & NS Plot visual`

1. Copy the Top Customers visual and convert it to Scatter Plot. Add market field before customer field in values to aid drill down and fix congestion. Enable Category labels and Zoom Sliders.
2. We need Net Sales values as the X Axis and the Gross Margin % values as the Y Axis and the Gross Margin $ values as the Bubble size. Add region field as the Legend.
3. Copy the FY, Quarters & YTD-YTG Slicers from Finance View to Sales View and Sync them across pages.
4. Add an additional Region, Market & Customer fields Dropdown Slicers for country-specific business users.

`Step 3: Building Product Performance visual`

1. Copy the Customer Performance visual and replace the customer field in Rows by segment, category & product fields to aid drill down on product level.

`Step 4: Building Unit Economics visual`

1. Add a Donut chart with P&L Values measure as the Values and Description field as Legend. Now in the Filters pane, filter the Description field to only show Net Sales, Pre Invoice Deduction & Total Post Invoice Deduction.
2. Set Detail Labels Display Units as None. Align the legend to top center. Disable legend title.
3. Copy the above visual and change Description field filter to only show Total COGS & Gross Margin. This is a breakdown of Net Sales from above visual.

---

## Phase 6: Marketing View

Duplicate the Sales View. Remove the Customer Performance Matrix visual.

`Step 1: Building Product Performance visual`

- Add Net Profit and Net Profit % values to the Product Performance visual copied from Sales View.

`Step 2: Building Products GM & NS Plot visual`

- Replace the existing values field by segment, category and product fields. Add division field to the legend.

`Step 3: Building Unit Economics visual`

- Replace the Net Sales and Deduction Donut chart by a Waterfall chart with Gross Margin, Operation Cost & Net Profit fields to show the flow of expenses.
- Sort the waterfall by P&L Rows Order column such that the Gross Margin is on the left and Net Profit is on right.

`Step 4: Building Market Performance visual`

- Copy the Product Performance visual and change the Rows to region and market fields.

---

## Phase 7: Supply Chain View

`Step 1: Understanding Key Metrics`

- We already have a quantity measure but it is not the actual sales quantity because it has been calculated based on the fact_actuals&estimates that takes into consideration the estimate/forecast data as well. We only need the actual data sales qty.
- To extract the actual sales qty from the fact_actuals&estimates table we need to ignore the data after the last sales date. To get this date we’ll enable the load of the last_sales_month table from Power Query.
- Now since we have both fact_actuals&estimates & last_sales_month tables we don't really need the fact_sales_monthly table and can disable its load to reduce the file size. But first we need to replace its usage from the ytd_ytg calculated column. So we’ll update to: var LASTSALESDATE = MAX(last_sales_month[last_sales_month]) in the DAX code.
- Now we can disable the fact_sales_monthly table loading from Power Query.
- We’ll create a new measure to calculate the actual sales quantity for dates before the last_sales_month date.

  Sales Qty Measure: Sales Qty = CALCULATE([Quantity], 'fact_actuals&estimates'[date] <= MAX(last_sales_month[last_sales_month]))
- Similarly we create a measure to calculate the forecast sales quantity without using an filters based on entire fact_forecast_monthly table. 

  Forecast Qty Measure: Forecast Qty = SUM(fact_forecast_monthly[forecast_quantity])

`Step 2: Creating Supply Chain Measures`

- Net Error is the difference between the Forecast and Actual Sales figures. Net Error Measure: Net Error = [Forecast Qty] - [Sales Qty]
- Net Error % is calculated over the Forecast figure. Net Error % Measure: Net Error % = DIVIDE([Net Error], [Forecast Qty], 0)
- We’ll modify Forecast Qty Measure to restrict it till last sales date since we want to compare it with the Actual Sales.

  Updated Forecast Qty Measure: Forecast Qty = 

  var lsalesdate = MAX(last_sales_month[last_sales_month])

  RETURN CALCULATE(SUM(fact_forecast_monthly[forecast_quantity]), fact_forecast_monthly[date] <= lsalesdate)
- For the Absolute Error Measure, in AtliQ as per the supply chain team’s requirement - the ABS Error needs to be measured for each product at the monthly level.  In other words, we need to convert the Net error to ABS Error at the product and month level granularity.

  To apply the formula to Products and Months, we need a list of products and Months. Hence DISTINCT(dim_product[product_code]) and DISTINCT(dim_date[month]) are used.

  Now, you need to iterate the ABS([Net Error]) for each product and sum them up. Hence, SUMX is required.
- Absolute Error Measure: Abs Error = SUMX(DISTINCT(dim_date[date]), SUMX(DISTINCT(dim_product[product_code]), ABS([Net Error])))
- Absolute Error % is also calculated over the Forecast figure.Absolute Error Measure: Abs Error % = DIVIDE([Abs Error], [Forecast Qty], 0)
- Forecast Accuracy is logical opposite of Absolute Error. However doing simply, Forecast Accuracy % = 1 - [Abs Error %] gives us some blank rows in visualizations and the Forecast Accuracy % as 1 or 100%. This happens due to the fact that products without sales or forecast is included in this calculation. Because for products without forecast or sales, the ABS Error % is blank. By that logic, Forecast Accuracy = 1- Blank( ) which returns 1. Hence we’ll add an IF condition such that if the [ABS Error %] is blank the formula will also return blank and hence it won’t be displayed in the table.

  Updated Forecast Accuracy % Measure: Forecast Accuracy % = IF([Abs Error %] <> BLANK( ), 1 - [Abs Error %], BLANK( ))

`Step 3: Building Supply Chain visuals`

1. Add Forecast Accuracy %, Net Error & Abs Error KPI Cards to the top left Supply Chain View page area.
2. Copy the Customer Performance visual from Sales View. Replace the values fields by Forecast Accuracy %, Net Error and Net Error %.
3. Copy the Product Performance visual from Sales View. Replace the values fields by Forecast Accuracy %, Net Error and Net Error %.
4. Add a Line & Clustered Column chart with Months field on X Axis (Set Value type as Categorical), Net Error on Column Y Axis and Forecast Accuracy % on Line Y Axis.

   We also need the Forecast Accuracy Year Ago % value on the Line Y Axis, for this we need to create a new measure. Forecast Accuracy % LY Measure:

   Forecast Accuracy % LY = CALCULATE([Forecast Accuracy %], SAMEPERIODLASTYEAR(dim_date[date]))

   Add this measure to both the Customer and Product Performance visuals.
5. We’ll create a new Risk measure to display if the item is in excess inventory or out of stock based on the Net Error measure value. Risk Measure:

   Risk = IF([Net Error]>0, "EI", IF([Net Error]<0, "OOS", BLANK()))

   Add this Risk Measure to both the Customer and Product Performance visuals.
6. Add Footer as: BM: Benchmark | LY: Last Year | EI: Excess Inventory | OOS: Out of Stock

---

## Phase 8: Designing Effective Dashboard
