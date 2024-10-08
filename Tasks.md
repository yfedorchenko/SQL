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

## Risky Projects

### Problem Statement
Identify projects that are at risk for going over budget based on employee costs compared to the project budget. A project is considered to be overbudget if the cost of all employees assigned to the project is greater than the budget of the project.

You'll need to prorate the cost of the employees to the duration of the project. For example, if the budget for a project that takes half a year to complete is $10K, then the total half-year salary of all employees assigned to the project should not exceed $10K. Salary is defined on a yearly basis, so be careful how to calculate salaries for the projects that last less or more than one year.

### Dataset Information
- **Tables**: `linkedin_projects`, `linkedin_emp_projects`, `linkedin_employees`
- **Columns**:
  - **linkedin_projects**:
    - `id`: Unique identifier for the project.
    - `title`: Name of the project.
    - `budget`: Budget allocated for the project.
    - `start_date`: Project start date.
    - `end_date`: Project end date.
  - **linkedin_emp_projects**:
    - `emp_id`: Employee identifier.
    - `project_id`: Project identifier.
  - **linkedin_employees**:
    - `id`: Employee identifier.
    - `first_name`: First name of the employee.
    - `last_name`: Last name of the employee.
    - `salary`: Annual salary of the employee.

### SQL Solution
```sql
WITH sub_table AS (
    SELECT title, budget, 
           CEIL((end_date::date - start_date::date) * (SUM(salary) / 365)) AS prorated_expenses
    FROM linkedin_projects
    JOIN linkedin_emp_projects ON linkedin_projects.id = linkedin_emp_projects.project_id
    JOIN linkedin_employees ON linkedin_emp_projects.emp_id = linkedin_employees.id
    GROUP BY title, budget, end_date, start_date
)
SELECT title, budget, prorated_expenses
FROM sub_table
WHERE prorated_expenses > budget;
```

## Highest Energy Consumption

### Problem Statement
Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers.

### Dataset Information
- **Tables**: `fb_eu_energy`, `fb_asia_energy`, `fb_na_energy`
- **Columns**:
  - **fb_eu_energy**:
    - `date`: Date of the energy consumption record.
    - `consumption`: Amount of energy consumed (in units).
  - **fb_asia_energy**:
    - `date`: Date of the energy consumption record.
    - `consumption`: Amount of energy consumed (in units).
  - **fb_na_energy**:
    - `date`: Date of the energy consumption record.
    - `consumption`: Amount of energy consumed (in units).

### SQL Solution
```sql
WITH full_table AS (
    SELECT * FROM fb_eu_energy
    UNION ALL
    SELECT * FROM fb_asia_energy
    UNION ALL
    SELECT * FROM fb_na_energy
),
date_totals AS (
    SELECT date, SUM(consumption) AS total
    FROM full_table
    GROUP BY date
)
SELECT date, total
FROM date_totals
WHERE total = (SELECT MAX(total) FROM date_totals)
ORDER BY total DESC;
```

## Word Occurrences in Drafts

### Problem Statement
Find the number of times each word appears in drafts. Output the word along with the corresponding number of occurrences.

### Dataset Information
- **Table Name**: `google_file_store`
- **Columns**:
  - `filename`: Name of the file.
  - `contents`: The text content of the file.

### SQL Solution
```sql
WITH sub_table AS (
    SELECT filename,
           string_to_table(lower(TRIM(REPLACE(REPLACE(contents, '.', ''), ',', ''))), ' ') AS word
    FROM google_file_store
    WHERE filename != 'final.txt'
)
SELECT word, COUNT(*) AS count
FROM sub_table
GROUP BY word
ORDER BY count DESC;
```
