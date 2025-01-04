# SQL Queries with Outputs

## CS1: Extract Table Information

Query:
```sql
describe orders;
describe order_details;
```
Output:
| Field      | Type         | Null | Key | Default | Extra |
|------------|--------------|------|-----|---------|-------|
| order_id   | int          | YES  | PRI | NULL    |       |
| customer   | varchar(255) | YES  |     | NULL    |       |
| date       | date         | YES  |     | NULL    |       |
| order_id   | int          | Yes  | PRI | NULL    |       |
| pizza_id   | int          | YES  | PRI | NULL    |       |
| quantity   | int          | YES  |     | NULL    |       |

## CS2: Total Revenue from All Orders

Query:
```sql
WITH CTE AS (
    SELECT 
        ORDER_ID, OD.PIZZA_ID, QUANTITY, PRICE, (QUANTITY * PRICE) AS TOTAL_PRICE
    FROM 
        ORDER_DETAILS OD
    LEFT JOIN PIZZAS P ON OD.PIZZA_ID = P.PIZZA_ID ) 
SELECT 
    ROUND(SUM(TOTAL_PRICE),2) AS TOTAL_REVENUE
FROM CTE;
```
Output:
| TOTAL_REVENUE |
|---------------|
| 817860.05     |

## CS3: Revenue by Pizza Size

Query:
```sql
SELECT 
    P.SIZE, 
    ROUND(SUM(P.PRICE * OD.QUANTITY),0) AS TOTAL_REVENUE
FROM ORDER_DETAILS OD 
LEFT JOIN PIZZAS P ON OD.PIZZA_ID = P.PIZZA_ID
GROUP BY 1
ORDER BY 2 DESC;
```
Output:
| SIZE   | TOTAL_REVENUE |   
|--------|---------------|
| L      | 375319        |
| M      | 249382        |
| S      | 178076        |
| XL     | 14076         |
| XXL    | 1007          |

## CS4: Count of Orders by Day

Query:
```sql
WITH CTE AS (
    SELECT
        DATE, 
        COUNT(ORDER_ID) AS DAILY_SALES,
        RANK() OVER (ORDER BY COUNT(ORDER_ID) DESC) AS RN
        FROM ORDERS
        GROUP BY 1)
SELECT * 
FROM CTE;
```
Output:
| DATE       | DAILY_SALES | RN |
|------------|-------------|----|
| 2015-11-27 | 115         | 1  |
| 2015-11-26 | 113         | 2  |
| 2015-10-15 | 107         | 3  |
| 2015-04-04 | 105         | 4  |
| 2015-05-15 | 94          | 5  |

## CS5: Count of Orders in December

Query:
```sql
SELECT 
    SUM(IDS) AS TOTAL_ORDERS_IN_DECEMBER 
FROM  
    (SELECT 
        DATE,
        COUNT(ORDER_ID) AS IDS
    FROM ORDERS 
    WHERE STR_TO_DATE(date, '%Y-%m-%d')  BETWEEN '2015-12-01' AND '2015-12-30' 
    GROUP BY 1) X ;
```
Output:
| TOTAL_ORDERS_IN_DECEMBER |
|--------------------------|
| 1607                     |

## CS6: Monthly Pizza Sales

Query:
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
Output:
| MONTH | TOTAL_ORDERS |
|-------|--------------|
| 1     | 1799         |
| 2     | 1853         |
| 3     | 1773         |
| 4     | 1935         |
| 5     | 1841         |
| 6     | 1661         |
| 7     | 1646         |
| 8     | 1792         |
| 9     | 1846         |
| 10    | 1680         |
| 11    | 1685         |
| 12    | 1840         |

## CS7: Orders by Time of Day

Query:
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
Output:
| TIME_OF_THE_DAY | TOTAL_ORDERS |
|-----------------|--------------|
| EVENING         | 12171        |
| AFTERNOON       | 7939         |
| MORNING         | 1240         |
