# ⚙️ SQL Procedure & App Script Automated Reporting

## 🧠 Business Problem
The company needed to automate analytical reports that were previously
generated manually. This process was repetitive, error-prone and
time-consuming, making it difficult to deliver consistent and timely insights.

Additionally, the solution was designed to support business analytics use cases
and serve as a portfolio demonstration of advanced SQL capabilities.

## 🗂 Process & Data Sources
- Excel / Google Sheets
- Relational databases

## 🛠 Tools & Technologies
- SQL
- Microsoft SQL Server (T-SQL 2017+)
- Google Apps Script
- Google Sheets

## ⚙️ Solution
I designed an automated reporting workflow that:
- Cleans and transforms data directly within Google Sheets
- Executes a SQL stored procedure to aggregate monthly sales by product category
- Calculates month-over-month growth percentages
- Returns structured, analytics-ready results for dashboards and reports

The stored procedure is optimized for business analytics and repeatable reporting.

## 🚀 Impact
- Significant reduction in manual data preparation
- Improved data accuracy and consistency
- Faster access to up-to-date analytical insights

## 🖼 Workflow Preview

### 💻​ App Script Sheet
![App Script Sheet](images/App%20Script%20Sheet.jpg)

### 💻​ SQL Stored Procedure: `sp_GetMonthlySalesGrowthByCategory`

```sql
USE YourDatabaseName;
GO

CREATE OR ALTER PROCEDURE sp_GetMonthlySalesGrowthByCategory
    @Year INT
AS
BEGIN
    SET NOCOUNT ON;

    /*
    =============================================================
     Author: Laura
     Project: SQL Portfolio – Sales Analytics
     Date: 2025-10-21

     Description:
        Returns monthly total sales by category and calculates 
        the percentage growth compared to the previous month.

     Parameters:
        @Year INT – The year to analyze.

     Output Columns:
        - Category        (VARCHAR)
        - Month           (YYYY-MM)
        - TotalSales      (DECIMAL)
        - GrowthPercent   (% change from previous month)

     Formula:
        ((CurrentMonth - PreviousMonth) / PreviousMonth) * 100
    =============================================================
    */

    ;WITH MonthlySales AS (
        SELECT 
            c.CategoryName AS Category,
            FORMAT(v.SaleDate, 'yyyy-MM') AS [Month],
            SUM(v.Quantity * p.Price) AS TotalSales
        FROM Sales v
        INNER JOIN Products p ON v.ProductID = p.ProductID
        INNER JOIN Categories c ON p.CategoryID = c.CategoryID
        WHERE YEAR(v.SaleDate) = @Year
        GROUP BY c.CategoryName, FORMAT(v.SaleDate, 'yyyy-MM')
    )
    SELECT 
        Category,
        [Month],
        TotalSales,
        ROUND(
            CASE 
                WHEN LAG(TotalSales) OVER (PARTITION BY Category ORDER BY [Month]) = 0 
                    THEN NULL
                ELSE 
                    ((TotalSales - LAG(TotalSales) OVER (PARTITION BY Category ORDER BY [Month])) 
                    / LAG(TotalSales) OVER (PARTITION BY Category ORDER BY [Month])) * 100
            END, 2
        ) AS GrowthPercent
    FROM MonthlySales
    ORDER BY Category, [Month];

END;
GO
