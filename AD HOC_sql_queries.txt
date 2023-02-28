SQL Queries

Requests:

1.	Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

		 	select  distinct market from  dim_customer
			where region='apac';



2.	What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields,
unique_products_2020 unique_products_2021 percentage_chg

		select uniqueproducts_2020,uniqueproducts_2021,
		abs(uniqueproducts_2020-uniqueproducts_2021)
		/uniqueproducts_2020*100 as percentage_change
		from (select count(distinct( case when fiscal_year=2020
		then  product_code end)) as uniqueproducts_2020,
                count( distinct (case when fiscal_year = 2021
		then  product_code end)) as uniqueproducts_2021
		from fact_gross_price  )x;


3.	Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields,
segment product_count
		
		select segment,count(distinct product_code) as product_count
 		from dim_product 
 		group by segment
		order by product_count desc;



4.	Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields,
segment product_count_2020 product_count_2021 difference

		select *,(no_of_products2021-no_of_products2020) as difference from 
 		(select  p.segment,count(distinct (case when gp.fiscal_year=2020 
		then p.product_code end)) as no_of_products2020,
 		count(distinct (case when gp.fiscal_year=2021 
		then p.product_code end)) as no_of_products2021 from dim_product p
 		join fact_gross_price gp on gp.product_code=p.product_code  
 		group by p.segment )x


5.	Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields,
product_code product manufacturing_cost

		select x.product_code,p.product,x.manufacturing_cost from 
 		dim_product p join 
 		(select  product_code,manufacturing_cost
 		from fact_manufacturing_cost
 		where manufacturing_cost=(select max(manufacturing_cost) from 
 		fact_manufacturing_cost) 
 		union 
		select  product_code,manufacturing_cost
		from fact_manufacturing_cost
 		where manufacturing_cost=(select min(manufacturing_cost) from 
 		fact_manufacturing_cost) )x 
 		on x.product_code=p.product_code

6.	Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields,
customer_code customer
average_discount_percentage
		
		select c.customer, c.customer_code, x.average from dim_customer c join
		(select customer_code,avg(pre_invoice_discount_pct) as average
		from fact_pre_invoice_deductions
		where fiscal_year=2021
		group by customer_code
		order by average desc)x on
		x.customer_code=c.customer_code
		where c.market='india'
		order by x.average desc limit 5


7.	Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions.
The final report contains these columns:
Month Year
Gross sales Amount

		select x.month,x.year,concat(round(((x.quantity * gp.gross_price)/ 1000000.0 * 1000),2),'M') as gross_sales_amount from
		(select distinct product_code,monthname(date) as month,
		year(date) as year,sum(sold_quantity) as quantity
		from fact_sales_monthly
		where customer_code in(
		select distinct customer_code from dim_customer
		where customer='Atliq Exclusive' )
		group by product_code,month,year)x join
		fact_gross_price gp
		on gp.product_code = x.product_code

8.	In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity,
Quarter total_sold_quantity

		with x as (select month(date) as month ,date,sold_quantity
		from fact_sales_monthly
		where fiscal_year=2020),
		y as (select sold_quantity,
		case  when month between 9 and 11
		then 'Quarter1'
		when month between 3 and 5
		then 'Quarter3'
		when month between 6 and 8
		then 'Quarter4'
		when month between 12 and 2
		then 'Quarter2' end  as Quarter
		from x )
		, z as (select Quarter,sum(sold_quantity) as
		total_sold_quantity from y
		group by Quarter)
		select Quarter,concat(round(total_sold_quantity/1000000,2),'M')
		as sold_quantity from z
		where total_sold_quantity=(select max(total_sold_quantity)
		from z);



9.	Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields,
channel gross_sales_mln percentage



		with x as (select channel,customer_code from dim_customer ) ,
 		y as( select x.channel,sm.product_code,sm.sold_quantity from 
		fact_sales_monthly sm
		join x on x.customer_code = sm.customer_code
 		where sm.fiscal_year=2021),
		z as (select y.channel,
 		(y.sold_quantity* gp.gross_price) as sales_price from 
 		fact_gross_price gp join y on gp.product_code=y.product_code),
 		chnl_sum as ( select z.channel,sum(z.sales_price) as total from z 
  		group by z.channel)  
  		select channel,total,total/
  		sum(total) over(order by channel)* 100 as percentage
  		from chnl_sum
  		order by total desc limit 1




10.	Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields,
division product_code
product total_sold_quantity rank_order



		select division,product_code,product,total_sold_quantity,rnk  from
		(select division,product_code,product,total_sold_quantity,
		rank() over(partition by division order by total_sold_quantity desc) as rnk
		from (select p.product,p.product_code,p.division,sum(sm.sold_quantity)
		as total_sold_quantity from dim_product p join
		fact_sales_monthly sm on
		sm.product_code=p.product_code
		where sm.fiscal_year=2021
		group by p.product_code,p.division,p.product)x)y
		where rnk <=3












