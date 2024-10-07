# SQL Portfolio

## Table of Contents
1. [Monthly Percentage Difference](#monthly-percentage-difference)
2. [New Products](#new-products)
3. [Risky Projects](#risky-projects)
4. [Highest Energy Consumption](#highest-energy-consumption)
5. [Word Occurrences in Drafts](#word-occurrences-in-drafts)

---

## Monthly Percentage Difference

### Problem Statement
Given a table of purchases by date, calculate the month-over-month percentage change in revenue. The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.

### Dataset Information
- **Table Name**: `sf_transactions`
- **Columns**:
  - `created_at`: The date of the transaction.
  - `value`: The revenue value of the transaction.

### SQL Solution
```sql
WITH grouped_table AS (
    SELECT TO_CHAR(created_at, 'YYYY-MM') AS formatted_date,
           SUM(value) AS revenue,
           LAG(SUM(value)) OVER (ORDER BY TO_CHAR(created_at, 'YYYY-MM')) AS prev_revenue
    FROM sf_transactions
    GROUP BY formatted_date
)
SELECT formatted_date, 
       ROUND(((revenue - prev_revenue) / prev_revenue) * 100, 2) AS running_diff_perc
FROM grouped_table;
```

---

## New Products
### Problem Statement
Write a query to count the net difference between the number of products companies launched in 2020 and those launched in the previous year. Output the name of the companies and the net difference.

### Dataset Information
- **Table Name**: `car_launches`
- **Columns**:
  - `year`: The year of product launches.
  - `company_name`: Name of the company.
  - `product_name`: Name of the product.
 
### SQL Solution
```sql
WITH query1 AS (
    SELECT company_name, COUNT(*) AS total_product
    FROM car_launches
    WHERE year = 2020
    GROUP BY company_name
),
query2 AS (
    SELECT company_name, COUNT(*) AS total_product
    FROM car_launches
    WHERE year = 2019
    GROUP BY company_name
)
SELECT a.company_name, (a.total_product - b.total_product) AS total_launch
FROM query1 a
INNER JOIN query2 b ON a.company_name = b.company_name;
```
