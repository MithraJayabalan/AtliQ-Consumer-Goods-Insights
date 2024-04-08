# AtliQ-Consumer-Goods-Insights
### 📊 Elevating Consumer Goods Insights: Unveiling the Power of SQL and Data Visualization 🛍️
####  Sharing my recent project in the consumer goods domain, where I delved deep into SQL querying, advanced analytics, and dynamic data visualization techniques! Here's a glimpse into the journey:
#### 🔍 Project Overview: Used MySQL for data querying, Power BI for visualization, and Power Point for presentation to address management's ad hoc requests.
#### 💡 Knowledge Gained: Enhanced SQL skills including subqueries, CTEs, JOIN types, and window functions like ROW_NUMBER, RANK, and DENSE_RANK.
#### 📊 Insights Delivered: Uncovered unique product insights, identified top customers, analyzed discount percentages, computed sales amounts, and evaluated total quantities sold.
#### 🚀 Impact and Learnings: Showcased the power of SQL in extracting insights and emphasized the importance of data visualization in communicating findings effectively.
#### 💬 Key Takeaways: Integrating various tools is crucial for data analysis, emphasizing continuous learning in the evolving data landscape.

## Adhoc Requests
#### 1) Provide the list of markets in which customer "Atliq Exclusive" operates itsbusiness in the APAC region
```
select  distinct(market),customer,region 
from gdb023.dim_customer
where region = "APAC" and customer = "Atliq Exclusive";
```
#### 2) What is the percentage of unique product increase in 2021 vs. 2020? Thefinal output contains these fields, unique_products_2020, unique_products_2021, percentage_chg
```
with unique_product_count as (
		select count(distinct case when fiscal_year = "2020" then product_code end) as unique_product_2020,
				count(distinct case when fiscal_year = "2021" then product_code end) as unique_product_2021
        from gdb023.fact_sales_monthly
        )
    select unique_product_2020,
          unique_product_2021,
          concat(round(((unique_product_2021 - unique_product_2020)/unique_product_2020)*100,2), '%') as percentage_chg
    from unique_product_count;
```
#### 3) Provide a report with all the unique product counts for each segment andsort them in descending order of product counts. The final output contains2 fields,segmentproduct_count
```
select segment, count(distinct product_code) product_count
from gdb023.dim_product
group by segment
order by product_count desc;
```
#### 4) Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields, segment, product_count_2020, product_count_2021,difference
```
with uniqueproductcount as (select  segment,
		count(distinct case when fiscal_year = "2020" then s.product_code end) as product_count_2020,
        count(distinct case when fiscal_year = "2021" then s.product_code end) as product_count_2021
        from gdb023.fact_sales_monthly s
        inner join gdb023.dim_product p
        on s.product_code = p.product_code
        group by segment
        )
select segment, product_count_2020, product_count_2021,
(product_count_2021 - product_count_2020) as difference
from uniqueproductcount
order by difference desc;
```
#### 5) Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields,product_code, product, manufacturing_cost
```
select p.product_code, p.product, concat("$",round(manufacturing_cost,2)) as manufacturing_cost
from gdb023.dim_product p
inner join gdb023.fact_manufacturing_cost m
on p.product_code = m.product_code
where manufacturing_cost = (select max(manufacturing_cost)
from gdb023.fact_manufacturing_cost) 
or manufacturing_cost = (select min(manufacturing_cost)
from gdb023.fact_manufacturing_cost)
order by manufacturing_cost desc;
```
#### 6) Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields,customer_code, customer, average_discount_percentage
```
select d.customer_code, c.customer, concat(round(avg(pre_invoice_discount_pct)*100,2),"%") as Avg_discount_percentage
from gdb023.fact_pre_invoice_deductions d
inner join gdb023.dim_customer c
on d.customer_code = c.customer_code
where market = "India" and
 fiscal_year = "2021"
 group by customer, customer_code
 order by avg(pre_invoice_discount_pct) desc
 limit 5;
```
#### 7) Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns: Month, Year, Gross sales Amount
```
select monthname(date) as month, 
P.fiscal_year,
 concat("$",round(sum(sold_quantity * gross_price)/1000000,2)) as gross_sales_amt_millions
from gdb023.fact_sales_monthly s
inner join gdb023.fact_gross_price p
on p.product_code = s.product_code
and p.fiscal_year = s.fiscal_year
inner join gdb023.dim_customer c
on c.customer_code = s.customer_code
where c.customer = "Atliq Exclusive"
group by month, fiscal_year
order by fiscal_year;
```
#### 8) In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity, Quarter, total_sold_quantity
```
with quarterly_sales as (
select 
	date,
    month(date) as Month,
	case
	 when month(date) between  9 and 11 then 'Q1'
     when month(date) between  12 and 2 then 'Q2'
     when month(date) between  3 and 5 then 'Q3'
     when month(date) between 6 and 8 then 'Q4'
    end as Quarter,
    sold_quantity
    from gdb023.fact_sales_monthly
    where fiscal_year = 2020
    )
    select
    Quarter, 
    sum(sold_quantity) as total_sold_quantity
    from quarterly_sales
    group by quarter
    order by total_sold_quantity desc;
```
#### 9) Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields, channel, gross_sales_mln, percentage
```
with channel_sales as(
select c.channel as channel,
round(sum(p.gross_price*s.sold_quantity)/1000000,2) as gross_sales_mln
from gdb023.fact_sales_monthly s
join gdb023.dim_customer c
on s.customer_code = c.customer_code
join gdb023.fact_gross_price p
on s.product_code = p.product_code
where s.fiscal_year = 2021
group by channel
)
select
channel,
 gross_sales_mln,
concat(round(gross_sales_mln/(select sum(gross_sales_mln) from channel_sales)*100,2),'%') as percentage
from channel_sales
order by gross_sales_mln desc;
```
#### 10) Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields, division, product_code
```
with cte as
(select division, product_code, product, sum(sold_quantity) as total_sold_quantity,
rank() over(partition by division order by sum(sold_quantity) desc) as rank_order
from dim_product p
join fact_sales_monthly s
using (product_code)
where fiscal_year = 2021
group by product_code, product, division
order by total_sold_quantity desc)
select division, product_code, product, total_sold_quantity, rank_order
from cte
where rank_order <=3;
```
### Thank You Dhaval Patel and Hemanand Vadivel and Codebasics team for this opportunity.
