# <img src="https://miro.medium.com/v2/resize:fit:1400/1*8bUjUiCWk0VhS8-lgAj0Og.png" width="4%" height="4%"> Business Insights 360

This repository serves as my documentation for the AtliQ Hardwares Business Insights 360 - Power BI Project.
It was created as a self-learning project for the course: [Get Job Ready: Power BI Data Analytics for All Levels 2.0](https://codebasics.io/courses/power-bi-data-analysis-with-end-to-end-project) by [Codebasics](https://codebasics.io/).

The entire project has been implemented using Microsoft Power BI Desktop 2.128.751.0 and published on Microsoft Power BI Service.

The project data files have not been uploaded to this repository in compliance with Codebasics Data & Content Distribution Policy.

---

## Contents:
Please find the sectional links for the project below:
- []()

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

## Tools used:
1. Microsoft Power BI: for Data ETL, Data Modelling, Data Visualization & Dashboarding
2. GitHub - for Documentation

## Skills & Methodologies implemented:
1. Data Cleaning: **Power Query**
2. Data Manipulation: **DAX Measures & Columns, Parameters**
3. Data Modelling
4. Data Visualization: **Custom Tooltip**
5. Dashboarding: **Filters, Custom Icon Buttons, Slicers, Bookmarks, Page Navigation**
6. Report Publishing: **PBI Service and Report Optimization**
7. Documentation

---

## About the Dataset:

### Data Sources:
The dataset contains 11 tables in total, namely -
- dim_customer: 209 records | 5 columns
- dim_market: 27 records | 3 columns
- dim_product: 397 records | 6 columns
- fact_forecast_monthly: 1,885,941 records | 4 columns
- dim_freight_cost: 135 records | 4 columns
- dim_manufacturing_cost: 1,197 records | 3 columns
- dim_market_share: 737 records | 6 columns
- dim_ns_gm_target: 321 records | 5 columns
- dim_operational_expense: 113 records | 4 columns
- dim_post_invoice_deductions: 2,063,076 records | 5 columns
- fact_sales_monthly: 1,885,941 records | 4 columns

### Data Model - ERD:
<div align="center"> <img src="https://github.com/5ifar/Business_Insights_360/blob/main/Data%20Model/Data%20Model%20Final.PNG" width="100%" height="100%"> </div>

## Data Integrity:
ROCCC Evaluation:
- Reliability: MED - The raw dataset is created and updated by Codebasics. It has total 9 files. All of them were utilized in the analysis.
- Originality: HIGH - First party provider (Codebasics)
- Comprehensiveness: HIGH - Total 11 Files with a total of around 5.8 Million records were provided. Dataset contains multiple dimension parameters for Customers & Products as well as comprehensive sales transaction data.
- Current: LOW - Dataset was updated upto FY 2022 i.e almost 2 years old. So its not very relevant. Any trends observed and insights gained need to be comprehended as a general (not FY-specific) trend.
- Citation: LOW - No official citation/reference available.

---

