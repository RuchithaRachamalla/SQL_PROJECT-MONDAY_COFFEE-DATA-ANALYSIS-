# Monday Coffee Expansion SQL Project

![Company Logo](https://github.com/RuchithaRachamalla/SQL_PROJECT-MONDAY_COFFEE-DATA-ANALYSIS-/blob/main/1.png)

## Objective
The goal of this project is to analyze the sales data of Monday Coffee, a company that has been selling its products online since January 2023, and to recommend the top three major cities in India for opening new coffee shop locations based on consumer demand and sales performance.

## Schemas

```sql

/* CREATING DATABASE  */

CREATE DATABASE MONDAY_COFFEE

/* CREATING CITY TABLE */

DROP TABLE IF EXISTS CITY
CREATE TABLE CITY
(
	CITY_ID	INT PRIMARY KEY,
	CITY_NAME VARCHAR(15),	
	POPULATION	BIGINT,
	ESTIMATED_RENT	FLOAT,
	CITY_RANK INT
)

/* CREATING CUSTOMERS TABLE */

DROP TABLE IF EXISTS CUSTOMERS
CREATE TABLE CUSTOMERS
(
	CUSTOMER_ID INT PRIMARY KEY,	
	CUSTOMER_NAME VARCHAR(25),	
	CITY_ID INT,
	FOREIGN KEY (CITY_ID) REFERENCES CITY(CITY_ID)
)

/* CREATING PRODUCTS TABLE */

DROP TABLE IF EXISTS PRODUCTS
CREATE TABLE PRODUCTS
(
	PRODUCT_ID	INT PRIMARY KEY,
	PRODUCT_NAME VARCHAR(35),	
	PRICE FLOAT
)

/* CREATING SALES TABLE */

DROP TABLE IF EXISTS SALES
CREATE TABLE SALES
(
	SALE_ID	INT PRIMARY KEY,
	SALE_DATE DATE,
	PRODUCT_ID	INT,
	CUSTOMER_ID	INT,
	TOTAL FLOAT,
	RATING INT,
	FOREIGN KEY (PRODUCT_ID) REFERENCES PRODUCTS(PRODUCT_ID),
	FOREIGN KEY (CUSTOMER_ID) REFERENCES CUSTOMERS(CUSTOMER_ID) 
)

--IMPORT DATASETS RULES
--1ST IMPORT TO CITY
--2ND IMPORT TO PRODUCTS
--3RD IMPORT TO CUSTOMERS
--4TH IMPORT TO SALES

```
## Key Questions and answers

**MONDAY-COFFEE DATA ANALYSIS**

```sql
SELECT * FROM CITY
SELECT * FROM PRODUCTS
SELECT * FROM CUSTOMERS
SELECT * FROM SALES
``` 

**REPORTS AND DATA ANALYSIS**

**Q.1 Coffee Consumers Count**
**How many people in each city are estimated to consume coffee, given that 25% of the population does?**

```sql
SELECT 
      CITY_NAME,
	  ROUND((POPULATION * 0.25)/1000000,2) AS COFFEE_CONSUMERS_IN_MILLIONS,
	  CITY_RANK
	  FROM CITY
ORDER BY 2 DESC
```

**Q.2 Total Revenue from Coffee Sales**
**What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?**

```sql
SELECT 
	SUM(TOTAL) as TOTAL_REVENUE
FROM SALES
WHERE 
	EXTRACT(YEAR FROM SALE_DATE)  = 2023
	AND
	EXTRACT(QUARTER FROM SALE_DATE) = 4

SELECT 
	C.CITY_NAME,
	SUM(S.TOTAL) AS TOTAL_REVENUE
FROM SALES AS S
JOIN CUSTOMERS AS C1
ON S.CUSTOMER_ID = C1.CUSTOMER_ID
JOIN CITY AS C
ON C.CITY_ID = C1.CITY_ID
WHERE 
	EXTRACT(YEAR FROM S.SALE_DATE)  = 2023
	AND
	EXTRACT(QUARTER FROM S.SALE_DATE) = 4
GROUP BY 1
ORDER BY 2 DESC
```

**Q.3 Sales Count for Each Product**
**How many units of each coffee product have been sold?**

```sql
SELECT 
      P.PRODUCT_NAME,
	  COUNT(S.SALE_ID) AS TOTAL_ORDERS
FROM PRODUCTS AS P
LEFT JOIN SALES AS S
ON S.PRODUCT_ID=P.PRODUCT_ID
GROUP BY 1
ORDER BY 2 DESC
```

**Q.4 Average Sales Amount per City**
**What is the average sales amount per customer in each city?**

```sql
-- city abd total sale
-- no cx in each these city

SELECT 
	CI.CITY_NAME,
	SUM(S.TOTAL) as TOTAL_REVENUE,
	COUNT(DISTINCT S.CUSTOMER_ID) as TOTAL_CUSTOMERS,
	ROUND(
			SUM(S.TOTAL)::NUMERIC/
				COUNT(DISTINCT S.CUSTOMER_ID)::NUMERIC
			,2) as AVG_SALES_PER_CUSTOMER
	
FROM SALES AS S
JOIN CUSTOMERS AS C
ON S.CUSTOMER_ID = C.CUSTOMER_ID
JOIN CITY AS CI
ON CI.CITY_ID = C.CITY_ID
GROUP BY 1
ORDER BY 2 DESC
```

**Q.5 City Population and Coffee Consumers (25%)**
**Provide a list of cities along with their populations and estimated coffee consumers.**
**return city_name, total current cx, estimated coffee consumers (25%)**

```sql
WITH CITY_TABLE AS
(
    SELECT
        CITY_NAME,
        ROUND((POPULATION * 0.25)/1000000, 2) AS COFFEE_CONSUMERS
    FROM CITY
),
CUSTOMERS_TABLE AS
(
    SELECT
        CI.CITY_NAME,
        COUNT(DISTINCT C.CUSTOMER_ID) AS UNIQUE_CX
    FROM SALES AS S
    JOIN CUSTOMERS AS C
    ON C.CUSTOMER_ID = S.CUSTOMER_ID
    JOIN CITY AS CI
    ON CI.CITY_ID = C.CITY_ID
    GROUP BY 1
)
SELECT
    CUSTOMERS_TABLE.CITY_NAME,
    CITY_TABLE.COFFEE_CONSUMERS AS COFFEE_CONSUMER_IN_MILLIONS,
    CUSTOMERS_TABLE.UNIQUE_CX
FROM CITY_TABLE
JOIN
CUSTOMERS_TABLE
ON CITY_TABLE.CITY_NAME = CUSTOMERS_TABLE.CITY_NAME
```

**Q6 Top Selling Products by City**
**What are the top 3 selling products in each city based on sales volume?**

```sql
SELECT *
FROM (
    SELECT
        CI.CITY_NAME,
        P.PRODUCT_NAME,
        COUNT(S.SALE_ID) AS TOTAL_ORDERS,
        DENSE_RANK() OVER (PARTITION BY CI.CITY_NAME ORDER BY COUNT(S.SALE_ID) DESC) AS RANK
    FROM SALES AS S
    JOIN PRODUCTS AS P
    ON S.PRODUCT_ID = P.PRODUCT_ID
    JOIN CUSTOMERS AS C
    ON C.CUSTOMER_ID = S.CUSTOMER_ID
    JOIN CITY AS CI
    ON CI.CITY_ID = C.CITY_ID
    GROUP BY 1, 2
) AS T1
WHERE RANK <= 3
```

**Q.7 Customer Segmentation by City**
**How many unique customers are there in each city who have purchased coffee products**

```sql
SELECT
    CI.CITY_NAME,
    COUNT(DISTINCT C.CUSTOMER_ID) AS UNIQUE_CX
FROM CITY AS CI
LEFT JOIN CUSTOMERS AS C
ON C.CITY_ID = CI.CITY_ID
JOIN SALES AS S
ON S.CUSTOMER_ID = C.CUSTOMER_ID
WHERE
    S.PRODUCT_ID IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14)
GROUP BY 1
```

**Q.8 Average Sale vs Rent**
**Find each city and their average sale per customer and avg rent per customer**
**Conclusions**

```sql
WITH 
CITY_TABLE 
AS
(
    SELECT
        CI.CITY_NAME,
        SUM(S.TOTAL) AS TOTAL_REVENUE,
        COUNT(DISTINCT S.CUSTOMER_ID) AS TOTAL_CX,
        ROUND(
            SUM(S.TOTAL)::NUMERIC / COUNT(DISTINCT S.CUSTOMER_ID)::NUMERIC,2)
        AS AVG_SALE_PR_CX
    FROM SALES AS S
    JOIN CUSTOMERS AS C
    ON S.CUSTOMER_ID = C.CUSTOMER_ID
    JOIN CITY AS CI
    ON CI.CITY_ID = C.CITY_ID
    GROUP BY 1
    ORDER BY 2 DESC
),
CITY_RENT AS (
    SELECT
        CITY_NAME,
        ESTIMATED_RENT
    FROM CITY
)
SELECT
    CR.CITY_NAME,
    CR.ESTIMATED_RENT,
    CT.TOTAL_CX,
    CT.AVG_SALE_PR_CX,
    ROUND(
        CR.ESTIMATED_RENT::NUMERIC / CT.TOTAL_CX::NUMERIC,2)
    AS AVG_RENT_PER_CX
FROM CITY_RENT AS CR
JOIN CITY_TABLE AS CT
ON CR.CITY_NAME = CT.CITY_NAME
ORDER BY 4 DESC
```

**Q.9 Monthly Sales Growth**
**Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly)**
**by each city**

```sql
WITH 
MONTHLY_SALES
AS 
(
    SELECT
        CI.CITY_NAME,
        EXTRACT(MONTH FROM SALE_DATE) AS MONTH,
        EXTRACT(YEAR FROM SALE_DATE) AS YEAR,
        SUM(S.TOTAL) AS TOTAL_SALE
    FROM SALES AS S
    JOIN CUSTOMERS AS C
    ON C.CUSTOMER_ID = S.CUSTOMER_ID
    JOIN CITY AS CI
    ON CI.CITY_ID = C.CITY_ID
    GROUP BY 1, 2, 3
    ORDER BY 1, 3, 2
),
GROWTH_RATE
AS 
(
    SELECT
        CITY_NAME,
        MONTH,
        YEAR,
        TOTAL_SALE AS CR_MONTH_SALE,
        LAG(TOTAL_SALE, 1) OVER (PARTITION BY CITY_NAME ORDER BY YEAR, MONTH) AS LAST_MONTH_SALE
    FROM MONTHLY_SALES
)
SELECT
    CITY_NAME,
    MONTH,
    YEAR,
    CR_MONTH_SALE,
    LAST_MONTH_SALE,
    ROUND(
        (CR_MONTH_SALE - LAST_MONTH_SALE)::NUMERIC / LAST_MONTH_SALE::NUMERIC * 100,2)
    AS GROWTH_RATIO
FROM GROWTH_RATE
WHERE
    LAST_MONTH_SALE IS NOT NULL
```

**Q.10 Market Potential Analysis**
**Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated coffee consumer**

```sql
WITH 
CITY_TABLE
AS 
(
    SELECT
        CI.CITY_NAME,
        SUM(S.TOTAL) AS TOTAL_REVENUE,
        COUNT(DISTINCT S.CUSTOMER_ID) AS TOTAL_CX,
        ROUND(
            SUM(S.TOTAL)::NUMERIC / COUNT(DISTINCT S.CUSTOMER_ID)::NUMERIC,2)
        AS AVG_SALE_PR_CX
    FROM SALES AS S
    JOIN CUSTOMERS AS C
    ON S.CUSTOMER_ID = C.CUSTOMER_ID
    JOIN CITY AS CI
    ON CI.CITY_ID = C.CITY_ID
    GROUP BY 1
    ORDER BY 2 DESC
),
CITY_RENT AS (
    SELECT
        CITY_NAME,
        ESTIMATED_RENT,
        ROUND((POPULATION * 0.25) / 1000000, 3) AS ESTIMATED_COFFEE_CONSUMER_IN_MILLIONS
    FROM CITY
)
SELECT
    CR.CITY_NAME,
    TOTAL_REVENUE,
    CR.ESTIMATED_RENT AS TOTAL_RENT,
    CT.TOTAL_CX,
    ESTIMATED_COFFEE_CONSUMER_IN_MILLIONS,
    CT.AVG_SALE_PR_CX,
    ROUND(
        CR.ESTIMATED_RENT::NUMERIC / CT.TOTAL_CX::NUMERIC,2)
    AS AVG_RENT_PER_CX
FROM CITY_RENT AS CR
JOIN CITY_TABLE AS CT
ON CR.CITY_NAME = CT.CITY_NAME
ORDER BY 2 DESC
```   

## Recommendations
After analyzing the data, the recommended top three cities for new store openings are:

**City 1: Pune**  
1. Average rent per customer is very low.  
2. Highest total revenue.  
3. Average sales per customer is also high.

**City 2: Delhi**  
1. Highest estimated coffee consumers at 7.7 million.  
2. Highest total number of customers, which is 68.  
3. Average rent per customer is 330 (still under 500).

**City 3: Jaipur**  
1. Highest number of customers, which is 69.  
2. Average rent per customer is very low at 156.  
3. Average sales per customer is better at 11.6k.

## Author - Ruchitha Rachamalla 

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

Thank you for your interest in this project!
