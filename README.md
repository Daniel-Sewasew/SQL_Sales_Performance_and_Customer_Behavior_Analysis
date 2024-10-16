# Sales Performance and Customer Behavior Analysis
### Project Overview:
The purpose of this project is to analyze sales performance and customer behavior using SQL. The dataset consists of five interconnected tables: Customer, DiscountPeriod, Product, Sales, and SalesReturnReason. This project will focus on understanding critical metrics like sales trends, customer segmentation, return reasons, and discount effectiveness. The goal is to derive actionable insights that can improve business strategies.
### Project Requirements:
- It highly encouraged the use of window analytic functions to improve code efficiency, i.e.
ROW_NUMBER, RANK, SUM, MAX, LAG, LAST_VALUE, etc.
- It discourages the use of the SELECT DISTINCT keyword phrase. This phrase is not required to 
answer any questions.
- It's highly discouraged the use of keywords that restrict the number of records returned, i.e., TOP, ROWCOUNT, 
FIRST, SAMPLE. These keywords are not required to answer any questions.
- All answers should be written using a single ANSI SQL statement. No procedural SQL will be accepted as a 
valid answer.
- Subqueries or CTEs are acceptable and will be necessary for some responses.
- Understand joins, aggregations, filtering, and grouping in SQL.
- Clear documentation of SQL scripts and interpretation of results.
- Presentation of insights with recommendations based on the analysis.
### Business Questions and Answers:
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
