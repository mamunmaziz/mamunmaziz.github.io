---
title: "MYSQL Portfolio: Some useful MySQL queries I have run"
Date: 2022-12-12
tags: [MYSQL]
header:
  image: "/images/mysql.jpg"
excerpt: "MYSQL"
mathjax: "true"
---


## Introduction:

In my practise of enhancing my SQL knowledge, I have sorted out a few extremely helpful intermediate and advanced SQL queries  as listed below.

## :large_blue_diamond: :white_circle: Problem :one::

Write a query to display the comparison result (higher/lower/same) of the average salary of employees in a department to the company’s average salary per month..

### Create schema 
```sql
CREATE DATABASE IF NOT EXISTS practicesql;
```

### TABLE STRUCTURE 

```sql
# salary table
CREATE TABLE IF NOT EXISTS salary (
  id int(9) unsigned NOT NULL AUTO_INCREMENT,
  employee_id int(9) unsigned NOT NULL,
  amount int(9) unsigned NOT NULL,
  pay_date date DEFAULT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO salary (id, employee_id, amount, pay_date)
VALUES ( 1  , 1 , 9000   , '2022-03-31'), 
	(2 ,  2 , 6000   , '2022-03-31'), 
	(3,   3, 10000   , '2022-03-31'),  
	(4,   1 , 7000   , '2022-02-28'), 
	(5  , 2 , 6000   , '2022-02-28'), 
	(6  , 3 , 8000   , '2022-02-28'),
	(7  , 3 , 8500   , '2022-03-31');

# employee table
CREATE TABLE IF NOT EXISTS employee (
  employee_id varchar(255) NOT NULL,
  department_id int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
INSERT INTO employee (employee_id, department_id)
VALUES  (1 , 1 ), 
(2 , 2 ), 
(3 , 2 );
```
( NBote:  Explanation of " ENGINE=InnoDB DEFAULT CHARSET=utf8;"
UTF-8 [Universal Character Set Transformation Format 8-bit] character set
UTF-8 is a standard code used to convert alphabets and characters into bits that computers can understand, much like ASCII. Almost majority of the characters we employ have already been assigned an 8-bit size value by UTF-8. The most often used character set as of late is this one.

ENGINE=InnoDB
A database storage engine is called InnoDB. Tables are saved, retrieved, and processed by the database storage engine. Although InnoDB is MySQL's fastest storage engine, effective configuration requires a specialist.

#### salary table

|id |employee_id|amount|pay_date  |
|---|-----------|------|----------|
|1  |1          |9000  |2017-03-31|
|2  |2          |6000  |2022-03-31|
|3  |3          |10000 |2022-03-31|
|4  |1          |7000  |2022-02-28|
|5  |2          |6000  |2022-02-28|
|6  |3          |8000  |2022-02-28|

#### employee table

|employee_id|department_id|
|-----------|-------------|
|1          |1            |
|2          |2            |
|3          |2            |


With the help of the WITH keyword, a temporary table was made utilising the window function, subquestions, and joins.

```sql
With final_table AS(
  SELECT  DATE_FORMAT(pay_date, "%Y-%m") AS pay_month, department_id,
    AVG(amount) OVER(PARTITION BY DATE_FORMAT(pay_date, "%m"), department_id) AS dept_avg,
    AVG(amount) OVER(PARTITION BY DATE_FORMAT(pay_date, "%m")) AS comp_avg 
FROM salary 
JOIN employee USING (employee_id)
  )
 SELECT* FROM final_table;
```
The outcome of the code above is shown here:

| pay_month | department_id | dept_avg  | comp_avg  |
| --------- | ------------- | --------- | --------- |
| 2022-02   | 1             | 7000.0000 | 7000.0000 |
| 2022-02   | 2             | 7000.0000 | 7000.0000 |
| 2022-02   | 2             | 7000.0000 | 7000.0000 |
| 2022-03   | 1             | 9000.0000 | 8375.0000 |
| 2022-03   | 2             | 8166.6667 | 8375.0000 |
| 2022-03   | 2             | 8166.6667 | 8375.0000 |
| 2022-03   | 2             | 8166.6667 | 8375.0000 |

All that remains to be done is to select the proper features and provide the proper options for our final presentation. The DISTINCT keyword will be used to remove the duplicate columns from the result set, and the CASE WHEN clause will be used to provide our criteria.

```sql
With final_table AS(
      SELECT  DATE_FORMAT(pay_date, "%Y-%m") AS pay_month, department_id,
        AVG(amount) OVER(PARTITION BY DATE_FORMAT(pay_date, "%m"), department_id) AS dept_avg,
        AVG(amount) OVER(PARTITION BY DATE_FORMAT(pay_date, "%m")) AS comp_avg 
    FROM salary 
    JOIN employee USING (employee_id)
      )
      
     SELECT DISTINCT pay_month, department_id,
    	CASE 
    		WHEN dept_avg > comp_avg THEN 'higher'
            WHEN dept_avg = comp_avg THEN 'same'
        ELSE 'lower' END AS comparison
        FROM final_table;
```
## Output:

Now, we can show whether the average salary of employees in a department is higher, lower, or the same as the average salary of the company:

| pay_month | department_id | comparison |
| --------- | ------------- | ---------- |
| 2022-02   | 1             | same       |
| 2022-02   | 2             | same       |
| 2022-03   | 1             | higher     |
| 2022-03   | 2             | lower      |

---



---

## :large_blue_diamond: :white_circle: Problem :two: :
Create a query to display the number of unpurchased seats for each flight ID.

## Table provided :
### Table1: flights

|flight_id|plane_id|
|---------|:------:|
|1        |12      |
|2        |15      |
|3        |13      |
|4        |14      |
|5        |16      |
|6        |11      |

### Table2: planes

|plane_id|number_of_seats|
|--------|:-------------:|
|15      |30             |
|12      |18             |
|13      |40             |
|14      |50             |
|16      |35             |

### Table3: purchases

|flight_id|seat_no|
|---------|:-----:|
|1        |18     |
|1        |20     |
|1        |31     |
|1        |12     |
|2        |40     |
|2        |50     |
|3        |50     |
|3        |40     |
|3        |41     |
|4        |50     |
|5        |51     |
|5        |50     |
|5        |52     |
|4        |23     |
|5        |13     |


## Solution :
-- number of purchases for each flight.

-- (number of unpurchased seat = Total number of seats- Number of seats purchased) 

-- Need to join the plane data onto the flight data to determine the number of seats available, then join  new purchases table to subtract the purchases from the available seats

```sql
with purchased as (
select
flight_id
,count(flight_id) seats_purchased
from purchases
group by flight_id
)

select
f.flight_id
,p.number_of_seats-pr.seats_purchased seats_unpurchased
from flights f
left join planes p
on p.plane_id=f.plane_id
left join purchased pr
on pr.flight_id=f.flight_id
order by f.flight_id asc;
```

## output:

|flight_id| seats_unpurchased    |
|---------|:--------------------:|
|1        |14                    |
|2        |28                    |
|3        |37                    |
|4        |48                    |
|5        |31                    |

