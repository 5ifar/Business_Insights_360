# 🧬 Business Insights 360 Project Phase-wise Implementation
--Brief Intro--

---

## Table of Contents
- [Phase 1: Data Wrangling](#phase-1-data-wrangling)

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
