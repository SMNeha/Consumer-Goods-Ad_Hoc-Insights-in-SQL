# SQL-Consumer-Goods-Ad_Hoc-Insights
MySQL Project - Adhoc Analysis

In this project, I explored 10 ad-hoc queries, uncovering important insights about top products, customers, markets, and important metrics like gross price, pre-invoice deductions, manufacturing costs, and sales quantity. 

**Problem Statement:**

AtliQ Hardware is one of the leading computer hardware producers in India and well expanded in other countries too. It's managment recognized a gap in obtaining sufficient insights to support quick and informed decision-making, so they plan to expanf their data analytics team 

**Tools Used -**
- MySQL Workbench (For getting insights using SQL)
- Microsoft Excel (For data visualization)
- Canva (For presentation)
  
**Key Insights from the Ad_hoc Requests :**

- 36.33% of percentage_chg from unique products of 2020 to 2021
- The "Notebook" segment leads the highest product count, with 129 products.
- "Flipkart" leads with the highest average pre-invoice discount percentage at 30.83%
- Year 2020, Quarter 1 had the highest Sold quantity and Q3 the least.
- The "Retailer" channel contributes to 73.22% of gross sales, making it major player

**10 Ad_Hoc Queries to solve:**

#1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

Solution:
```
select distinct market from dim_customer
where customer = "Atliq Exclusive" and region = "APAC";
```
#2. What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields: unique_products_2020, unique_products_2021, percentage_chg.

Solution:
```
with a as (select count(distinct product_code) as products_2020 
        from fact_sales_monthly 
        where fiscal_year = 2020), 
    b as (
		select count(distinct product_code) as products_2021
        from fact_sales_monthly 
        where fiscal_year = 2021)
	select 
		a.products_2020, 
        b.products_2021,
        round((products_2021 - products_2020)*100/(products_2020),2) as percentage_chg 
	from a, b; 
```
#3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields: segment,product_count.

Solution:
```
select segment, 
count(distinct(product_code))as product_count 
from dim_product
group by segment
order by product_count desc;
```
#4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields: segment, product_count_2020, product_count_2021, difference.

Solution:
```
with cte1 as (select 
	p.segment, 
    count(distinct p.product_code) as product_count_2020
from dim_product p
join fact_sales_monthly f 
on p.product_code = f.product_code 
where fiscal_year = 2020 
group by p.segment 
order by product_count_2020 desc), 
cte2 as (select 
	p.segment, 
    count(distinct p.product_code) as product_count_2021
from dim_product p
join fact_sales_monthly f 
on p.product_code = f.product_code 
where fiscal_year = 2021
group by p.segment 
order by product_count_2021 desc) 
select 
	cte1.segment, 
    cte1.product_count_2020,
    cte2.product_count_2021,
    ((cte2.product_count_2021)-(cte1.product_count_2020)) as difference
from cte1, cte2 
where cte1.segment = cte2.segment
group by cte1.segment 
order by difference desc; 

```
#5. Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields: product_code, product, manufacturing_cost.

Solution:
```
with cte1 as (select 
	p.product_code,
    p.product,
    manufacturing_cost 
from dim_product p 
join fact_manufacturing_cost m 
on p.product_code = m.product_code 
where m.manufacturing_cost = (select max(manufacturing_cost) from fact_manufacturing_cost)), 
cte2 as (select 
	p.product_code,
    p.product,
    manufacturing_cost
from dim_product p 
join fact_manufacturing_cost m 
on p.product_code = m.product_code 
where m.manufacturing_cost = (select min(manufacturing_cost) from fact_manufacturing_cost))
select 
	cte1.product_code,
    cte1.product,
    manufacturing_cost 
from cte1 
union 
select 
	cte2.product_code,
    cte2.product,
    manufacturing_cost 
from cte2; 

```
#6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields customer_code, customer, average_discount_percentage.

Solution:
```
select 
	c.customer_code,
    c.customer,
    round(avg(d.pre_invoice_discount_pct)*100,2) as average_discount_percentage
from dim_customer c 
join fact_pre_invoice_deductions d 
on c.customer_code=d.customer_code
where d.fiscal_year = 2021 and c.market = "India" 
group by c.customer_code, c.customer
order by average_discount_percentage desc
limit 5; 
```
#7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns: Month, Year,Gross sales Amount.

Solution:
```
with cte1 as (
	select 
		monthname(s.date) as Month,
        s.fiscal_year as Fiscal_year,
        s.customer_code,
        s.sold_quantity,
        g.gross_price
	from fact_sales_monthly s 
    join fact_gross_price g 
    on s.product_code=g.product_code 
    and s.fiscal_year = g.fiscal_year
    join dim_customer c 
    on c.customer_code = s.customer_code 
    where c.customer = "Atliq Exclusive") 
select 
	Month, 
    Fiscal_year,
    round(sum(sold_quantity * gross_price)) as Gross_sales_amount
from cte1 
group by Month, Fiscal_year 
order by Fiscal_year; 

```
#8. In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity. Quarter, total_sold_quantity.

Solution:
```
with cte1 as (select 
	month(date) as month_no, 
    sum(sold_quantity) as sold_qty 
from fact_sales_monthly 
where fiscal_year = 2020
group by date
order by month_no)
select 
	case 
		when month_no in (9, 10, 11) then "Q1"
        when month_no in (12, 1, 2) then "Q2"
        when month_no in (3, 4, 5) then "Q3"
        when month_no in (6, 7, 8) then "Q4" 
        end as quarter, 
	sum(sold_qty) as sold_qty 
from cte1 
group by quarter 
order by sold_qty desc; 
```
#9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields: channel, gross_sales_mln, percentage.

Solution:
```
with cte1 as (select 
	c.channel,
	round(sum(s.sold_quantity * g.gross_price)/1000000,2) as gross_sales_mln
from fact_sales_monthly s
join fact_gross_price g 
on s.product_code = g.product_code
and s.fiscal_year = g.fiscal_year 
join dim_customer c 
on c.customer_code = s.customer_code
where s.fiscal_year = 2021
group by c.channel
order by gross_sales_mln desc) 
select 
	*, 
    round(gross_sales_mln*100/sum(gross_sales_mln) over(),2) as percentage
from cte1
order by percentage desc; 
```

#10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields: division, product_code, product, total_sold_quantity, rank_order.

Solution:
```
with cte1 as (
select 
	p.division,
    p.product_code,
    p.product,
    sum(s.sold_quantity) as total_sold_quantity
from dim_product p 
join fact_sales_monthly s 
on p.product_code=s.product_code
where s.fiscal_year = 2021 
group by division, product, product_code
order by total_sold_quantity desc),
cte2 as (
select *, 
	dense_rank() over (partition by division order by total_sold_quantity desc) as rank_order
from cte1)
select * from cte2 where rank_order<=3; 
```

LinkedIn: https://www.linkedin.com/feed/update/urn:li:activity:7239893915791740928/

I would like to thank @Dhaval Patel, @Hemanand Vadivel, and the entire @Codebasics team for this wonderful opportunity. 
