select database();
USE walmart_db;
SELECT * FROM walmart;

SELECT COUNT(*) FROM walmart;

SELECT 
	PAYMENT_METHOD, 
    COUNT(*) 
FROM WALMART
GROUP BY payment_method;

SELECT COUNT(DISTINCT Branch)
FROM walmart;

-- Buiseness problems
-- find the different payment method  and number of transaction, number of qty sold
SELECT 
	PAYMENT_METHOD, 
    COUNT(*) as no_payments,
    SUM(quantity) as no_qty_sold
FROM WALMART
GROUP BY payment_method;

-- identify the highest category in each branch, displaying the branch , category
-- AVG RATING
SELECT *
FROM
(
SELECT 
	Branch,
    category,
    AVG(rating) as avg_rating,
    RANK() OVER (PARTITION BY Branch ORDER BY AVG(rating) DESC) AS RANKED
    FROM walmart
    GROUP BY Branch, category
    ) AS RNK
    WHERE RANKED = 1;
   
-- identify the busiest day for each branch based  on the number of transactions

SELECT * 
FROM
  (SELECT 
    Branch,
    DATE_FORMAT(STR_TO_DATE(date, '%d/%m/%y'), '%W') AS day_name,
    COUNT(*) as no_transactions,
    RANK() OVER (PARTITION BY BRANCH ORDER BY COUNT(*) DESC) AS RNK
FROM walmart
GROUP BY Branch, day_name
) AS RANKING
where RNK = 1;

-- determine the average, minimum, and maximum rating of catrgory for each city.
-- list the city, average_rating, min_rating, and max_rating.

SELECT 
	CITY,
    MIN(RATING) AS MIN_RATING,
    MAX(RATING) AS MAX_RATING,
    AVG(RATING) AS AVG_RATING
FROM WALMART    
GROUP BY CITY, CATEGORY

-- calculate the total profit for each category  by considering total_profit as 
-- (unit_price * quantity * profit_margin).
-- list category and total_profit, ordered from highest to lowest profit.

-- I'm using CTE HERE

select
	category,
    sum(total) as total_revenue,
    SUM(total * profit_margin) as profit
FROM walmart
group by category;

-- determine the most common payment method for each branch.
-- display branch and the prefered_payment_method.

with cte
as
(select 
	branch,
    payment_method,
    count(*) as total_trans       								-- HERE I'M DOING COUNTING
 from walmart
 group by branch, payment_method
 ),
 ranked as (
 select 
	branch,
    payment_method,
    total_trans,															-- HERE I'M DOING RANKING
    rank() over(partition by branch order by total_trans desc) as ranke
from cte
)
select 
	branch,
    payment_method,
    total_trans,
    ranke
from ranked													-- HERE I'M FILTERING WITH RANK 1
where ranke = 1
order by branch


-- categorize sales into 3 groups morning, afternoon, evening
-- find out each of the shift and number of invoices
SELECT 
	branch,
    CASE 
        WHEN CAST(LEFT(time, 2) AS UNSIGNED) < 12 THEN 'Morning'
        WHEN CAST(LEFT(time, 2) AS UNSIGNED) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift_time,
    COUNT(*)
FROM walmart
GROUP BY branch,shift_time
ORDER BY Branch,shift_time, count(*) desc;


-- identify 5 branch with  highest decrease ratio in 
-- revenue compare to last year(current year 2023 and last yer 2022)

SELECT *,
	DATE_FORMAT(STR_TO_DATE(date,'%d/%m/%y'), '%Y') as formated_date
FROM WALMART

-- 2022 SALES
WITH revenue_2022
AS
(SELECT 
	branch,
	SUM(total) as revenue
from walmart
WHERE DATE_FORMAT(STR_TO_DATE(date,'%d/%m/%y'), '%Y') = 2022
group by branch
),

revenue_2023
AS
(
SELECT 
	branch,
	SUM(total) as revenue
from walmart
WHERE DATE_FORMAT(STR_TO_DATE(date,'%d/%m/%y'), '%Y') = 2023
group by branch
)

select 
	lys.branch,
    lys.revenue as last_year_revenue,
	cys.revenue as current_year_revenue,
    ROUND((lys.revenue - cys.revenue) / CAST(lys.revenue AS DECIMAL(10,2)) * 100, 2) AS rev_dec_ratio
from revenue_2022 as lys
join
revenue_2023 as cys
on lys.branch = cys.branch
WHERE 
	lys.revenue > cys.revenue 
ORDER BY rev_dec_ratio desc 
LIMIT 5

-- is there a correlation between quantity purchased and rating
SELECT 
	quantity, round(avg(rating),2) as avg_rating,
    count(*) as no_transacations
    from walmart
    group by quantity
    order by quantity;
    
    -- which payment method gets a higheest average rating
    select payment_method, round(avg(rating),2) as avg_rating
    from walmart
    group by payment_method
    order by avg_rating desc limit 1
    
    
    
    
   
    









