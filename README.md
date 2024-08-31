# <img src="https://miro.medium.com/v2/resize:fit:1400/1*8bUjUiCWk0VhS8-lgAj0Og.png" width="4%" height="4%"> Business Insights 360

This repository serves as my documentation for the AtliQ Hardwares Business Insights 360 - Power BI Project.
It was created as a self-learning project for the course: [Get Job Ready: Power BI Data Analytics for All Levels 2.0](https://codebasics.io/courses/power-bi-data-analysis-with-end-to-end-project) by [Codebasics](https://codebasics.io/).

The entire project has been implemented using Microsoft Power BI Desktop 2.128.751.0 and published on Microsoft Power BI Service.

The project data files have not been uploaded to this repository in compliance with Codebasics Data & Content Distribution Policy.

---

## Contents:
Please find the sectional links for the project below:
- [BI 360 Live Dashboard Link](https://app.powerbi.com/view?r=eyJrIjoiM2UwZTJjOWQtYjA0MC00ZDVlLThiYTMtZGY4NDc1YzRkYTZiIiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)
- [Introduction to AtliQ Hardware](#introduction-to-atliq-hardware)
- [Project Objective](#project-objective)
- [Tools used & Methodologies implemented](#tools-used)
- [About the Dataset](#about-the-dataset)
  - [Data Sources](#data-sources)
  - [Data Integrity](#data-integrity)
- [Data Model](#data-model)
- [Project Implementation](#project-implementation)
- [BI 360 Dashboard Overview](#bi-360-dashboard-overview)
- [Conclusion](#conclusion)

---

## [Business Insights 360 Live Dashboard Link](https://app.powerbi.com/view?r=eyJrIjoiM2UwZTJjOWQtYjA0MC00ZDVlLThiYTMtZGY4NDc1YzRkYTZiIiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)

---

## Introduction to AtliQ Hardware:
**Domain:** Consumer Goods | **Functions:** Finance, Sales, Marketing, Supply Chain and Executive

- AtliQ Hardwares is company that sells computer hardware and peripherals like PC, mouse, printer etc. to clients across the world.
- They have a major B2B business model wherein they sell to stores like Croma, Best Buy, Staples, Flipkart etc. who then sell it to the end users (consumers). These stores are their main customers.
- They sell through 3 channels: Retailer, Direct and Distributor.
- AtliQ Hardwares’s Customers are of two types. Both these Platforms are called Retailer channels.
  1. Brick & Mortar Customer: Actual physical stores e.g. Croma, Best Buy
  2. E-commerce Customer: Online websites E.g. Amazon, Flipkart
- AtliQ Hardwares also has a minor B2C business model wherein they own stores: AtliQ E-store and AtliQ Exclusive. These are called Direct channels.
- They also have Distributors in some countries with restricted trade. E.g. Neptune

## Project Objective:
AltiQ Hardware, a global leader in computers and accessories, faced unexpected losses after opening a store in America. These setbacks were identified to be caused due to reliance on outdated methods such as Excel for data analysis. To address this issue, the company's leadership recognized the need for a transformative approach to leveraging data for informed decision-making. With competitors boasting robust analytics teams, AltiQ Hardware recognizes the urgent need to develop its analytics capabilities using Power BI to thrive in the industry.

To outshine competitors, they've adopted Power BI for analytics with 1.8 million transaction records from Excel, CSV, and MySQL. The Power BI Dashboard includes:
- Home Page: Central navigation for Dashboard.
- Finance: Enhances financial planning.
- Sales: Boosts revenue and market share.
- Marketing: Elevates brand visibility.
- Supply Chain: Optimizes inventory management.
- Executive: Provides top management overview.

## Tools used:
1. Microsoft Power BI: for Data ETL, Data Modelling, Data Visualization & Dashboarding
2. GitHub - for Documentation

## Skills & Methodologies implemented:
1. Data Cleaning: **Power Query**
2. Data Manipulation: **DAX Measures & Columns, Numeric & Field Parameters**
3. Data Modelling
4. Data Visualization: **Conditional Formatting, Custom Tooltip**
5. Dashboarding: **Filters, Custom Icon Buttons, Slicers, Bookmarks, Page Navigation**
6. Report Publishing: **PBI Service and Report Optimization**
7. Documentation

---

## About the Dataset:

### Data Sources:
The dataset contains 11 tables in total, namely:
- From gdb041 MySQL Server:
  - dim_customer: 209 records | 5 columns
  - dim_market: 27 records | 3 columns
  - dim_product: 397 records | 6 columns
  - fact_forecast_monthly: 1,885,941 records | 4 columns
  - fact_sales_monthly: 1,885,941 records | 4 columns
- From gdb041 MySQL Server:
  - freight_cost: 135 records | 4 columns
  - manufacturing_cost: 1,197 records | 3 columns
  - post_invoice_deductions: 2,063,076 records | 5 columns
- Excel Files:
  - market_share: 737 records | 6 columns
  - operational_expense: 113 records | 4 columns
  - ns_gm_target: 321 records | 5 columns

## Data Integrity:
ROCCC Evaluation:
- Reliability: MED - The raw dataset is created and updated by Codebasics. It has total 9 files. All of them were utilized in the analysis.
- Originality: HIGH - First party provider (Codebasics)
- Comprehensiveness: HIGH - Total 11 Files with a total of around 5.8 Million records were provided. Dataset contains multiple dimension parameters for Customers & Products as well as comprehensive sales transaction data.
- Current: MED - Dataset was updated upto FY 2022 i.e almost 2 years old. So its not very relevant. Any trends observed and insights gained need to be comprehended as a general (not FY-specific) trend.
- Citation: HIGH - Official citation/reference available.

---

## Data Model:
<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Data%20Model/Data%20Model%20Final.PNG" width="100%" height="100%"> </div>

---

## Project Implementation:
Please find the documentation links for the project phase-wise implementation below:
- [Phase 1: Data Wrangling](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-1-data-wrangling)
  - [Loading Data to MySQL Workbench](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-loading-data-to-mysql-workbench)
  - [Connecting MySQL Database to Power BI](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-connecting-mysql-database-to-power-bi)
- [Phase 2: ETL with Power Query](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-2-etl-with-power-query)
  - [Creating custom Date Table](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-creating-custom-date-table)
  - [Creating Last Sales Month Reference Table](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-creating-last-sales-month-reference-table)
  - [Creating Remaining Forecast Reference Table](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-creating-remaining-forecast-reference-table)
  - [Creating a New Table with both Actual & Forecast Data](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-creating-a-new-table-with-both-actual--forecast-data)
  - [Calculating Net Invoice Sales based on FY varying Gross Price & Pre-invoice Deductions](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-5-calculating-net-invoice-sales-based-on-fy-varying-gross-price--pre-invoice-deductions)
- [Phase 3: Data Modelling & Calculated Columns](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-3-data-modelling--calculated-columns)
  - [Normalizing Data in Tables](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-normalizing-data-in-tables)
  - [Creating Table Relationships](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-creating-table-relationships)
  - [Creating fiscal_year table using DAX](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-creating-fiscal_year-table-using-dax)
  - [Calculated Columns for post_invoice Calculations](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-calculated-columns-for-post_invoice-calculations)
  - [Calculated Columns for COGS & Gross Margin Calculations](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-5-calculated-columns-for-cogs--gross-margin-calculations)
  - [Optimizing Report File Size](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-6-optimizing-report-file-size)
- [Phase 4: Finance View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-4-finance-view)
  - [Creating Measures Table](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-creating-measures-table)
  - [Creating P & L Measures](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-creating-p--l-measures)
  - [Creating P & L Rows Table](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-creating-p--l-rows-table)
  - [Building P&L Matrix visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-building-pl-matrix-visual)
  - [Configuring Quarters & YTD/YTG Slicers](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-5-configuring-quarters--ytdytg-slicers)
  - [Building P&L Performance over Time visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-6-building-pl-performance-over-time-visual)
  - [Building Top Market & Product visuals](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-7-building-top-market--product-visuals)
  - [Importing Operating Expenses Data](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-8-importing-operating-expenses-data)
  - [Calculated Columns & Measures for Operational Expenses & Net Profit Calculations](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-9-calculated-columns--measures-for-operational-expenses--net-profit-calculations)
  - [Updating the P&L visual with Operating Expenses and Net Profit](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-10-updating-the-pl-visual-with-operating-expenses-and-net-profit)
- [Phase 5: Sales View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-5-sales-view)
  - [Building Customer Performance visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-building-customer-performance-visual)
  - [Building Customers GM & NS Plot visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-building-customers-gm--ns-plot-visual)
  - [Building Product Performance visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-building-product-performance-visual)
  - [Building Unit Economics visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-building-unit-economics-visual)
- [Phase 6: Marketing View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-6-marketing-view)
  - [Building Product Performance visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-building-product-performance-visual)
  - [Building Products GM & NS Plot visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-building-products-gm--ns-plot-visual)
  - [Building Unit Economics visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-building-unit-economics-visual)
  - [Building Market Performance visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-building-market-performance-visual)
- [Phase 7: Supply Chain View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-7-supply-chain-view)
  - [Understanding Key Metrics](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-understanding-key-metrics)
  - [Creating Supply Chain Measures](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-creating-supply-chain-measures)
  - [Building Supply Chain visuals](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-building-supply-chain-visuals)
- [Phase 8: Designing Effective Dashboard](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-8-designing-effective-dashboard)
  - [Building Home View landing page](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-building-home-view-landing-page)
  - [Upgrading Finance View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-upgrading-finance-view)
  - [Adding Key Elements to Finance Dashboard](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-adding-key-elements-to-finance-dashboard)
  - [Configuring Navigation Bar](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-configuring-navigation-bar)
  - [Upgrading Supply Chain View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-5-upgrading-supply-chain-view)
  - [Upgrading Sales View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-6-upgrading-sales-view)
  - [Upgrading Marketing View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-7-upgrading-marketing-view)
  - [Incorporating Country level NS, GM & NP Target FY 2022 Data in Finance View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-8-incorporating-country-level-ns-gm--np-target-fy-2022-data-in-finance-view)
  - [Updating Finance View visuals](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-9-updating-finance-view-visuals)
  - [Creating a Dynamic Switch to toggle between LY and Target benchmarks](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-10-creating-a-dynamic-switch-to-toggle-between-ly-and-target-benchmarks)
  - [Configuring BM instead of LY for other Finance View visuals](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-11-configuring-bm-instead-of-ly-for-other-finance-view-visuals)
  - [Setup Dynamic GM% Parameter Slicer](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-12-setup-dynamic-gm-parameter-slicer)
  - [Configure Toggle Button to switch GM % and NP % in Marketing View Performance Plot visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-13-configure-toggle-button-to-switch-gm--and-np--in-marketing-view-performance-plot-visual)
  - [Implement custom Tooltip to show NS $ and GM % Trend in Sales View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-14-implement-custom-tooltip-to-show-ns--and-gm--trend-in-sales-view)
  - [Fix Data Quality Issues](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-15-fix-data-quality-issues)
  - [Create Support View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-16-create-support-view)
  - [Create Info View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-17-create-info-view)
  - [Save DAX Studio Metrics File](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-18-save-dax-studio-metrics-file)
- [Phase 9: Executive View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#phase-9-executive-view)
  - [Importing Market Share Data](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-1-importing-market-share-data)
  - [Configure the Data Model](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-2-configure-the-data-model)
  - [Building Market Share View Visuals](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-3-building-market-share-view-visuals)
  - [Creating the Executive View](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-4-creating-the-executive-view)
  - [Creating Executive KPI Cards](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-5-creating-executive-kpi-cards)
  - [Creating Revenue (NS) by Division & Channel Donut Charts](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-6-creating-revenue-ns-by-division--channel-donut-charts)
  - [Creating Key Insights by Sub Zone Matrix visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-7-creating-key-insights-by-sub-zone-matrix-visual)
  - [Creating Yearly Trend Chart](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-8-creating-yearly-trend-chart)
  - [Creating Market Share Ribbon Chart Visual](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-9-creating-market-share-ribbon-chart-visual)
  - [Creating Top 5 Customers & Products by Revenue Visuals](https://github.com/5ifar/Business_Insights_360/blob/main/Project%20Implementation/Documentation.md#step-10-creating-top-5-customers--products-by-revenue-visuals)

---

## BI 360 Dashboard Overview:

### I. Home View: Dashboard Central navigation with buttons to navigate to all views.

<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Power%20BI%20Dashboard/Dashboard%20Images/Business%20Insights%20360%20Report%20-%20Reinvented%20-%20Home%20View.png" width="100%" height="100%"> </div>

### II. Finance View: Aimed at enhancing financial planning and cost control. 

Features P & L statement, Net Sales Trend over time, Net Sales Breakdown over Products & Customers and more.

<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Power%20BI%20Dashboard/Dashboard%20Images/Business%20Insights%20360%20Report%20-%20Reinvented%20-%20Finance%20View.png" width="100%" height="100%"> </div>

### III. Sales View: Aimed at boosting sales revenue and market share. 

Features Customer performance by NS & GM %, Gross Margin % Variance across Customers & Products, NS Breakdown and more.

<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Power%20BI%20Dashboard/Dashboard%20Images/Business%20Insights%20360%20Report%20-%20Reinvented%20-%20Sales%20View.png" width="100%" height="100%"> </div>

### IV. Marketing View: Aimed at elevating brand visibility and customer engagement. 

Features Segment Performance by NP %, Gross Margin Breakdown, Net Profit % Variance across Regional/Segment Performance and more.

<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Power%20BI%20Dashboard/Dashboard%20Images/Business%20Insights%20360%20Report%20-%20Reinvented%20-%20Marketing%20View.png" width="100%" height="100%"> </div>

### V. Supply Chain View: Aimed at optimizing inventory management and better demand forecasting. 

Features Forecast Accuracy and Net Error Trend, Forecast Accuracy % by Customer & Products and more.

<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Power%20BI%20Dashboard/Dashboard%20Images/Business%20Insights%20360%20Report%20-%20Reinvented%20-%20Supply%20Chain%20View.png" width="100%" height="100%"> </div>

### VI. Executive View: Aimed at providing top level view on overall organizational performance. 

Features NS-GM%-NP%-FA% KPIs, Revenue Contribution by Divison/Channel, Top 5 Customers & Products, Sub Zone performance Market Share Trend and more.

<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Power%20BI%20Dashboard/Dashboard%20Images/Business%20Insights%20360%20Report%20-%20Reinvented%20-%20Executive%20View.png" width="100%" height="100%"> </div>

---

## Conclusion:
The Power BI Dashboard project for AtliQ Hardware provided a comprehensive, data-driven analysis across five critical business functions: Finance, Sales, Marketing, Supply Chain, and Executive. By integrating key business metrics such as Net Sales, Gross Margin, COGS, Net Profit % and Forecast Accuracy % the dashboard offers a holistic view of AtliQ's performance.

The insights gained from this analysis revealed areas of interest, such as robust sales performance and precise forecast accuracy, as well as areas of opportunities for improvement in weak profit margins and supply chain efficiency. Through interactive visualizations and detailed metrics, the dashboard empowers Atliq Hardware's leadership to make informed, strategic decisions that align with their business goals.

---
