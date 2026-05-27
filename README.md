A comprehensive data analysis project examining Walmart sales performance across multiple branches in Texas. Using MySQL, this project uncovers actionable business insights related to payment preferences, customer satisfaction, branch performance, and sales patterns.

Key Focus Areas:

Payment method effectiveness

Customer rating patterns

Branch-level performance metrics

Time-based sales analysis

Year-over-year revenue trends

📁 Database Schema
sql
Database: walmart_db
Table: walmart

Columns:
- invoice_id       | Transaction identifier
- Branch          | Store branch code  
- City            | Store location
- category        | Product category
- unit_price      | Price per unit ($)
- quantity        | Units purchased
- date            | Transaction date (DD/MM/YY)
- time            | Transaction time (HH:MM:SS)
- payment_method  | Cash/Credit card/Ewallet
- rating          | Customer rating (1-10)
- profit_margin   | Profit margin (%)
- total           | Calculated as unit_price × quantity
🔍 Business Problems Solved
1. Payment Method Analysis
sql
-- Transaction count and quantity sold by payment method
```
SELECT payment_method, COUNT(*) as no_payments, SUM(quantity) as no_qty_sold
FROM walmart GROUP BY payment_method;
```
Insight: Identifies most popular payment methods and their volume impact

2. Highest Rated Category by Branch
sql
-- Using RANK() window function to find top-performing categories
```
SELECT Branch, category, AVG(rating) as avg_rating
FROM walmart GROUP BY Branch, category
```
-- Ranked per branch to find #1 category
Insight: Reveals which product categories customers love most at each location

3. Busiest Day per Branch
sql
-- Day name extraction and transaction counting
```
DATE_FORMAT(STR_TO_DATE(date, '%d/%m/%y'), '%W') AS day_name
RANK() OVER (PARTITION BY Branch ORDER BY COUNT(*) DESC)
```
Insight: Optimizes staffing and inventory for peak days

4. Rating Statistics by City & Category
sql
```
SELECT City, MIN(rating), MAX(rating), AVG(rating)
FROM walmart GROUP BY City, category
```
Insight: Flags underperforming city-category combinations

5. Profitability Analysis
sql
```
SELECT category, SUM(total) as revenue, SUM(total * profit_margin) as profit
FROM walmart GROUP BY category
```
Insight: Identifies most profitable product categories

6. Preferred Payment Method per Branch
sql
-- Two CTEs: counting transactions → ranking → filtering rank=1
```
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

```
7. Sales Shifts Analysis
sql
```
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
```
Insight: Determines most productive sales hours

8. Year-over-Year Revenue Decline
sql
```
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
```
Insight: Flags branches with concerning downward trends

9. Quantity vs. Rating Correlation
sql
```
SELECT quantity, ROUND(AVG(rating),2) as avg_rating, COUNT(*) as transactions
FROM walmart GROUP BY quantity ORDER BY quantity
```
Insight: Reveals optimal basket sizes for customer satisfaction

10. Highest Rated Payment Method
sql
```
SELECT payment_method, ROUND(AVG(rating),2) as avg_rating
FROM walmart GROUP BY payment_method ORDER BY avg_rating DESC LIMIT 1
```
Insight: Identifies which payment experience delights customers most

📊 Sample Outputs & Insights
Payment Method Performance
payment_method	no_payments	no_qty_sold
Ewallet	310	1,245
Cash	195	892
Credit card	178	743
Top Categories by Branch (Sample)
Branch	Category	avg_rating
A	Health & Beauty	8.7
B	Food & Beverages	8.4
C	Electronic accessories	7.9
Revenue Decline Leaders (Top 3)
Branch	Last Year Revenue	Current Year Revenue	Decline %
WALM013	$45,230	$38,450	15.0%
WALM048	$32,890	$28,120	14.5%
WALM026	$28,760	$24,980	13.1%
🛠️ Technologies Used
Technology	Purpose
MySQL	Data storage, querying, and analysis
Window Functions	RANK() for branch-level rankings
CTEs	Complex multi-step queries
Date Functions	STR_TO_DATE(), DATE_FORMAT()
Case Statements	Time bucketing & categorization
📂 Repository Structure
text
walmart-sales-analysis/
│
├── sql_scripts/
│   ├── 01_database_setup.sql
│   ├── 02_payment_analysis.sql
│   ├── 03_rating_analysis.sql
│   ├── 04_branch_performance.sql
│   ├── 05_profitability.sql
│   └── 06_yoy_decline.sql
│
├── data/
│   └── walmart.csv
│
├── insights/
│   └── key_findings.md
│
├── README.md
└── requirements.txt
🚀 How to Run This Project
Prerequisites
MySQL Server 8.0+

MySQL Workbench (recommended)

Setup Steps
Clone the repository

bash
git clone https://github.com/yourusername/walmart-sales-analysis.git
cd walmart-sales-analysis
Create and use database

sql
CREATE DATABASE walmart_db;
USE walmart_db;
Import data

sql
LOAD DATA INFILE 'path/to/walmart.csv'
INTO TABLE walmart
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
Run analysis queries

sql
-- Execute any query from the sql_scripts/ folder
SOURCE sql_scripts/02_payment_analysis.sql;
📈 Key Business Recommendations
Based on analysis findings:

Payment Infrastructure → Prioritize Ewallet options (highest transaction volume)

Staff Scheduling → Align shifts with busiest days per branch

Category Focus → Promote top-rated categories in underperforming branches

Revenue Recovery → Investigate top 5 declining branches immediately

Basket Optimization → Promote quantity levels with highest ratings

🎓 Skills Demonstrated
✅ Advanced SQL (Window Functions, CTEs, Aggregations)

✅ Date/Time manipulation and formatting

✅ Case-based categorization

✅ Year-over-year trend analysis

✅ Business KPI development

✅ Ranking and filtering complex datasets


