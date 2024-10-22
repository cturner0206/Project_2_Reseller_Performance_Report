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
Before creating the report, I thought about what kind of visuals and features I could add that would allow end users to gain valuable insights while also being easily understandable. I knew I wanted to include:
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

### Missing Data Notice
Some months are completely empty of data, which can skew the overall results of the report. It's unclear as to why there are missing data points, but it likey stems from the original database file itself. 
- In 2021, the months of Feburary, April, and June don't have any reseller data (likely data loss/data issue stemming from the dataset)
- December 2023 is also blank (either data loss issue or Q4 was incomplete at the time of creating the dataset). 

An example where this is noticable is the card for PYTD sales in 2023 is at 25.5m while the YTD in 2022 is at 28.19m because the PYTD in 2023 is not accounting for December as there is no data for December in 2023 like in 2022. 


### Sales
- Total sales have increased by 55% from 2021 to 2022 and increased 19.1% from 2022 to 2023.
- In 2021, the United States resellers made up 79.16% of total sales, then Canada at 19.79%, France at 0.53%, and United Kingdom at 0.44%)
	- In 2022, the United States resellers made up 69.60% of total sales, then Canada at 19.44%, United Kingdom at 5.25%, France at 4.93%, Germany at 0.64%, and Australia at 0.18%.)
	- In 2023, the United States resellers made up 57.19% of total sales, then Canada at 15.4%, France at 9.29%, United Kingdom at 8.1%, Germany at 5.36%, and Australia at 4.59%.)
- In 2023, the newer countries of Germany, France, Australia, and the United Kingdom all surpased both the United States and Canada for haivng a higher YTD vs PYTD sales difference.
- In 2021, the United States and Canada experienced postive YTD vs PYTD growth, however, in 2022-2023 growth has significantly slowed down. Several months out of the year have negative YTD vs PYTD values. The negative values range from being down hundreds of thousands to up around 2 million a month. 
	- As for the other four new reseller countries, they all have positve YTD vs PYTD growth since being introduced.
 - In the more reccent years of 2022-2023, current sales trends show that Q1 has been the highest selling quarter. Sales then dip in Q2 and Q3—where the most drastic dip occurs in Q3 over the summer. Sales then go back up in Q4 and peak in Q1 (Note that Q4 of 2023 doesn't have data for December, but given October and November both had sales around 3.3m, it's likey that December would also be near that range to boost the Q4 total to around 9-10m).
	- 2021 shows sales starting low in Q1 and growing into Q4, however, this is expected as 2021 was when Adventure Works just started to have resellers be established. This pattern reflects the natural progression of building a business relationship with resellers


### Quantity
- The big takeaway from quantity is that it follows similar trends to sales—where when sales go up, so does quantity. 
- The change in quantity per country follows the same trend as sales, where the United States and Canada make up most of the total quantity from 2021-2023 and YTD vs PYTD from 2021-2022. In 2023, the newer countries of Germany, France, Australia, and the United Kingdom catch up to the United States in the YTD vs PYTD quantity difference.
- Like sales quarterly trends, quantity also peaks in Q1 then dips down in Q2-Q3 and rises back in Q4. 


### Products 
- Product category trends are consistant, where the bike category makes up the largest percentage of sales, then components as the second best selling, and lastly clothing and accessories that make up less than around 5% of total sales. 
- In 2023, the biggest product shift was the major decline in road bikes and the stark increase in touring bikes in both the United States and Canada. In the other four countries, road bikes sales remained nearly the same while touring bikes had a major increase.
  - In 2022, the components category had around two times the total sales as seen in 2021 and 2023. Road bikes had the most substantially product increase from the prior year when looking at all countries together. Some geographical differences include: United States being down 1.3m YTD in mountain bike sales, France and the United Kingdom had a larger mountain bike demographic along with road bikes, Germany and Australia both had touring bikes as their largest product sales increase though they both had a limited product selection as it was their first year selling.
  - In 2021, mountain bikes and road bikes were by far the best-selling products, except in Germany, where road bikes and road frames were the two main selling products.

  
# Recommendations

- Manage Seasonal Fluctuations: With Q1 being the highest selling quarter, create promotions or special events leading into Q2 and Q3 to sustain momentum and mitigate dips. This could include summer promotions or targeted advertising.
- Address Product Declines: Investigate the reasons behind the decline in road bike sales in the US and Canada. Consider redesigns, marketing campaigns, or bundle offers to rejuvenate interest.
- Focus on Emerging Markets: Given the positive growth in countries like Germany, France, Australia, and the UK, consider tailored marketing strategies for these regions. Allocate resources to enhance brand visibility and establish stronger reseller partnerships.
- Adapt to Regional Preferences: Recognize and promote the product preferences in different markets. For example, emphasize touring bikes in regions where they are gaining popularity.


