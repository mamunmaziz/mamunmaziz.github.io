---
title: "MYSQL Portfolio: Some useful MySQL case studies I have run "
Date: 2023-01-01
tags: [MYSQL]
header:
  image: "/images/mysql.jpg"
excerpt: "MYSQL"
mathjax: "true"
---

Solved on MySQL workbench 8.0 by Mamun Aziz, 27 Dec 2022

# Introduction
A variety of SQL techniques were used to solve the problem on MySQL Workbench 8.0, including conditional WHERE statements, data aggregation with GROUP BY and ordering with ORDER BY, CASE WHEN statements, string transformations, regular expressions, datetime manipulation, common table expressions (CTE), subqueries and nested queries, complex table joins (Inner, Outer, Left, Right, Lateral - no Cross Joins) and window functions (row_number(), rank(), dense_rank(), lag(), lead()).


# :white_circle: Case Study :one:: Shumi's Kitchen


### Schema (MySQL v8.0)

    ```sql
    CREATE SCHEMA shumi_kitchen;
    ```
### Create Tables 

```sql
    CREATE TABLE sales (
      customer_id VARCHAR(1),
      order_date DATE NOT NULL,
      product_id INT(11)
    )ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
    INSERT INTO sales
      (customer_id, order_date, product_id)
    VALUES
      ('A', '2022-01-01', '1'),
      ('A', '2022-01-01', '2'),
      ('A', '2022-01-07', '2'),
      ('A', '2022-01-10', '3'),
      ('A', '2022-01-11', '3'),
      ('A', '2022-01-11', '3'),
      ('B', '2022-01-01', '2'),
      ('B', '2022-01-02', '2'),
      ('B', '2022-01-04', '1'),
      ('B', '2022-01-11', '1'),
      ('B', '2022-01-16', '3'),
      ('B', '2022-02-01', '3'),
      ('C', '2022-01-01', '3'),
      ('C', '2022-01-01', '3'),
      ('C', '2022-01-07', '3');
      
      CREATE TABLE menu (
      product_id INTEGER,
      product_name VARCHAR(5),
      price INTEGER
    );
    
    INSERT INTO menu
      (product_id, product_name, price)
    VALUES
      ('1', 'rice', '10'),
      ('2', 'curry', '15'),
      ('3', 'pasta', '12');
      
    
    CREATE TABLE members (
      customer_id VARCHAR(1),
      join_date DATE
    );
    
    INSERT INTO members
      (customer_id, join_date)
    VALUES
      ('A', '2022-01-07'),
      ('B', '2022-01-09');
    ```
     
### Table 1: sales

| customer_id | order_date | product_id |
| ----------- | ---------- | ---------- |
| A           | 2022-01-01 | 1          |
| A           | 2022-01-01 | 2          |
| A           | 2022-01-07 | 2          |
| A           | 2022-01-10 | 3          |
| A           | 2022-01-11 | 3          |
| A           | 2022-01-11 | 3          |
| B           | 2022-01-01 | 2          |
| B           | 2022-01-02 | 2          |
| B           | 2022-01-04 | 1          |
| B           | 2022-01-11 | 1          |
| B           | 2022-01-16 | 3          |
| B           | 2022-02-01 | 3          |
| C           | 2022-01-01 | 3          |
| C           | 2022-01-01 | 3          |
| C           | 2022-01-07 | 3          |


### Table 2: menu

| product_id | product_name | price |
| ---------- | ------------ | ----- |
| 1          | rice         | 10    |
| 2          | curry        | 15    |
| 3          | pasta        | 12    |


### Table 3: members

| customer_id | join_date  |
| ----------- | ---------- |
| A           | 2022-01-07 |
| B           | 2022-01-09 |


## Problem Statement
Shumi wants to use the information to get clear answers about his consumers, particularly regarding their spending habits, frequency of visits, and favourite menu items. Delivering a better and more individualised experience for his devoted clients will be made possible by having this closer relationship with them.

## Questions and Solutions

### :pencil2: 1.	How much money did each customer spend overall at the restaurant?

```sql
SELECT
      customer_id,
      SUM(price) as total_spent
    FROM
      sales as s
      JOIN menu as m ON s.product_id = m.product_id
    GROUP BY
      1
    ORDER BY
      1;
```
### Output

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

### :pencil2: 2.	How many days did each customer spend at the restaurant?

```sql
SELECT
      customer_id,
      COUNT(DISTINCT order_date) as date_visited
    FROM
      sales
    GROUP BY
      1
    ORDER BY
      1;
```

### Output
| customer_id | date_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |


( Note:  DISTINCT is important here as a customer can have two entry in same day)

### :pencil2: 3.	Which dish did each customer order first from the menu?
 
### Solution

We must rank/row_number the products requested by each client in a temporary table using the WITH statement in order to obtain the first item.

After obtaining such rankings, we can choose the rows with row_num = 1. Due to the fact that customer A placed two orders on the first day, we must apply ORDER BY in the window function based on the order date and product id criteria.

```sql
WITH ranked AS(
      SELECT 
     customer_id,   
     product_id,
     product_name,
     order_date,
        ROW_NUMBER() OVER(PARTITION BY customer_id  ORDER BY order_date, product_id) AS row_num  
    FROM sales
    JOIN menu USING(product_id) 
     )
      
    SELECT
      customer_id,
      product_name,
      order_date
    FROM
      ranked
    Where
      row_num = 1;
```

### Output
| customer_id | product_name | order_date |
| ----------- | ------------ | ---------- |
| A           | rice         | 2022-01-01 |
| B           | curry        | 2022-01-01 |
| C           | pasta        | 2022-01-01 |


### :pencil2: 4.	Which item on the menu is most popular, and how frequently is it ordered by all customers?

```sql
WITH totals AS (
      WITH quantity AS(
      SELECT
         product_name,
         COUNT(product_name) AS purchase_quantity 
        FROM sales
        JOIN menu USING(product_id) 
      GROUP BY
      1 
      )
    
      SELECT  
      product_name,
      purchase_quantity,
      ROW_NUMBER() OVER(ORDER BY purchase_quantity DESC) AS row_num  
      FROM
       quantity
      )
      
      SELECT
      product_name,
      purchase_quantity as Maximum_purchased 
    FROM
      totals
    WHERE
      row_num = 1;

#(OR…

 WITH quantity AS(
 SELECT
     product_name,
     COUNT(product_name) AS purchase_quantity 
    FROM sales
    JOIN menu USING(product_id) 
  GROUP BY
  1 
),
totals AS ( 
   SELECT  
  product_name,
  purchase_quantity,
  ROW_NUMBER() OVER(ORDER BY purchase_quantity DESC) AS row_num  
  FROM
   quantity
  )
  
  SELECT
  product_name,
  purchase_quantity as Maximum_purchased 
  
 FROM
      totals
    WHERE
      row_num = 1
      
      )
```

### output

| product_name | Maximum_purchased |
| ------------ | ----------------- |
| pasta        | 8                 |

Pasta was the most frequently ordered item on the menu, with an overall purchase of 8.

### :pencil2: 5. Which product did each consumer find to be the most popular?
  


 ```sql
 SELECT
      customer_id,
      product_name,
      COUNT(product_name) AS total_purchase_quantity
    FROM
      sales AS s
      INNER JOIN menu AS m ON s.product_id = m.product_id
    GROUP BY
      customer_id,
      product_name
    ORDER BY
      customer_id,
      total_purchase_quantity DESC;
      
   ```
 All the results sorted by purchase frequency:
 
| customer_id | product_name | total_purchase_quantity |
| ----------- | ------------ | ----------------------- |
| A           | pasta        | 3                       |
| A           | curry        | 2                       |
| A           | rice         | 1                       |
| B           | curry        | 2                       |
| B           | rice         | 2                       |
| B           | pasta        | 2                       |
| C           | pasta        | 3                       |

With the help of the rank window function, we can now choose the most well-liked products for each customer:

```sql

WITH quantity AS(
      SELECT
          customer_id,
          product_name,
          COUNT(product_name) AS purchase_quantity
      FROM
          sales As s
          JOIN menu AS m ON s.product_id = m.product_id
        GROUP BY
          customer_id,
          product_name
    ),
    ranked AS(
       SELECT  
      customer_id,
      product_name,
      purchase_quantity,
      RANK() OVER(PARTITION BY customer_id ORDER BY purchase_quantity DESC) AS rank_number 
      FROM
       quantity
    )
    
    SELECT
      customer_id,
      product_name,
      purchase_quantity
    FROM
      ranked
    WHERE
      rank_number = 1;
```

### Output

| customer_id | product_name | purchase_quantity |
| ----------- | ------------ | ----------------- |
| A           | pasta        | 3                 |
| B           | curry        | 2                 |
| B           | rice         | 2                 |
| B           | pasta        | 2                 |
| C           | pasta        | 3                 |



### :pencil2: 6.	Which product was the customer's first purchase after joining?

### Solution:
Let's assume that if the purchase date and membership date are the same, the purchase made on that day was the first one made by the consumer after becoming a member. It implies that the WHERE statement must include this date.

```sql
WITH ranked AS (
        SELECT
          s.customer_id,
          order_date,
          join_date,
          product_name,
          row_number() OVER (PARTITION BY s.customer_id
            ORDER BY
              order_date
          ) AS rank_number
        FROM
          sales AS s
          JOIN members AS mm ON s.customer_id = mm.customer_id
          JOIN menu AS m ON s.product_id = m.product_id
        WHERE
          order_date >= join_date
      )
    SELECT
      customer_id,
      join_date,
      order_date,
      product_name
    FROM
      ranked
    WHERE
      rank_number = 1
    ORDER BY
      1;
```
### Output

| customer_id | join_date  | order_date | product_name |
| ----------- | ---------- | ---------- | ------------ |
| A           | 2022-01-07 | 2022-01-07 | curry        |
| B           | 2022-01-09 | 2022-01-11 | rice         |

### :pencil2: 7.	What product did the customer just buy before joining?

```sql
WITH ranked AS (
        SELECT
          s.customer_id,
          order_date,
          join_date,
          product_name,
          rank() OVER (
            PARTITION BY s.customer_id
            ORDER BY
              order_date DESC
          ) AS rank_number
        FROM
          sales AS s
          JOIN members AS mm ON s.customer_id = mm.customer_id
          JOIN menu AS m ON s.product_id = m.product_id
        WHERE
          order_date < join_date
      )
    SELECT
      customer_id,
      join_date,
      order_date,
      product_name
    FROM
      ranked
    WHERE
      rank_number = 1
    ORDER BY
      1;
 ```
 ### Output
 
| customer_id | join_date  | order_date | product_name |
| ----------- | ---------- | ---------- | ------------ |
| A           | 2022-01-07 | 2022-01-01 | rice         |
| A           | 2022-01-07 | 2022-01-01 | curry        |
| B           | 2022-01-09 | 2022-01-04 | rice         |

### :pencil2: 8.	What is the total items and amount spent for each member before they became a member?

```sql
SELECT
      s.customer_id,
      COUNT(product_name) AS total_number_of_items,
      SUM(price) AS total_purchase_amount
    FROM
      sales AS s
      JOIN members AS mm ON s.customer_id = mm.customer_id
      JOIN menu AS m ON s.product_id = m.product_id
    WHERE
      order_date < join_date
    GROUP BY
      1
    ORDER BY
      1;
```
### Output

| customer_id | total_number_of_items | total_purchase_amount |
| ----------- | --------------------- | --------------------- |
| A           | 2                     | 25                    |
| B           | 3                     | 40                    |


### :pencil2: 9.	How many points would each client have if each $1 spent equalled 10 points and rice  had a 2x points multiplier?

```sql
SELECT
      customer_id,
      SUM(points) AS points
    FROM
      sales AS s
      JOIN (
        SELECT
          product_id,
          CASE
            WHEN product_id = 1 THEN price * 20
            ELSE price * 10
          END AS points
        FROM
          menu
      ) AS p ON s.product_id = p.product_id
    GROUP BY
      1
    ORDER BY
      1;
```
### Output

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

### :pencil2: 10.	Join the three tables together, distinguish between members and non-members, then rank the members in the same table.

```sql
WITH mem_nonmem AS(
    SELECT
          s.customer_id,
          order_date,
          product_name,
          price,
         CASE
        WHEN order_date >= join_date THEN 'Y'
        ELSE 'N'
        END AS member
    FROM
          sales AS s
          JOIN menu AS m ON s.product_id = m.product_id
          LEFT JOIN members AS mm ON s.customer_id = mm.customer_id
    )
    SELECT
      *,
      CASE
        WHEN member = 'Y' 
        THEN rank() OVER (PARTITION BY customer_id, member  
                          ORDER BY  order_date)
        END AS ranking
    FROM
      mem_nonmem
    ORDER BY
      customer_id,
      order_date,
      product_name;
```
### Output

| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | :----: | :-----: |
| A           | 2022-01-01 | curry        | 15    | N      |         |
| A           | 2022-01-01 | rice         | 10    | N      |         |
| A           | 2022-01-07 | curry        | 15    | Y      | 1       |
| A           | 2022-01-10 | pasta        | 12    | Y      | 2       |
| A           | 2022-01-11 | pasta        | 12    | Y      | 3       |
| A           | 2022-01-11 | pasta        | 12    | Y      | 3       |
| B           | 2022-01-01 | curry        | 15    | N      |         |
| B           | 2022-01-02 | curry        | 15    | N      |         |
| B           | 2022-01-04 | rice         | 10    | N      |         |
| B           | 2022-01-11 | rice         | 10    | Y      | 1       |
| B           | 2022-01-16 | pasta        | 12    | Y      | 2       |
| B           | 2022-02-01 | pasta        | 12    | Y      | 3       |
| C           | 2022-01-01 | pasta        | 12    | N      |         |
| C           | 2022-01-01 | pasta        | 12    | N      |         |
| C           | 2022-01-07 | pasta        | 12    | N      |         |

---
---
