-- Given a table of purchases by date, calculate the month-over-month percentage change in revenue. 
-- The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.
-- The percentage change column will be populated from the 2nd month forward and can be calculated as ((this month's revenue - last month's revenue) / last month's revenue)*100.

with grouped_table AS (select TO_CHAR(created_at, 'YYYY-MM') as formatted_date, SUM(value) as revenue,
        lag(sum(value)) over() prev_rv
from sf_transactions
group by formatted_date
Order by formatted_date)

SELECT formatted_date, ROUND((revenue - LAG(revenue) OVER(Order by formatted_date)) / LAG(revenue) OVER(Order by formatted_date) * 100, 2) as running_diff_perc
FROM grouped_table
