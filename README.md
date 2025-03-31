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
CREATE DATABASE SalesAnalysis;
GO
```

2. Run the table creation script:
```sql
-- scripts/database_setup.sql
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

## Sample Query
```sql
-- Top 5 products by revenue
SELECT TOP 5
    product,
    SUM(sales) AS total_revenue
FROM sales
GROUP BY product
ORDER BY total_revenue DESC;
```

## License
This project is licensed under the MIT License.
