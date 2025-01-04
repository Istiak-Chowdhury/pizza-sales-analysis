# Pizza Sales Analysis SQL Queries

This README contains a set of SQL queries written to analyze pizza sales data. The dataset includes information about orders, order details, and pizza attributes such as size and price. Below, we describe each query's purpose and its output.

---

## CS1: Extract Only the Table Information
**Query:**
```sql
describe orders;
describe order_details;
```
**Purpose:** Provides the schema of the `orders` and `order_details` tables, including column names, data types, and constraints.

**Output:** Table structure with columns, types, and constraints.

---

## CS2: Total Revenue from All Orders
**Query:**
```sql
WITH CTE AS (SELECT 
        ORDER_ID, OD.PIZZA_ID, QUANTITY, PRICE, (QUANTITY * PRICE) AS TOTAL_PRICE
FROM 
    ORDER_DETAILS OD
LEFT JOIN PIZZAS P ON OD.PIZZA_ID = P.PIZZA_ID ) 
SELECT 
        ROUND(SUM(TOTAL_PRICE),2) AS TOTAL_REVENUE
FROM CTE;
```
**Purpose:** Calculates the total revenue generated from all orders by summing up the total price of pizzas sold.

**Output:** Total revenue rounded to 2 decimal places. Example:
```
TOTAL_REVENUE
-------------
12345.67
```

---

## CS3: Revenue by Pizza Size
**Query:**
```sql
SELECT 
    P.SIZE, 
    ROUND(SUM(P.PRICE * OD.QUANTITY),0) AS TOTAL_REVENUE
FROM ORDER_DETAILS OD 
LEFT JOIN PIZZAS P ON OD.PIZZA_ID = P.PIZZA_ID
GROUP BY 1
ORDER BY 2 DESC;
```
**Purpose:** Computes the total revenue for each pizza size and sorts them in descending order.

**Output:** Revenue breakdown by pizza size. Example:
```
SIZE     TOTAL_REVENUE
-------- -------------
Large    5000
Medium   3000
Small    1500
```

---

## CS4: Count of Orders by Day
**Query:**
```sql
WITH CTE AS (SELECT
        DATE, 
        COUNT(ORDER_ID) AS DAILY_SALES,
        RANK() OVER (ORDER BY COUNT(ORDER_ID) DESC) AS RN
FROM ORDERS
GROUP BY 1)
SELECT * 
FROM CTE;
```
**Purpose:** Provides the count of orders for each day, ranking them by the highest daily sales.

**Output:** Orders ranked by day. Example:
```
DATE       DAILY_SALES RN
---------- ----------- --
2023-12-01 50          1
2023-12-02 40          2
```

---

## CS5: Count of Orders in December
**Query:**
```sql
SELECT 
    SUM(IDS) AS TOTAL_ORDERS_IN_DECEMBER 
FROM  
    (SELECT 
        DATE,
        COUNT(ORDER_ID) AS IDS
    FROM ORDERS 
    WHERE STR_TO_DATE(date, '%Y-%m-%d')  BETWEEN '2015-12-01' AND '2015-12-30' 
    GROUP BY 1) X;
```
**Purpose:** Computes the total number of orders placed in December 2015.

**Output:** Total number of orders in December. Example:
```
TOTAL_ORDERS_IN_DECEMBER
-------------------------
120
```

---

## CS6: Monthly Pizza Sales
**Query:**
```sql
WITH M_P_S AS (
    SELECT 
        STR_TO_DATE(date, '%Y-%m-%d') AS DAYS, 
        ORDER_ID, 
        TIME
    FROM ORDERS) 
SELECT 
    EXTRACT(MONTH FROM DAYS) AS MONTH, 
    COUNT(ORDER_ID) AS TOTAL_ORDERS
FROM M_P_S 
GROUP BY 1
ORDER BY 1, 2 DESC;
```
**Purpose:** Aggregates the number of orders per month.

**Output:** Monthly sales count. Example:
```
MONTH TOTAL_ORDERS
----- ------------
1     100
2     90
```

---

## CS7: Orders by Time of Day
**Query:**
```sql
WITH M_A_E AS (
    SELECT 
        HOUR(STR_TO_DATE(TIME, '%H:%i:%s')) AS TIMESS, 
        ORDER_ID
    FROM ORDERS)
SELECT 
    CASE
        WHEN TIMESS BETWEEN 0 AND 11 THEN 'MORNING'
        WHEN TIMESS BETWEEN 12 AND 17 THEN 'AFTERNOON'
        WHEN TIMESS BETWEEN 18 AND 23 THEN 'EVENING'
    END AS TIME_OF_THE_DAY,
    COUNT(ORDER_ID) AS TOTAL_ORDERS
FROM M_A_E
GROUP BY 1
ORDER BY 2 DESC;  
```
**Purpose:** Categorizes and counts orders based on the time of the day.

**Output:** Orders segmented by time of day. Example:
```
TIME_OF_THE_DAY TOTAL_ORDERS
--------------- ------------
EVENING         50
AFTERNOON       30
MORNING         20
