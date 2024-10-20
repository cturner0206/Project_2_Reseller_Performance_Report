# Project 2: PowerBI Reseller Performance Report
Adventure Works Reseller Sales Performance Report

   
<img src="https://github.com/user-attachments/assets/fcd4f93a-99cf-4ac3-a002-2ed5813cd5ce" alt="dash" width="900" />

# Table of Contents 
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Data Preparation and Cleaning](#data-preparation-and-cleaning)
- [Data Analysis and Building the Report](#data-analysis-and-building-the-report)
- [Findings](#findings)
- [Recommendations](#recommendations)

# Project Overview

The goal of this project was to query necessary data from the Adventure Works database (SSMS) and use it to create an interactive dashboard in PowerBI. The dashboard aims to provide insights into Adventure Work's reseller sales performance from 2021-2023 to look at present and past sales trends, track new reseller regions being introduced, look into product popularity, and differences in regional sales to facilitate informed decision-making and future strategic planning. 

# Data Source

The data being used for this project is from the AdventureWorksDW2022.bak dataset. 

# Data Preparation and Cleaning

The initial data preparation/cleaning phase involved:

- Loading and restoring the AdventureWorksDW2022.bak file in SQL Server Management Studio.
- Looking through the different database tables to get a better understanding of the data provied and the connections between the tables.
- Given the numerous tables, I had to decide what information would be worth querying to be used for the report. I decided on having one sales fact table and two dimension tables of resellers and products.
- The data cleaning consisted of renaming columns, rounding values, selecting an order date range, converting the date column, and joining tables.

The final three table queries:

```sql
--Sales Fact Table 
SELECT 
	DP.ProductKey AS ProductID,
	OrderQuantity,
	ROUND(UnitPrice,2) AS UnitPrice,
	ROUND(SalesAmount,2) AS SalesAmount,
	CONVERT(DATE,OrderDate) AS OrderDate,
	ResellerName,
	FRS.ResellerKey
FROM FactResellerSales FRS
JOIN DimProduct DP ON FRS.ProductKey = DP.ProductKey
JOIN DimReseller DRS ON FRS.ResellerKey = DRS.ResellerKey
WHERE OrderDate BETWEEN '2021-01-01' AND '2023-12-31'
ORDER BY OrderDate; 
```

```sql
--Dim Products Table
SELECT 
	ProductKey AS ProductID,
	DPC.ProductCategoryKey AS ProductCategoryID,
	EnglishProductCategoryName AS ProductCategory,
	DPS.ProductSubcategoryKey AS ProductSubcategoryID,
	EnglishProductSubcategoryName AS ProductSubcategory,
	Size,
	Weight
FROM DimProductCategory DPC
JOIN DimProductSubcategory DPS ON DPC.ProductCategoryKey = DPS.ProductCategoryKey
JOIN DimProduct DP ON DPS.ProductSubcategoryKey = DP.ProductSubcategoryKey
ORDER BY PRODUCTID;
```

```sql
--Dim Resellers Table
SELECT
	ResellerKey AS ResellerID,
	ResellerAlternateKey AS ResellerAlternateID,
	ResellerName,
	AddressLine1,
	AddressLine2,
	EnglishCountryRegionName AS Country,
	City,
	PostalCode
FROM DimReseller DRS
JOIN DimGeography DG ON DRS.GeographyKey = DG.GeographyKey
ORDER BY ResellerID;
```
    
# Data Analysis and Building the Report

## Report Brainstorm
Before creating the report, I thought about what kind of visuals and features I could add that would allow end users to easily understand the data and gain valuable insights from. I knew I wanted to include:
- A Slicer between sales and quantity
- YTD vs PYTD comparison
- MoM comparison
- Filter to swap between different countries 
- Top performing resellers
- Product category performance 


## Creating the Report 
Steps in creating the report: 

1. Imported the SQL queries into PowerBI to create the fact and dim tables. 
2. Created a Dim Date table to allow for date filtering, as well as a calculated column to determine if a date is one year in the past for when filtering for PYTD later down the line.
3. Connected the different tables in the model view (STAR schema).
<p align="center">   
<img src="https://github.com/user-attachments/assets/a6ac4352-e5a0-43c9-96cc-85142f32f8c5" alt="model" width="600" />
</p> 

4. Created most of the measures (did the dynamic title measures after the visuals had been created). The different types of measures included YTD, PYTD, YTD vs PYTD, and switch measures.
<p align="center">     
<img src="https://github.com/user-attachments/assets/b24730e4-31a5-48fc-85c0-59f12a6d4d07" alt="measures" width="200" />
</p> 

Examples of some of the DAX used to create measures: 

```dax
Switch_PYTD = 
VAR selected_value =SELECTEDVALUE(Slicer_Values[Values])
VAR result = SWITCH(selected_value,
    "Sales", [PYTD_Sales],
    "Quantity", [PYTD_Quantity],
    BLANK()
)
RETURN
result
```
```dax
Inpast = 
VAR lastsalesdate = MAX(Fact_Sales[OrderDate])
VAR lastsalesdatePY = EDATE(lastsalesdate,-12)
RETURN
Dim_Date[Date] <= lastsalesdatePY
```
```dax
PYTD_Sales = 
    CALCULATE(
        [Sales], 
        SAMEPERIODLASTYEAR(Dim_Date[Date]), 
        Dim_Date[Inpast] = TRUE()
    )
```

5. Built the different visuals (cards, tree map, waterfall chart, bar chart, column chart).
   - Altered color palette.
   - Created sales/quantity slicer and year select dropdown.
   - Applied conditional formating on the visuals to make them easier to understand.
   - Added drill downs in the waterfall chart (month -> country -> product) and in the column chart (quarter -> month) to allow for further analysis and data exploration.
   - Created dynamic titles for the report and visuals when switching between sales and quanity slicer. 


# Findings

- Sales have increased by 55% from 2021 to 2022 and increased 19.1% from 2022 to 2023.
- .

### Missing Data Notice
Some months are completely empty of data, which can skew the overall results of the report. It's unclear as to why there are missing data points, but it likey stems from the original database file itself. 
- In 2021, the months of Feburary, April, and June don't have any reseller data (likely data loss/data issue stemming from the dataset)
- December 2023 is also blank (either data loss issue or Q4 was incomplete at the time of creating the dataset). 

 An example where this is noticable is the card for PYTD sales in 2023 is at 25.5m while the YTD in 2022 is at 28.19m because the PYTD in 2023 is not accounting for December as there is no data for December in 2023 like in 2022. 

  
# Recommendations

.


