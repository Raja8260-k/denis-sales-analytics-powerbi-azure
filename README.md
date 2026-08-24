Denis Sales Analytics & Azure Data Pipeline

Overview :-
Implemented an end-to-end data pipeline and BI solution following client BRD/FRD specifications. The pipeline extracts multi-source raw files (JSON, CSV, Excel, and an on-prem MySQL database), processes them through Azure Data Factory into Azure SQL Database, and visualizes KPIs in a 3-page Power BI dashboard.

 Pipeline Architecture 
- Data Ingestion:-
  - Cloud storage (Azure Blob & ADLS Gen2) ingested via Azure Integration Runtime (AIR).
  - On-premises MySQL Product table ingested via Self-Hosted Integration Runtime (SHIR).
- ADLS Medallion Layers:-
  - Bronze: Raw staging for source files across disparate formats.
  - Silver: Cleaned, schema-standardized CSV files.
  - Gold: Consolidated analytical data staged for database loading.
- Serving Layer:- Azure SQL Database consumed directly by Power BI (Import mode).

 Data Cleaning & Transformations (Azure Data Factory / ADF)
- Folder Ingestion: Built dynamic file loading pipelines in ADF to combine yearly files without errors on file additions or deletions.
- String Cleaning: Implemented transformation logic in ADF to strip the "ID - " prefix from 'SalesRepID' and 'SubCategoryID'.
- Geographic Data: Split combined Location fields into Country and Town, assigned correct geographical data types, and created composite 'GeoKey' values to establish relationships with the Geography table.
- Data Model: Structured a Star Schema linking dimension tables (Product, SalesRep, SubCategories, Categories, Geography, Calendar) to the central 'Sales' fact table.

DAX Measures

      1. Base Measures
- TotalRevenue = SUM(Denis_db[TotalRevenue])
- TotalCost = SUM(Denis_db[TotalCost])
- GrossProfit = [TotalRevenue] - [TotalCost]
- Gross Profit% = DIVIDE([GrossProfit], [TotalRevenue], 0)
- Total Units = SUM(Denis_db[Units])
  
      2. Quarterly Measures (QBR)
- Previous.Quarter Revenue = CALCULATE([TotalRevenue], DATEADD(Calender_TB[Date], -1, QUARTER))
- Prev. Quarter Cost = CALCULATE([TotalCost], DATEADD(Calender_TB[Date], -1, QUARTER))
- Prev. Quarter Profit = CALCULATE([GrossProfit], DATEADD(Calender_TB[Date], -1, QUARTER))
- Prev. Quarter Profit% = DIVIDE([Prev. Quarter Profit], [Previous.Quarter Revenue], 0)
- QOQ Rev. Growth% = DIVIDE([TotalRevenue] - [Previous.Quarter Revenue], [Previous.Quarter Revenue], 0)
- QOQ cost. Growth% = DIVIDE([TotalCost] - [Prev. Quarter Cost], [Prev. Quarter Cost], 0)
- QOQ Profit. Growth% = DIVIDE([GrossProfit] - [Prev. Quarter Profit], [Prev. Quarter Profit], 0)
- QOQ Profit%. Growth% = [Gross Profit%] - [Prev. Quarter Profit%]
- 
      3. Monthly Measures (MBR)
- Prev. Month Revenue = CALCULATE([TotalRevenue], DATEADD(Calender_TB[Date], -1, MONTH))
- Prev. Month Cost = CALCULATE([TotalCost], DATEADD(Calender_TB[Date], -1, MONTH))
- Prev. Month Profit = CALCULATE([GrossProfit], DATEADD(Calender_TB[Date], -1, MONTH))
- Prev. Month Profit% = DIVIDE([Prev. Month Profit], [Prev. Month Revenue], 0)
- MoM Rev. Growth% = DIVIDE([TotalRevenue] - [Prev. Month Revenue], [Prev. Month Revenue], 0)
- MoM Cost. Growth% = DIVIDE([TotalCost] - [Prev. Month Cost], [Prev. Month Cost], 0)
- MoM Profit. Growth% = DIVIDE([GrossProfit] - [Prev. Month Profit], [Prev. Month Profit], 0)
- MoM Profit%. Growth% = [Gross Profit%] - [Prev. Month Profit%]
- Actual sales Days = DISTINCTCOUNT(Denis_db[Date])
- Avg Revenue Per Day = DIVIDE([TotalRevenue], [Actual sales Days], 0)

Dashboard & Power BI Service Deployment
1. Executive & QBR View: Tracks macro KPIs (Revenue: $126.01M, Profit: $86.89M, Margin: 68.95%) and quarter-over-quarter comparison tables with conditional formatting.
2. MoM & MBR View: Monthly sales breakdown, daily sales run-rate, and category analysis.
3. Product & Geo View: Profit vs. Revenue distribution, product ranking (Quad, Carlota, Magnum), and town-level slicers.
4. Security & Administration:
   - Configured Row-Level Security (RLS) to restrict regional/sales data access based on user roles.
   - Published report to dedicated Power BI Service workspace and managed role-based access control (Viewer/Contributor).
   - Set up automated scheduled refresh to keep dashboards aligned with the cloud database.

Tools & Technologies
- Azure Data Factory, ADLS Gen2, Azure SQL DB, MySQL
- Power BI Desktop, Power BI Service (RLS, Scheduled Refresh, Workspace Management), DAX, Excel
