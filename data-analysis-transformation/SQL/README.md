# ⚙️ SQL procedure & App Script Automated Analysis Reporting

## 🧠 Business Problem
The company needed to automate reports that they had been doing manually. This process was repetitive, error-prone and time-consuming.
The stored procedure is designed for business analytics and portfolio demonstrations.  
It aggregates total monthly sales per product category and computes the month-over-month growth percentage.  
Useful for dashboards, reports, or showcasing analytical SQL capabilities.

## 🗂 Process & Data Sources
- Excel / Google Sheets
- Databases

## 🛠 Tools & Technologies
- SQL
- App Script
- Google Sheet
- Microsoft SQL Server (T-SQL 2017+)  

## ⚙️ Solution
I designed a workflow that:
- Cleans and transforms data in the sheet
- A procedure that returns monthly sales totals by category and calculates the percentage growth compared to the previous month.

## 🚀 Impact
- Significant reduction in manual clean up
- Improved data accuracy
- Faster access to updated insights

## 🖼 Workflow Preview

### 💻​ App Script Sheet
![App Script Sheet](https://github.com/LauraFrey11/Portfolio/blob/main/App%20Script%20Sheet.jpg)

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
