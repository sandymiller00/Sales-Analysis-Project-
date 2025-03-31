# SQL Sales Data Analysis Project

## Project Overview
This project analyzes sales data using SQL Server to uncover insights about product performance, sales trends, and customer purchasing behavior.

## Technologies Used
- SQL Server Management Studio 20
- (Optional: Python for visualization if you used it)

## Dataset
The dataset contains:
- 800+ sales transactions
- 15+ different products
- Sales across multiple cities
- December 2019 timeframe

## Key Findings
1. Top performing products: Macbook Pro Laptop, iPhone
2. Peak sales hours: 11AM-1PM and 7PM-9PM
3. Highest sales city: San Francisco

## Setup Instructions

### Database Setup
1. Create the database:
```sql
CREATE DATABASE Sales Analysis;
USE [Sales Analysis]
GO
```

2. Run the table creation script:
```sql
CREATE TABLE [dbo].[Sales Data](
	[column1] [smallint] NOT NULL,
	[Order_ID] [int] NOT NULL,
	[Product] [nvarchar](50) NOT NULL,
	[Quantity_Ordered] [tinyint] NOT NULL,
	[Price_Each] [float] NOT NULL,
	[Order_Date] [datetime2](7) NOT NULL,
	[Purchase_Address] [nvarchar](50) NOT NULL,
	[Month] [tinyint] NOT NULL,
	[Sales] [float] NOT NULL,
	[City] [nvarchar](50) NOT NULL,
	[Hour] [tinyint] NOT NULL
) ON [PRIMARY]
GO
```

### Data Import
```sql
-- scripts/data_import.sql
```

## How to Run the Analysis
Execute the queries in this order:
1. `database_setup.sql`
2. `data_import.sql`
3. `analysis_queries.sql`

## Data Exploration and Cleaning

Check first 10 rows
SELECT TOP 10 * FROM [dbo].[Sales Data];
GO

-- Check for missing values
```sql
SELECT 
    COUNT(*) - COUNT(order_id) AS missing_order_ids,
    COUNT(*) - COUNT(product) AS missing_products,
    COUNT(*) - COUNT(sales) AS missing_sales
FROM [dbo].[Sales Data]; 
GO
```

-- Check for duplicates
```sql
SELECT order_id, COUNT(*) as count
FROM [dbo].[Sales Data]
GROUP BY order_id
HAVING COUNT(*) > 1;
GO
```

### Basic Analysis Queries

1.Total Sales Revenue
```sql
SELECT SUM(sales) AS total_revenue, FORMAT(SUM(sales), 'C') AS formatted_revenue
FROM [dbo].[Sales Data];
GO
```

2. Sales by Product (top 10)
 ```sql
 SELECT TOP 10
    product,
    SUM(quantity_ordered) AS total_quantity,
    SUM(sales) AS total_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY product
ORDER BY total_sales DESC;
GO
```


3. Sales by City
```sql
SELECT 
    city,
    SUM(sales) AS total_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales,
    ROUND(SUM(sales) * 100.0 / (SELECT SUM(sales) FROM [dbo].[Sales Data]), 2) AS percentage
FROM [dbo].[Sales Data]
GROUP BY city
ORDER BY total_sales DESC;`
GO
```
   

 4. Hourly Sales Trends
```sql
SELECT 
    hour,
    SUM(sales) AS hourly_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY hour
ORDER BY hour;
GO
```

### Advanced Analysis

 1. Products Frequently Sold Together
```sql
WITH product_pairs AS (
    SELECT 
        a.order_id,
        a.product AS product1,
        b.product AS product2
    FROM sales a
    JOIN sales b ON a.order_id = b.order_id AND a.product < b.product
)
SELECT TOP 10
    product1, 
    product2, 
    COUNT(*) AS times_purchased_together
FROM product_pairs
GROUP BY product1, product2
ORDER BY times_purchased_together DESC;
GO
```

2. Monthly Sales Growth (though all data is December)
```sql
SELECT 
    month,
    SUM(sales) AS monthly_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM sales
GROUP BY month
ORDER BY month;
GO
```

 3. Average Order Value
```sql
SELECT 
    AVG(order_total) AS avg_order_value,
    FORMAT(AVG(order_total), 'C') AS formatted_avg
FROM (
    SELECT 
        order_id, 
        SUM(sales) AS order_total
    FROM sales
    GROUP BY order_id
) AS order_totals;
GO
```

4. Sales by Day of Week
```sql
`SELECT 
    DATENAME(WEEKDAY, order_date) AS day_of_week,
    SUM(sales) AS total_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY DATENAME(WEEKDAY, order_date)
ORDER BY total_sales DESC;
GO
```

### Advanced Analysis

 1. Products Frequently Sold Together
```sql
WITH product_pairs AS (
    SELECT 
        a.order_id,
        a.product AS product1,
        b.product AS product2
    FROM sales a
    JOIN sales b ON a.order_id = b.order_id AND a.product < b.product
)
SELECT TOP 10
    product1, 
    product2, 
    COUNT(*) AS times_purchased_together
FROM product_pairs
GROUP BY product1, product2
ORDER BY times_purchased_together DESC;
GO
```sql

 
 2. Monthly Sales Growth (though all data is December)
```sql
`SELECT 
    month,
    SUM(sales) AS monthly_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY month
ORDER BY month;
GO
```

3. Average Order Value
```sql
SELECT 
    AVG(order_total) AS avg_order_value,
    FORMAT(AVG(order_total), 'C') AS formatted_avg
FROM (
    SELECT 
        order_id, 
        SUM(sales) AS order_total
    FROM [dbo].[Sales Data]
    GROUP BY order_id
) AS order_totals;
GO
```

4. Sales by Day of Week
```sql
SELECT 
    DATENAME(WEEKDAY, order_date) AS day_of_week,
    SUM(sales) AS total_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY DATENAME(WEEKDAY, order_date)
ORDER BY total_sales DESC;
GO
```

 ### Time Series Analysis

1. Daily Sales Trend
```sql
SELECT 
    CAST(order_date AS DATE) AS sales_date,
    SUM(sales) AS daily_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY CAST(order_date AS DATE)
ORDER BY sales_date;
GO
```

2. Rolling 7-Day Average
```sql
WITH daily_sales AS (
    SELECT 
        CAST(order_date AS DATE) AS sales_date,
        SUM(sales) AS daily_sales
    FROM [dbo].[Sales Data]
    GROUP BY CAST(order_date AS DATE)
)
SELECT 
    sales_date,
    daily_sales,
    AVG(daily_sales) OVER (ORDER BY sales_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS rolling_7day_avg
FROM daily_sales
ORDER BY sales_date;
GO
```

 ### Create Views for Reporting

 Sales by product view
```sql
CREATE VIEW vw_sales_by_product AS
SELECT 
    product,
    SUM(quantity_ordered) AS total_quantity,
    SUM(sales) AS total_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY product;
GO
```

Daily sales view
```sql
CREATE VIEW vw_daily_sales AS
SELECT 
    CAST(order_date AS DATE) AS sales_date,
    SUM(sales) AS daily_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales
FROM [dbo].[Sales Data]
GROUP BY CAST(order_date AS DATE);
GO
```

City performance view
```sql
CREATE VIEW vw_city_performance AS
SELECT 
    city,
    COUNT(DISTINCT order_id) AS order_count,
    SUM(sales) AS total_sales,
    FORMAT(SUM(sales), 'C') AS formatted_sales,
    ROUND(SUM(sales) * 100.0 / (SELECT SUM(sales) FROM [dbo].[Sales Data]), 2) AS percentage
FROM [dbo].[Sales Data]
GROUP BY city;
GO
```

### Export query results to CSV using SQL Server Import/Export Wizard
### Or use this command (requires appropriate permissions):
```sql
EXEC xp_cmdshell 'bcp "SELECT * FROM Sales Analysis" queryout "C:\temp\sales_export.csv" DESKTOP-RB7SA33\SQLEXPRESS';
GO
```

### For specific queries:
```sql
EXEC xp_cmdshell 'bcp "SELECT product, SUM(sales) AS total_sales FROM Sales Analysis GROUP BY product" queryout "C:\temp\product_sales.csv"  ';
GO
```



 ### Create a stored procedure for monthly report

Add an index for better performance on large datasets
```sql
CREATE INDEX idx_sales_order_date ON sales(order_date);
CREATE INDEX idx_sales_product ON sales(product);
CREATE INDEX idx_sales_city ON sales(city);
GO
```

 
Get table size information
```sql
EXEC sp_spaceused '[dbo].[Sales Data]';
GO
```

### First check if the procedure exists and drop it if it does
```sql
IF EXISTS (SELECT * FROM sys.objects WHERE type = 'P' AND name = 'sp_monthly_sales_report')
BEGIN
    DROP PROCEDURE sp_monthly_sales_report;
    PRINT 'Existing procedure drop';
END
GO
```

### Then create the new version
```sql
CREATE PROCEDURE sp_monthly_sales_report
AS
BEGIN
    SELECT 
        DATENAME(MONTH, order_date) AS month,
        YEAR(order_date) AS year,
        SUM(sales) AS total_sales,
        FORMAT(SUM(sales), 'C') AS formatted_sales,
        COUNT(DISTINCT order_id) AS order_count
    FROM [dbo].[Sales Data]
    GROUP BY DATENAME(MONTH, order_date), YEAR(order_date), MONTH(order_date)
    ORDER BY YEAR(order_date), MONTH(order_date);
END;
GO
```

```sql
PRINT 'Procedure created successfully';
GO
```

## License
This project is licensed under the MIT License.
