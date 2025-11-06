# Apple’s global retail and product performance project.

![Apple Logo](https://github.com/NicolaeAm/Apple_SQL_Project_2025/blob/main/apple.webp)

## Overview 

Project Overview
This project is an SQL-based analysis of Apple’s global retail and product performance. Built on a custom relational database with over 1 million sales records, it examines key business metrics including store performance, product category trends, warranty claims, and seasonal sales dynamics.

## Objective

The main goal of this project is to analyze Apple’s sales and warranty data to uncover key insights about:
  -Revenue distribution across countries and stores
  -Product category performance and demand trends
  -Warranty claim patterns and reliability metrics
  -Store growth ratios and performance benchmarking
  -Seasonality and correlation analysis for sales optimization

## Dataset

The dataset used in this project represents Apple sales environment, containing over one million transaction records.
It models a realistic business structure with five relational tables that store data about Apple’s stores, products, categories, sales, and warranty operations.

## Dataset schema

```sql
CREATE TABLE stores(
    store_id VARCHAR(5) PRIMARY KEY,
    store_name VARCHAR(30),
    city VARCHAR(25),
    country VARCHAR(25)
);

CREATE TABLE category(
    category_id VARCHAR(10) PRIMARY KEY,
    category_name VARCHAR(20)
);

CREATE TABLE products(
    product_id VARCHAR(10) PRIMARY KEY,
    product_name VARCHAR(35),
    category_id VARCHAR(10),
    launch_date DATE,
    price FLOAT,
    CONSTRAINT fk_category FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE sales(
    sale_id VARCHAR(15) PRIMARY KEY,
    sale_date DATE,
    store_id VARCHAR(10),
    product_id VARCHAR(10),
    quantity INT,
    CONSTRAINT fk_store FOREIGN KEY (store_id) REFERENCES stores(store_id),
    CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE warranty(
    claim_id VARCHAR(10) PRIMARY KEY,
    claim_date DATE,
    sale_id VARCHAR(15),
    repair_status VARCHAR(15),
    CONSTRAINT fk_orders FOREIGN KEY (sale_id) REFERENCES sales(sale_id)
);
```

## Data Exploration & Optimization
 - Before running analytics, Exploratory Data Analysis (EDA) and performance tuning were performed.
```sql
SELECT * FROM category;
SELECT * FROM products;
SELECT * FROM sales;
SELECT * FROM stores;
SELECT * FROM warranty;

SELECT DISTINCT repair_status FROM warranty;
SELECT COUNT(*) FROM sales;
```
 - Creating indexes on frequently filtered columns improved query execution time by over 90%.

```sql
EXPLAIN ANALYZE
SELECT * FROM sales
WHERE product_id = 'P-44';

CREATE INDEX sales_product_id ON sales(product_id);

EXPLAIN ANALYZE
SELECT * FROM sales
WHERE product_id = 'ST-31';

CREATE INDEX sales_store_id ON sales(store_id);
```

  # Business Problems & SQL Solutions
## 1. Find the number of stores in each country.
```sql
SELECT
count(*) AS total_stores,
country
FROM stores
GROUP BY country
ORDER BY 1 DESC
;
```
## 2. Calculate the total number of units sold by each store.
```sql
SELECT 
	sl.store_id,
	st.store_name,
	SUM(sl.quantity) as total_units_sold
FROM 
	sales as sl
JOIN
	stores as st
	ON st.store_id = sl.store_id
GROUP BY 1, 2
ORDER BY 3 DESC
;
```
## 3. Identify how many sales occurred in December 2023.
```sql
SELECT 
	count(sale_id) as  total_sale_dec
FROM sales
WHERE TO_CHAR(sale_date, 'MM-YYYY') = '12-2023'
;
```
## 4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT 
	COUNT(*)
FROM stores
	WHERE store_id NOT IN (
						SELECT
							DISTINCT store_id
						FROM sales AS sl 
						RIGHT JOIN warranty as wr
						ON sl.sale_id = wr.sale_id)
						;
```
## 5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
SELECT 
	ROUND
	(COUNT(claim_id)/(SELECT COUNT(*)FROM warranty)::numeric 
	* 100, 2) AS warranty_precentage

FROM warranty
WHERE repair_status LIKE '%Warranty Void%';
```

## 6. Identify which store had the highest total units sold in the last year.
```sql
SELECT 
	s.store_id,
	st.store_name,
	SUM(s.quantity) as total_unit_sales
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id
WHERE sale_date >= (CURRENT_DATE - INTERVAL '2 year')
GROUP BY 1, 2
ORDER BY 3 DESC;

select current_date - Interval '2 year'
```
## 7. Count the number of unique products sold in the last year.
```sql
SELECT
	COUNT(DISTINCT product_id) AS total_product_sold
FROM sales
WHERE sale_date >= (CURRENT_DATE - INTERVAL '2 year') 
;
``` 	
## 8. Find the average price of products in each category.
```sql
SELECT
	p.category_id,
	c.category_name,
	AVG(p.price) AS avg_price
FROM products AS p
JOIN
category AS c
ON p.category_id = c.category_id
GROUP BY 1, 2
ORDER BY 3 DESC
;
```
## 9. How many warranty claims were filed in 2020?
```sql
SELECT 
	COUNT(*) AS warranty_claim 
FROM warranty
WHERE (EXTRACT (YEAR FROM claim_date)) = 2020;
```
## 10. For each store, identify the best-selling day based on the highest quantity sold.
```sql
SELECT *
FROM
(SELECT 
	store_id,
	SUM(quantity) AS total_units_sold,
	TO_CHAR (sale_date, 'Day') as day_name,
	RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS rating
FROM sales
GROUP BY 1, 3
)as r
WHERE rating = 1
;
```
## 11. Identify the least selling product in each country for each year based on total units sold.
```sql
WITH product_rank
AS 
(
SELECT 
	st.country,
	p.product_name,
	SUM(s.quantity) AS total_qty_sold,
	RANK() OVER(PARTITION BY st.country ORDER BY SUM(s.quantity)) as rating
FROM sales AS s
JOIN 
stores as st
ON s.store_id = st.store_id
JOIN 
products AS p
ON s.product_id = p.product_id
GROUP BY 1, 2
)
SELECT * 
FROM product_rank
WHERE rating = 1
;
```
## 12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT  
	COUNT(*)
	--w.*,
	--s.sale_date,
	--w.claim_date - sale_date
FROM warranty AS w 
LEFT JOIN 
sales AS s
ON s.sale_id = w.sale_id
WHERE 
	w.claim_date - sale_date <= 180
;
```
## 13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT  
	p.product_name,
	COUNT(DISTINCT w.claim_id) AS nr_claim,
	COUNT(s.sale_id)
FROM warranty AS w 
RIGHT JOIN 
sales AS s
ON s.sale_id = w.sale_id
JOIN products AS p
ON p.product_id = s.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY 1
HAVING COUNT(w.claim_id) > 0
;
```
## 14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT * FROM
(
SELECT 
	TO_CHAR (sale_date, 'MM-YYYY') AS month,
	SUM(s.quantity) AS total_unit_sold
FROM sales AS s
JOIN 
stores AS st
ON s.store_id = st.store_id
WHERE 
	st.country = 'USA'
	AND
	s.sale_date >= CURRENT_DATE - INTERVAL '3 year'
GROUP BY 1
) AS ta
WHERE total_unit_sold >= 5000
;
```

## 15. Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT 
 	c.category_name, 
	COUNT(w.claim_id) AS total_claims
FROM warranty AS w 
LEFT JOIN 
sales AS s 
ON w.sale_id = s.sale_id
JOIN 
products AS p
ON p.product_id = s.product_id
JOIN 
category AS c
ON c.category_id = p.category_id
WHERE 
	w.claim_date >= CURRENT_DATE - INTERVAL '3.1 year'
GROUP BY 1
ORDER BY 2 DESC
;
```

## 16.Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT 
	country,
	total_unit_sold,
	total_claim,
	COALESCE(total_claim::numeric/total_unit_sold::numeric * 100, 0) 
	AS porc_of_risk
FROM
(
SELECT 
	st.country,
	SUM(s.quantity) AS total_unit_sold,
	COUNT(w.claim_id) AS total_claim
	
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id
LEFT JOIN
warranty AS w
ON w.sale_id = s.sale_id
GROUP BY 1
) AS t1
ORDER BY 4 DESC
;
```
## 17 Analyze the year-by-year growth ratio for each store
```sql
WITH yearly_sales
AS
(
	SELECT 
		s.store_id,
		st.store_name,
		EXTRACT (YEAR FROM sale_date) as year,
		SUM(s.quantity * p.price) AS total_sales
	FROM sales AS S
	JOIN products AS p
	ON s.product_id = p.product_id
	JOIN stores AS st
	ON st.store_id = s.store_id
	GROUP BY 1, 2, 3
	ORDER BY 2, 3
),
growth_ratio
AS
(
SELECT 
	store_name,
	year,
	LAG(total_sales, 1) OVER(PARTITION BY store_name ORDER BY year) AS last_year_sale,
	total_sales as current_year_sale
FROM yearly_sales
)
SELECT 
	store_name,
	year,
	last_year_sale,
	current_year_sale,
	ROUND(
	(current_year_sale - last_year_sale)::numeric/last_year_sale::numeric * 100
	,2) growth_ratio
FROM growth_ratio 	
WHERE last_year_sale IS NOT NULL
	AND 
	YEAR <> EXTRACT(YEAR FROM CURRENT_DATE)
;
```

## 18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
SELECT
	CASE 
		WHEN p.price < 500 THEN 'Low Expensive'
		WHEN p.price BETWEEN 500 AND 1000 THEN 'Mid Expensive'
		WHEN p.price BETWEEN 1000 AND 2000 THEN 'High Expensive'
		ELSE 'Premium'
		END AS price_segment,
	COUNT(w.claim_id) AS total_claims
FROM warranty AS w
LEFT JOIN 
sales AS s
ON w.sale_id = s.sale_id
JOIN 
products AS p
ON p.product_id = s.product_id
WHERE claim_date >= (CURRENT_DATE - INTERVAL '6.1 year')
GROUP BY 1
;
```
## 19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
WITH paid_repair
AS
	(SELECT
		s.store_id,
		COUNT(w.claim_id) AS paid_repaired
	FROM warranty AS w
	JOIN sales AS s
	ON w.sale_id = s.sale_id
	WHERE w.repair_status = 'Paid Repaired'
	GROUP BY 1
	),
total_repaired

AS
	(SELECT
		s.store_id,
		COUNT(w.claim_id) AS total_repaired
	FROM warranty AS w
	JOIN sales AS s
	ON w.sale_id = s.sale_id
	GROUP BY 1)

SELECT
	tr.store_id,
	st.store_name,
	pr.paid_repaired,
	tr.total_repaired, 
	ROUND(
	pr.paid_repaired::numeric/tr.total_repaired::numeric * 100,
	2) AS precen_paid_repaired
FROM paid_repair as pr
JOIN
total_repaired tr
ON pr.store_id = tr.store_id
JOIN stores AS st
ON tr.store_id = st.store_id
;
```

## 20.Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
WITH monthly_sales
AS
	(SELECT 
		store_id,
		EXTRACT (YEAR FROM sale_date) AS Year_,
		EXTRACT (MONTH FROM sale_date) AS Month_,
		--TO_CHAR(sale_date, 'Month') as Month_,
		SUM(p.price * s.quantity) AS total_reven
	FROM sales as s
	JOIN
	products as p
	ON s.product_id = p.product_id
	GROUP BY 1,2,3
	ORDER BY 1,2, 3
	)
select
	store_id,
	Year_,
	Month_,
	total_reven,
	SUM(total_reven) OVER(PARTITION BY store_id order by Year_, Month_) AS running_tot
FROM monthly_sales
;
```
## 21. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
``sql
SELECT 
	p.product_name,
	CASE 
		WHEN s.sale_date BETWEEN p.launch_date AND p.launch_date + INTERVAL '6 month' 
		THEN '0-6 month'
		WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '6 month' and
		p.launch_date + INTERVAL '12 month' THEN '6-12 month'
		WHEN s.sale_date BETWEEN p.launch_date + INTERVAL '12 month' and
		p.launch_date + INTERVAL '18 month' THEN '12-18 month'
		ELSE '18+ months'
	END as plc,
	SUM(s.quantity) AS total_qt_sale
FROM sales as s
JOIN products as p
ON s.product_id = p.product_id
GROUP BY  1, 2
order by 1, 3 desc
;
```









