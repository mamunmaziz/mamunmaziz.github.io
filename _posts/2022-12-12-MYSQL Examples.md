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
CREATE DATABASE IF NOT EXISTS practicesql_1;
```

### Table structure 

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
( note:  Explanation of " ENGINE=InnoDB DEFAULT CHARSET=utf8;"
UTF-8 [Universal Character Set Transformation Format 8-bit] character set
UTF-8 is a standard code used to convert alphabets and characters into bits that computers can understand, much like ASCII. Almost the majority of the characters we employ have already been assigned an 8-bit size value by UTF-8. The most often used character set as of late is this one.

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

## ### Table structure 

```sql

 # flight table
    
    CREATE TABLE IF NOT EXISTS flights(
      flight_id int(9) unsigned NOT NULL auto_increment,
      plane_id int(9) unsigned NOT NULL,
      PRIMARY KEY (flight_id)
      );
      
    INSERT INTO flights (flight_id, plane_id)
    VALUES  (1, 12), 
    		(2, 15),
            (3, 13),
            (4, 14),
            (5, 16),
            (6, 11);
    
    # planes table
    
    CREATE TABLE IF NOT EXISTS planes(
      plane_id VARCHAR(255),
      no_of_seats int(12) unsigned NOT NULL
       );
      
    INSERT INTO planes (plane_id,  no_of_seats)
    VALUES  (15, 30), 
    		(12, 25),
            (13, 50),
            (14, 50),
            (16, 55),
            (11, 45);
            
    # purchases table
    
    CREATE TABLE IF NOT EXISTS purchases(
      flight_id VARCHAR(255),
      seat_no int(9) unsigned NOT NULL
       );
      
    INSERT INTO purchases (flight_id, seat_no)
    VALUES  (1, 18), 
    		(1, 20),
            (1, 12),
            (1, 21), 
    		(2, 20),
            (2, 30),
            (3, 50),
            (3, 41),
            (3, 30),
            (4, 51),
            (5, 16),
            (5, 52),
            (6, 19),
            (4, 23),
            (5, 18),
            (6, 37),
            (6, 43);
```

            
### Table1: flights

| flight_id | plane_id |
| --------- | -------- |
| 1         | 12       |
| 2         | 15       |
| 3         | 13       |
| 4         | 14       |
| 5         | 16       |
| 6         | 11       |


### Table2: planes

| plane_id | no_of_seats |
| -------- | ----------- |
| 15       | 30          |
| 12       | 25          |
| 13       | 50          |
| 14       | 50          |
| 16       | 55          |
| 11       | 45          |


### Table3: purchases

| flight_id | seat_no |
| --------- | ------- |
| 1         | 18      |
| 1         | 20      |
| 1         | 12      |
| 1         | 21      |
| 2         | 20      |
| 2         | 30      |
| 3         | 50      |
| 3         | 41      |
| 3         | 30      |
| 4         | 51      |
| 5         | 16      |
| 5         | 52      |
| 6         | 19      |
| 4         | 23      |
| 5         | 18      |
| 6         | 37      |
| 6         | 43      |


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
    ,p.no_of_seats-pr.seats_purchased seats_unpurchased
    from flights f
    left join planes p
    on p.plane_id=f.plane_id
    left join purchased pr
    on pr.flight_id=f.flight_id
    order by f.flight_id asc;

```

## output:

| flight_id | seats_unpurchased |
| :-------: | :---------------: |
| 1         | 21                |
| 2         | 28                |
| 3         | 47                |
| 4         | 48                |
| 5         | 52                |
| 6         | 42                |

---

---

## :large_blue_diamond: :white_circle: Problem :three::

To report the students (student id, student name) being "silent" in ALL tests, use a SQL query. A "silent" student is one who has taken at least one exam but did not receive either a high or bad grade.

### Create schema 
```sql
CREATE DATABASE IF NOT EXISTS practicesql_3;
```

### Changing default schema
```sql
USE practicesql_3;
```

### Table structure 

```sql
# student table
     CREATE TABLE IF NOT EXISTS student (
      student_id int(9) NOT NULL,
      student_name varchar(255) NOT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
    INSERT INTO student (student_id, student_name)
    VALUES (1, 'Shaila'), (2, 'Raiyan'), (3, 'Annaya'), (4, 'James'), (5, 'Inara'), (6, 'Ruhi'), (7, 'Aayan');
      
    
    # exam table
    CREATE TABLE IF NOT EXISTS exam (
      exam_id int(9) NOT NULL,
      student_id int(9) NOT NULL,
      score int(11) NOT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
    INSERT INTO exam (exam_id, student_id, score)
    VALUES (10 ,1,70), 
    	(10 ,2,80), 
    	(10 ,3,90),
        (10 ,6,85),
    (20 ,1,80),
    (20 ,6,85),
    (12 ,7,90),
    (30 ,1,70),
    (30 ,3,80),
    (30 ,4,90),
    (40 ,1,60),
    (40 ,2,70),
    (40 ,4,80);
```
### Table: student    

| student_id | student_name |
| ---------- | ------------ |
| 1          | Shaila       |
| 2          | Raiyan       |
| 3          | Annaya       |
| 4          | James        |
| 5          | Inara        |
| 6          | Ruhi         |
| 7          | Aayan        |

### Table: exam   

| exam_id | student_id | score |
| ------- | ---------- | ----- |
| 10      | 1          | 70    |
| 10      | 2          | 80    |
| 10      | 3          | 90    |
| 10      | 6          | 85    |
| 20      | 1          | 80    |
| 20      | 6          | 85    |
| 12      | 7          | 90    |
| 30      | 1          | 70    |
| 30      | 3          | 80    |
| 30      | 4          | 90    |
| 40      | 1          | 60    |
| 40      | 2          | 70    |
| 40      | 4          | 80    |

### Solution :
-- A student who took at least one exam but didn't receive either a high score or the low score is considered to be "silent".
-- Returning a student who has never taken an exam is not acceptable.
-- Return the result table ordered by student_id.
-- Since there is no interest in kids who don't take tests, we do an inner join.


### Create temp table/ intermediary columns

```sql
WITH table1 AS(
SELECT exam_id, student_id, student_name, score,
	MIN(score) OVER(PARTITION BY exam_id) AS min_score,
	MAX(score) OVER(PARTITION BY exam_id) AS max_score
	FROM student INNER JOIN exam USING (student_id)
	ORDER BY exam_id
	) 
	SELECT * FROM table1;
```
 


| exam_id | student_id | student_name | score | min_score | max_score |
| ------- | ---------- | ------------ | ----- | --------- | --------- |
| 10      | 1          | Shaila       | 70    | 70        | 90        |
| 10      | 2          | Raiyan       | 80    | 70        | 90        |
| 10      | 3          | Annaya       | 90    | 70        | 90        |
| 10      | 6          | Ruhi         | 85    | 70        | 90        |
| 12      | 7          | Aayan        | 90    | 90        | 90        |
| 20      | 1          | Shaila       | 80    | 80        | 85        |
| 20      | 6          | Ruhi         | 85    | 80        | 85        |
| 30      | 1          | Shaila       | 70    | 70        | 90        |
| 30      | 3          | Annaya       | 80    | 70        | 90        |
| 30      | 4          | James        | 90    | 70        | 90        |
| 40      | 1          | Shaila       | 60    | 60        | 80        |
| 40      | 2          | Raiyan       | 70    | 60        | 80        |
| 40      | 4          | James        | 80    | 60        | 80        |

The same thing can be done using a WINDOW function as :
[[
```sql

WITH t1 AS(
  SELECT *
                FROM (
                       SELECT *,
                              MIN(score) OVER main_window as least,
                              MAX(score) OVER main_window as most
                         FROM exam
                       WINDOW main_window AS (PARTITION BY exam_id)
                     ) as a
              where least = score or most = score
  )
  
  SELECT *
  FROM t1;
```
]]



### Complete CODE
```sql
# Query from nested query
WITH table1 AS(
SELECT student_id
  FROM (
    SELECT *,
	MIN(score) OVER(PARTITION BY exam_id) AS min_score,
	MAX(score) OVER(PARTITION BY exam_id) AS max_score
	FROM exam
	ORDER BY exam_id
	)  as a   # as an alias
	WHERE min_score = score OR max_score = score 
)

# Added condition that outputs all the instances where score = min or max scores
# Need to return a unique result with at least one exam and no high or low score using the DISTINCT function.

SELECT DISTINCT student_id, student_name
	FROM exam JOIN student
	USING (student_id)

WHERE student_id != ALL(SELECT student_id FROM table1)
	ORDER BY 1;
```
### Output:
| student_id | student_name |
| ---------- | ------------ |
| 2          | Raiyan       |

---
---
