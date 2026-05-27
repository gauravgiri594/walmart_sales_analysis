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

.

🎯 Payment Method Insights
Ewallet is the most popular payment method, handling the highest number of transactions and largest quantity of items sold across all branches.

Credit card receives the highest average customer rating, indicating better customer satisfaction compared to other payment methods.

Cash transactions show the lowest average rating, suggesting customers prefer digital payment experiences.

⭐ Customer Rating Patterns
Health and beauty category consistently receives the highest ratings across multiple branches, particularly in locations like San Antonio and Round Rock.

Purchase quantity of 5 items generates the highest average customer rating, while quantity of 10 items shows declining satisfaction.

Evening transactions (after 6 PM) receive lower average ratings compared to morning and afternoon purchases.

Cities with higher average transaction values also report higher customer ratings, indicating price doesn't negatively impact satisfaction.

🏪 Branch Performance
Branch WALM013 in Irving shows the most diverse category performance, ranking #1 in multiple product categories including Sports & Travel and Fashion accessories.

Tuesday is the busiest day for most branches, requiring maximum staffing and inventory availability.

Saturday shows the lowest transaction volume across all branches, presenting an opportunity for weekend promotion campaigns.

Branch WALM065 in Texas City has the highest single transaction value in the Food & Beverages category.

📉 Revenue & Profitability
Electronic accessories generate the highest total profit, driven by high unit prices and consistent demand.

Five branches experienced revenue decline of over 10% year-over-year, with WALM013 leading at 15% decrease.

Food and beverages category shows the highest profit margin at 0.57, outperforming all other categories.

Home and lifestyle products have the lowest average profit margin at 0.18, despite high unit prices.

⏰ Time-Based Patterns
Morning hours (10 AM - 11 AM) generate the highest average transaction value across all branches.

Afternoon shift (12 PM - 5 PM) accounts for 45% of daily revenue, making it the most productive sales period.

March shows the highest total revenue among all months, while January has the lowest sales volume.

Weekday evenings (Monday-Thursday, 6-8 PM) show surprising sales spikes for Electronic accessories category.

📦 Product Category Analysis
Electronic accessories shows the highest price volatility, with unit prices ranging from 
15
t
o
15to85 across different branches.

High-margin products (0.48+) receive lower average ratings compared to medium-margin products, suggesting price sensitivity.

Sports and travel category shows the strongest rating consistency, with minimal variation across different cities.

Fashion accessories have the lowest average rating at 4.5, indicating significant room for improvement.

🔍 Correlation Findings
There is no strong correlation between quantity purchased and rating, as both high (7-8 items) and low (2-3 items) quantities receive similar satisfaction scores.

Higher profit margins do not guarantee higher customer ratings - some low-margin categories outperform premium products in satisfaction.

Branches with higher transaction frequency tend to have lower average ratings, suggesting volume may impact service quality.

🚨 Critical Alerts
Branch WALM088 in Cleburne shows consistently low ratings (5.8) despite high-margin products, requiring immediate quality review.

Cash payments in Harlingen and San Angelo branches receive ratings below 6.0, significantly lower than other payment methods in same locations.

Food and beverages category in Conroe shows profit margin of 0.57 but rating of 8.6, representing an ideal high-performance segment to replicate.

Three branches (WALM013, WALM026, WALM048) appear in both top performers and revenue decline lists, indicating inconsistent performance management.

💡 Strategic Recommendations
Prioritize Ewallet infrastructure upgrades as it handles highest transaction volume with moderate satisfaction scores.

Launch Tuesday promotional campaigns to maximize revenue on busiest days across all branches.

Investigate revenue decline at WALM013 immediately - despite being a top performer in categories, revenue dropped 15% year-over-year.

Optimize basket sizes around 5-item purchases since this quantity generates highest customer ratings.

Expand high-margin Food & Beverages success from Conroe branch to underperforming locations.

Consider evening staff training program as ratings drop significantly after 6 PM across all branches.

Review Electronic accessories pricing strategy - high volatility may be confusing customers and impacting ratings.

Implement morning shift incentives to capitalize on highest transaction values between 10-11 AM.

📈 Performance Summary
Metric	Best Performing	Worst Performing
Payment Method	Credit Card (7.5 r

🎓 Skills Demonstrated
✅ Advanced SQL (Window Functions, CTEs, Aggregations)

✅ Date/Time manipulation and formatting

✅ Case-based categorization

✅ Year-over-year trend analysis

✅ Business KPI development

✅ Ranking and filtering complex datasets


