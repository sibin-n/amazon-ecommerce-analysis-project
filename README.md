# Amazon E-commerce Analysis Project Using SQL(MySql)

## Overview
The attached dataset contains ecommerce orders of Amazon from multiple countries in 2023. It is an ecommerce company with a global footprint. In India, the company is headquartered in Hyderabad. The supply chain of the company is very strong. In our dataset, customers are located in 14 countries and they receive orders within a week. The category for which most of these orders were placed is mobile accessories.

There are various ways in which a customer can order. An order can be placed using the company’s app, its website, using Whatsapp and also via other sources (for example: dialing in the 24*7 support helpline number). Customers are divided into 5 categories: A, B, C, D, E. 

The sales team in the company is structured into 5 teams: Alpha, Beta, Gamma, Delta and Epsilon. Every team has multiple Sales Managers and a few Sales POCs, typically 3-4. Every sales POC is given individual targets and the sum of the targets of the sales POCs is the target of the Sales Manager.

## Objective
As an analyst, your goal is to help the company derive insights from the data. These insights will help the company take decisions w.r.t. Orders. Eventually it will help the company increase the number of orders, value of orders and hence revenue.

## Dataset

Dataset is provided by Coding Ninjas

- **Dataset Link:** [Amazon Dataset](https://docs.google.com/spreadsheets/d/1xSh5N5aGsb7MLmpzxF-CuS8nSqlbPDJ9/edit?gid=1905338476#gid=1905338476)

## Schema
![schema](https://github.com/sibin-n/amazon-ecommerce-analysis-project/blob/main/schema.png?raw=true)

## Data Cleaning

```sql

With amazon_dataset2 as (
select o.order_id, o.customer_id, o.customer_country, o.order_datetime, o.order_source, o.sales_poc, o.order_value, c.gender,
c.age, c.category, st.Sales_Manager_First_Name, st.Sales_Manager_Last_Name, st.Sales_Team, st.sales_target from orders o
left join customer c
on c.customer_id=o.customer_id
left join sales_target st
on o.sales_poc=st.sales_poc)
select * from amazon_dataset2;

-- Joined 3 Tables


create view amazon_dataset as
select * from amazon_dataset2;

-- Deleted the Duplicate Column and added view


select * from amazon_dataset;
create table amazon_data as
select o.order_id, o.customer_id, o.customer_country, o.order_datetime, o.order_source, o.sales_poc, o.order_value, c.gender,
c.age, c.category, st.Sales_Manager_First_Name, st.Sales_Manager_Last_Name, st.Sales_Team, st.sales_target from orders o
join customer c
on c.customer_id=o.customer_id
join sales_target st
on o.sales_poc=st.sales_poc;

-- Created a new table


select * from amazon_data; -- DATA--


select order_id,count(*) 
from amazon_data
group by order_id
having count(*)>1;

-- No Duplicate Found


select * FROM amazon_data 
WHERE order_id is null or order_id = ""; -- No Null for Order_id

Select * FROM amazon_data 
WHERE customer_id IS NULL OR customer_id = ""; -- No Null for customer_id

SELECT * FROM amazon_data
WHERE customer_country IS NULL OR customer_country = ""; -- No Null for customer_country

SELECT * FROM amazon_data
WHERE order_datetime IS NULL OR order_datetime = ""; -- No null for order_datetime

SELECT * FROM amazon_data
WHERE order_source IS NULL OR order_source = "";  -- Have Blank Values

SELECT * FROM amazon_data
WHERE sales_poc IS NULL OR sales_poc = "";  -- No null values

SELECT * FROM amazon_data
WHERE order_value IS NULL OR order_value = "";  -- No null values

SELECT * FROM amazon_data
WHERE gender IS NULL OR gender = "";  -- No null values

SELECT * FROM amazon_data
WHERE age IS NULL OR age = ""; -- Have Blank values

SELECT * FROM amazon_data
WHERE category IS NULL OR category = ""; -- No null values

SELECT * FROM amazon_data
WHERE Sales_Manager_First_Name IS NULL OR Sales_Manager_First_Name = ""; -- No null Values

SELECT * FROM amazon_data
WHERE Sales_Manager_Last_Name IS NULL OR Sales_Manager_Last_Name = ""; -- No null values

SELECT * FROM amazon_data
WHERE Sales_Team IS NULL OR Sales_Team = ""; -- No null values

SELECT * FROM amazon_data
WHERE sales_target IS NULL OR sales_target = "";  -- No null values


UPDATE amazon_data
SET order_source = CASE WHEN order_source = "" THEN "Other" ELSE order_source END; -- Blank Value updated in order_source column

SET @avg_age = (SELECT ROUND(AVG(age)) FROM amazon_data);
UPDATE amazon_data
SET age= CASE WHEN age = 0 THEN @avg_age ELSE age END; -- Updated Null Valu in Age 



ALTER TABLE amazon_data
ADD COLUMN sales_manager varchar(200);

UPDATE amazon_data
SET sales_manager = CONCAT(Sales_Manager_First_Name," ",Sales_Manager_Last_Name);   -- Combined First and Last Name --

ALTER TABLE amazon_data
DROP COLUMN Sales_Manager_First_Name;

ALTER TABLE amazon_data
DROP COLUMN Sales_Manager_Last_Name;

ALTER TABLE amazon_data
ADD COLUMN Age_Category VARCHAR(100) AFTER Age;

UPDATE amazon_data
SET age_category = CASE WHEN age<35 THEN "Young"
WHEN age<65 THEN "Middle"
ELSE "Senior" END;                                -- Age Category Added --

ALTER TABLE amazon_data
ADD COLUMN Age_Gender VARCHAR(100) AFTER age_category;

UPDATE amazon_data
SET age_gender = CONCAT(age_category,"-",gender);   -- Age Bracket Created --


ALTER TABLE amazon_data
ADD COLUMN Order_month VARCHAR(15) AFTER order_datetime;

ALTER TABLE amazon_data
ADD COLUMN order_day INT AFTER order_month;

ALTER TABLE amazon_data
ADD COLUMN order_hour INT AFTER order_day;

UPDATE amazon_data
SET order_month = DATE_FORMAT(Order_Datetime, '%M'),
Order_day= EXTRACT(DAY FROM Order_Datetime),
Order_hour= EXTRACT(HOUR FROM Order_Datetime);

-- Created month, day & Hour column from order_datetime
-- -- -- -- -- -- -- -- -- -- -- --


SELECT DISTINCT customer_country FROM amazon_data
ORDER BY customer_country;

SELECT DISTINCT order_source FROM amazon_data
ORDER BY order_source;

SELECT DISTINCT sales_poc FROM amazon_data 
ORDER BY sales_poc;

SELECT DISTINCT gender FROM amazon_data 
ORDER BY gender;

SELECT DISTINCT category FROM amazon_data
ORDER BY category;

SELECT DISTINCT sales_manager FROM amazon_data
ORDER BY sales_manager;

SELECT DISTINCT sales_team FROM amazon_data
ORDER BY sales_team;

-- Vales inside cell are well distributed --




-- OUTLIER OF ORDER VALUE

with per as (
SELECT order_value,
ROUND(PERCENT_RANK() OVER (ORDER BY order_value)*100,2) AS percentile
FROM amazon_data),

Quartile as (
select 
MAX(CASE WHEN percentile<=25 THEN order_value END) Q1,
MAX(CASE WHEN percentile<=75 THEN order_value END) Q3,
MAX(CASE WHEN percentile<=75 THEN order_value END) - MAX(CASE WHEN percentile<=25 THEN order_value END) IQR
FROM per),

Upper_lower_bound as (
SELECT q1-(1.5*iqr) Lower_Bound, Q3+(1.5*iqr) Upper_Bound from quartile)

SELECT * FROM amazon_data
WHERE order_value<0 AND order_value> (SELECT upper_Bound FROM upper_lower_bound); -- No Outlier for order_value --


SELECT COUNT(*) Total_Orders FROM amazon_data;  -- TOTAL ORDERS -- 

SELECT COUNT(DISTINCT customer_id) Total_Customers FROM amazon_data; -- TOTAL CUSTOMERS -- 

SELECT SUM(order_value) Total_Order_Value , ROUND(AVG(order_value),2) Average_Order_Value FROM amazon_data; -- TOTAL AND AVERAGE ORDER VALUE -- 

SELECT COUNT(*)/COUNT(DISTINCT customer_id) Avg_Order_Per_Customer, SUM(order_value)/COUNT(DISTINCT customer_id) Avg_Order_Value_Per_Customer
FROM amazon_data;

-- Avg_Order_Per_Customer is low so increasing it is the main objective --

```

## Analysis

```sql
-- TOTAL ORDERS AND ORDER VALUE BY COUNTRY --
SELECT customer_country, COUNT(order_id) Total_Orders, SUM(order_value) Total_Order_Value
FROM amazon_data
GROUP BY customer_country
ORDER BY Total_Orders DESC;
 

-- TOTAL ORDERS AND ORDER VALUE BY age_gender
SELECT age_gender, COUNT(order_id) Total_Count, SUM(order_value) Total_Order_Value
FROM amazon_data
GROUP BY age_gender
ORDER BY total_count;


-- Orderd in month, day & hour --
SELECT order_month, COUNT(Order_id) FROM amazon_data
GROUP BY order_month
ORDER BY order_month;  

SELECT order_day, COUNT(Order_id) FROM amazon_data
GROUP BY order_day
ORDER BY order_day; 

SELECT order_hour, COUNT(Order_id) FROM amazon_data
GROUP BY order_hour
ORDER BY order_hour; -- Most order was from night time


-- Details of Sales POC by target achieved and not --
WITH Sales_poc_target as (
SELECT sales_poc, SUM(order_value) Total_Order_Value, GROUP_CONCAT(DISTINCT sales_target) Sales_Target
FROM amazon_data
GROUP BY sales_poc
ORDER BY Total_Order_Value DESC)

SELECT *, CASE WHEN Total_Order_Value > Sales_Target THEN "Target Met"
ELSE "Target Not Met" END AS Target_Status
FROM sales_poc_target;                             


-- Detail of Sales Manager by target achieved or not
WITH  Sales_manager_total AS (
SELECT sales_poc,sales_manager, SUM(order_value) Total_Order_Value 
FROM amazon_data
GROUP BY sales_poc,sales_manager
ORDER BY Total_Order_Value DESC),

sales_poc_total as (
SELECT sales_poc, SUM(order_value) Total_Order_Value, GROUP_CONCAT(DISTINCT sales_target) Sales_Target
FROM amazon_data
GROUP BY sales_poc
ORDER BY Total_Order_Value DESC),

Sales_target AS (
SELECT smt.sales_manager,SUM(smt.total_order_value) TotalOrderValue, SUM(spt.sales_target) Target ,
CASE WHEN SUM(smt.total_order_value) > SUM(spt.sales_target) THEN "Target Met"
ELSE "Target Not Met" END AS Target_Status
FROM sales_manager_total smt
JOIN sales_poc_total spt
ON smt.sales_poc=spt.sales_poc
GROUP BY smt.sales_manager
ORDER BY TotalOrderValue DESC)

SELECT target_status, COUNT(*) Count
FROM Sales_target
GROUP BY target_status;


 -- Total Orders & Order Value by Team --
SELECT sales_team ,SUM(order_value) Sum_Of_Order_Value, COUNT(order_id) Total_Orders
FROM amazon_data
GROUP BY sales_team
ORDER BY Sum_Of_Order_Value DESC;


-- Team Performance in Sales Poc --
WITH sales_poc_target_andsum as(
SELECT sales_poc, SUM(order_value) Total_Order_Value, GROUP_CONCAT(DISTINCT sales_target) Sales_Target
FROM amazon_data
GROUP BY sales_poc),

target_of_sales_poc_in_yeam as (
select ad.sales_team, spts.Total_Order_Value, spts.Sales_Target,
CASE WHEN spts.Total_Order_Value > spts.Sales_Target THEN "Target Met"
ELSE "Target Not Met" END AS Target_Status
 from sales_poc_target_andsum spts
JOIN amazon_data ad
on spts.sales_poc=ad.sales_poc
GROUP  BY spts.sales_poc,ad.sales_team
ORDER BY Total_Order_Value DESC)

SELECT sales_team , COUNT(CASE WHEN target_status="Target Met" THEN 1 END) Target_Met,
COUNT(CASE WHEN target_status="Target Not Met" THEN 1 END) Target_Not_Met
FROM target_of_sales_poc_in_yeam
GROUP BY sales_team
ORDER BY Target_Met DESC;


-- Team performance in Sales Manager --
WITH team_by_manager AS (
SELECT sales_team, sales_manager, SUM(order_value) Total_Order_Value 
FROM amazon_data
GROUP BY sales_team, sales_manager),

manager_target as(
SELECT sales_manager, sales_poc, GROUP_CONCAT(Distinct sales_target) Total_Target
FROM amazon_data
GROUP BY sales_manager, sales_poc),

manager_with_target AS (
SELECT sales_manager, SUM(total_target) Target
FROM manager_target
GROUP BY sales_manager),

team_manager_target as (
SELECT tbm.sales_team, tbm.sales_manager,tbm.Total_Order_Value, mwt.target,
CASE WHEN tbm.total_order_value > mwt.target THEN "Target Met"
ELSE "Target Not Met" END AS Target_Status
FROM team_by_manager tbm
JOIN manager_with_target mwt
ON tbm.sales_manager=mwt.sales_manager)

SELECT sales_team, COUNT(CASE WHEN target_status="Target Met" THEN 1 END) Target_Met,
COUNT(CASE WHEN target_status="Target Not Met" THEN 1 END) Target_Not_Met
FROM team_manager_target
GROUP BY sales_team
ORDER BY target_met DESC;
```

## Findings And Conclusion

- The average customer places only 1.72 orders, indicating a relatively low order frequency, with a total of 2,500 orders across 1,453 customers.

- The peak ordering time occurred around midnight, with the busiest hours for purchases being between 10 PM and 2 AM.

- Seniors are the major contributors in purchasing (Total).

- Ireland experienced a concerning sales drought, with zero sales recorded in three consecutive months: January, April, and June.

- The Sales Achievement Rate was 95.8%.

- Performance analysis revealed that the Epsilon team surpassed their sales target by 35%, while Beta and Gamma teams were unable to meet their target which is 500,000+ loss.

- The overall sales performance achieved a 95% target completion rate, with consistency across days and months. Therefore, increasing the sales target could be a promising strategy to drive further growth.
