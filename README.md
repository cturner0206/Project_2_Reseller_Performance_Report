# Project 2: PowerBI Reseller Performance Report
Adventure Works Reseller Sales Performance Report

  <img src="https://github.com/user-attachments/assets/c0cdb122-3d3c-4873-b796-49ade112d396" alt="dash" width="900" />


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

To build the report 


<img src="https://github.com/user-attachments/assets/a6ac4352-e5a0-43c9-96cc-85142f32f8c5" alt="model" width="700" />



# Findings

.
  
# Recommendations

.


