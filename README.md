# ` Sales Performance and Customer Behavior Analysis`
### Table of Content 
  - [Project Overview](#project-overview)
  - [Data Model](#data-model)
  - [Project Requirements](#project-requirements)
  - [Business Questions and Answers](#business-questions-and-answers)
  - [Insights and Recommendations](#insights-and-recommendations)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Project Overview:
The purpose of this project is to analyze sales performance and customer behavior using SQL. The dataset consists of five interconnected tables: Customer, DiscountPeriod, Product, Sales, and SalesReturnReason. This project will focus on understanding critical metrics like sales trends, customer segmentation, return reasons, and discount effectiveness. The goal is to derive actionable insights that can improve business strategies.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Project Requirements:
- Using window analytic functions, such as ROW_NUMBER, RANK, SUM, MAX, LAG, and LAST_VALUE, to improve code efficiency is highly encouraged.
- The use of the SELECT DISTINCT keyword is discouraged, as it is not required to answer any questions.
- Using keywords that restrict the number of records returned, such as TOP, ROWCOUNT, FIRST, and SAMPLE, is highly discouraged and unnecessary when answering questions.
- Subqueries or CTEs are acceptable and may be necessary for some responses.
- Present joins, aggregations, filtering, and grouping using readable SQL.
- Provide clear documentation of SQL scripts and interpretation of results.
- Deliver meaningful insights with recommendations based on the analysis.
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Data Model 

![DataModel ](https://github.com/user-attachments/assets/516c349d-84a7-4149-a59d-41c5542a9f18)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Business Questions and Answers:
##### Note: Please compare your answers with the output file attached in the document section (Output_File_for_Reference). 
 #### 1. What are the total sales per product category, and how do they rank within each category?
```
WITH ProductSales AS (
    SELECT P.ProductCategory, P.ProductSubCategory, SUM(S.SalesAmount) AS TotalSales
    FROM Sales S
    JOIN Product P ON S.ProductID = P.ProductID
    GROUP BY P.ProductCategory, P.ProductSubCategory
)
SELECT ProductCategory, ProductSubCategory, TotalSales,
       RANK() OVER (PARTITION BY ProductCategory ORDER BY TotalSales DESC) AS CategoryRank
FROM ProductSales;

```
#### 2. What is the cumulative total sales per customer over time, and how has discount usage impacted their purchasing?
```
SELECT C.CustomerID, C.CustomerName, S.TransactionTimestamp, S.SalesAmount,
       SUM(S.SalesAmount) OVER (PARTITION BY C.CustomerID ORDER BY S.TransactionTimestamp) AS CumulativeSales,
       LAG(DP.DiscountRate) OVER (PARTITION BY C.CustomerID ORDER BY DP.StartDate) AS PreviousDiscount
FROM Sales S
JOIN Customer C ON S.CustomerID = C.CustomerID
LEFT JOIN DiscountPeriod DP ON C.CustomerID = DP.CustomerID;

```
#### 3. Which product categories and subcategories have the highest return rates, and how do these rates change over time?
```
WITH ReturnRates AS (
    SELECT P.ProductCategory, 
           COUNT(SRR.TransactionID) AS ReturnCount, 
           COUNT(S.TransactionID) AS TotalSales,
           FORMAT(S.TransactionTimestamp, 'yyyy-MM') AS SalesMonth
    FROM Sales S
    JOIN Product P ON S.ProductID = P.ProductID
    LEFT JOIN SalesReturnReason SRR ON S.TransactionID = SRR.TransactionID
    GROUP BY P.ProductCategory, FORMAT(S.TransactionTimestamp, 'yyyy-MM')
)
SELECT ProductCategory, SalesMonth, 
       (ReturnCount * 1.0 / TotalSales) AS ReturnRate,
       LAG(ReturnCount * 1.0 / TotalSales) OVER (PARTITION BY ProductCategory ORDER BY SalesMonth) AS PreviousMonthReturnRate
FROM ReturnRates;
```
#### 4. What are the monthly sales trends and their month-over-month changes?
```
SELECT FORMAT(S.TransactionTimestamp, 'yyyy-MM') AS SalesMonth, 
       SUM(S.SalesAmount) AS MonthlySales,
       SUM(S.SalesAmount) - LAG(SUM(S.SalesAmount)) OVER (ORDER BY FORMAT(S.TransactionTimestamp, 'yyyy-MM')) AS MoMChange
FROM Sales S
GROUP BY FORMAT(S.TransactionTimestamp, 'yyyy-MM');
```
#### 5. Who are the top customers by sales volume, and how does their purchase history evolve?
```
WITH CustomerSales AS (
    SELECT C.CustomerID, C.CustomerName, SUM(S.SalesAmount) AS TotalSales
    FROM Sales S
    JOIN Customer C ON S.CustomerID = C.CustomerID
    GROUP BY C.CustomerID, C.CustomerName
)
SELECT CustomerID, CustomerName, TotalSales, 
       RANK() OVER (ORDER BY TotalSales DESC) AS CustomerRank,
       LAG(TotalSales) OVER (ORDER BY TotalSales DESC) AS PreviousCustomerSales
FROM CustomerSales;
```
#### 6 How do education levels and occupations influence purchasing behavior, ranked by total sales?
```
SELECT C.EducationLevel, C.Occupation, SUM(S.SalesAmount) AS TotalSales,
       RANK() OVER (PARTITION BY C.EducationLevel ORDER BY SUM(S.SalesAmount) DESC) AS OccupationRank
FROM Sales S
JOIN Customer C ON S.CustomerID = C.CustomerID
GROUP BY C.EducationLevel, C.Occupation;
```
#### 7 What are the most frequent reasons for returns, and how do they impact sales performance?
```
SELECT SRR.ReturnReason, COUNT(SRR.TransactionID) AS ReturnCount, SUM(S.SalesAmount) AS ImpactedSales,
       RANK() OVER (ORDER BY COUNT(SRR.TransactionID) DESC) AS ReasonRank
FROM SalesReturnReason SRR
JOIN Sales S ON SRR.TransactionID = S.TransactionID
GROUP BY SRR.ReturnReason;
```
#### 8 How do discounts influence total sales during specific periods, with trends across the discount periods?
```
WITH DiscountedSales AS (
    SELECT DP.StartDate, DP.EndDate, SUM(S.SalesAmount) AS DiscountedSales
    FROM DiscountPeriod DP
    JOIN Sales S ON DP.CustomerID = S.CustomerID
    GROUP BY DP.StartDate, DP.EndDate
)
SELECT StartDate, EndDate, DiscountedSales,
       LAG(DiscountedSales) OVER (ORDER BY StartDate) AS PreviousPeriodSales
FROM DiscountedSales;
```
#### 9 What are the trends in product defects and returns, and how do shipment errors affect return rates?

```
SELECT P.ProductName, 
       COUNT(CASE WHEN SRR.ProductDefect = 1 THEN 1 END) AS DefectReturns, 
       COUNT(CASE WHEN SRR.ShipmentError = 1 THEN 1 END) AS ShipmentErrors,
       RANK() OVER (ORDER BY COUNT(CASE WHEN SRR.ProductDefect = 1 THEN 1 END) DESC) AS DefectRank,
       LAG(COUNT(CASE WHEN SRR.ProductDefect = 1 THEN 1 END)) OVER (ORDER BY COUNT(CASE WHEN SRR.ProductDefect = 1 THEN 1 END) DESC) AS PreviousDefectRank
FROM SalesReturnReason SRR
JOIN Sales S ON SRR.TransactionID = S.TransactionID
JOIN Product P ON S.ProductID = P.ProductID
WHERE SRR.ProductDefect = 1 OR SRR.ShipmentError = 1
GROUP BY P.ProductName;
```
##### 10 How do customers' return behaviors change over time, and what is the overall impact on product categories?
```
WITH CustomerReturns AS (
    SELECT C.CustomerID, C.CustomerName, 
           COUNT(SRR.TransactionID) AS ReturnCount,
           FORMAT(S.TransactionTimestamp, 'yyyy-MM') AS ReturnMonth
    FROM SalesReturnReason SRR
    JOIN Sales S ON SRR.TransactionID = S.TransactionID
    JOIN Customer C ON S.CustomerID = C.CustomerID
    GROUP BY C.CustomerID, C.CustomerName, FORMAT(S.TransactionTimestamp, 'yyyy-MM')
)
SELECT CustomerID, CustomerName, ReturnMonth, ReturnCount,
       LAG(ReturnCount) OVER (PARTITION BY CustomerID ORDER BY ReturnMonth) AS PreviousMonthReturns
FROM CustomerReturns;
```
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### Insights and Recommendations:
- Insight: Certain product categories, like electronics, show higher return rates month over month, indicating potential issues with product quality.
- Recommendation: Introduce more robust quality control measures for these products to reduce returns.

- Insight: Customers who use discounts tend to make larger cumulative purchases but have higher return rates. 
- Recommendation: Implement customer education initiatives during discount periods to reduce returns and improve satisfaction.

- Insight: Shipment errors contribute significantly to returns, particularly for international shipments. 
- Recommendation: Review and optimize the logistics and shipping processes to reduce errors.

### Tools Used
- SQL Server Management Studio - SSMS
- Excel


