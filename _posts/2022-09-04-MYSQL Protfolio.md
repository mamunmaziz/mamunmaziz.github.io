---
title: "MYSQL Portfolio: MySQL Queries for Study Purpose"
Date: 2022-06-13
tags: [MYSQL]
header:
  image: "/images/mysql.jpg"
excerpt: "MYSQL"
mathjax: "true"
---


## Introduction 
I am very familiar with the open-source relational database management system (RDBMS), MySQL. The master's in data science course I took included a full course dealing with MySQL. In addition, I have completed a few MYSQL courses on Coursera and Linkdin. Moreover, I used MySQL to query the customers' bills and call details from the switching server while I worked for the Bangladesh Telecommunication (BTCL) Company in Bangladesh.  

## The Key skills and knowledge applied :

- Designed the database architecture
- Designing Data Flow Diagrams, Entity Relationship Diagrams, and Data Structure Diagrams.
- Performed database development and implementation activities.
- Responsible for configuring, tuning, and maintaining MySQL Server database servers.
- Implemented the necessary procedures and packages for extracting, transforming, and loading data.
-  Enabled views, stored procedures, and queries in the database



# Example of creating, loading, and searching a database under study courses 
---------------------------------------------------------------------------------------
In this study a large dataset of “California Population Projection by County, Age, Gender, and Ethnicity” was used.

### Create a DATABASE:
We need to create a new schema once the MySQL Workbench is open, and the local server is connected. We'll call it ‘cal_pop’ for our California population data.  After I've entered the name, I'll click Apply. Once the newly created schemas have appeared in the Navigator pane, we can see our new 'cal_pop' database. When I double-clicked it, I saw that there are folders for Tables, Views, and Procedures and Functions, but they are all empty right now.

### Create a table in MySQL:
The next step is to create a table within our database so that we can store our data. The easiest way to create a new table is to right-click on the Tables folder beneath the name of your database and select Create Table. Because we will be storing California population projections, I named the table 'pop_projection'.

<img src="/images/2022-09-04/2sql.JPG" width="912"/>

The same table can be create using mysql command syntex as:

```sql
CREATE TABLE `cal_pop`.`pop_projection` (
  `county_code` INT NOT NULL,
  `county` VARCHAR(45) NULL,
  `date_year` VARCHAR(4) NULL,
  `race_code` INT NULL,
  `race` TEXT NULL,
  `gender` VARCHAR(6) NULL,
  `age` INT NULL,
  `population` INT NULL);
  ```

### Load DATA :
Data can be entered into a MySQL table in several ways. In my first attempt, I tried using the 'Table Data Import Wizard', however, it takes much longer to load the data than I expected. Furthermore, it can only load a portion of it.
Therefore, I have used MySQL's 'Load Data' statement.


```sql
LOAD DATA LOCAL INFILE dir:\\<path…>\\CA_DRU_proj_2010-2060.csv'
INTO TABLE pop_projection
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES;
```
But, when I use 'LOAD DATA LOCAL INFILE' in mysqlworkbench 8.0, I receive the following error.
```sql
Error Code: 3948. Loading local data is disabled; this must be enabled on both the client and server sizes 0.000 sec.
```
Solution of loading data from local data:

01) With the mysql terminal command:

-	Logged in first with the usual:

```sql
mysql -u root -p
```
-	Check the  status with the command:
```sql
SHOW GLOBAL VARIABLES LIKE 'local_infile';
```
-	set the option: local-infile=1, or local_infile=true;

```sql
SET GLOBAL local_infile = true;

```

02) Edit the connection settings in MySQL Workbench.

- The following connector option should be added to the others window under the Advanced tab:

OPT_LOCAL_INFILE=1

<img src="/images/2022-09-04/3sql.JPG" width="700"/>


Next, 'LOAD DATA LOCAL INFILE...' is being executed . This resulted in 36 million rows of data being loaded into our newly created table.


<img src="/images/2022-09-04/4sql.jp.JPG" width="912"/>

The following command displays the loaded data in our created table.

```sql
SELECT * FROM cal_pop.pop_projection;
```
Result:
<img src="/images/2022-09-04/5sql.JPG" width="500"/>

<br>

<br>

# Example of JOIN datasets under a study course
------------------------------------------------
As a starting point, we basically started with two datasets and joined them. Our next step was to perform ETL on them.
We used road safety data for year 2019 published by the UK government at their open data website, <https://ckan.publishing.service.gov.uk/dataset/road-accidents-safety-data>

Three fields were used from those datasets: accident severity, accident index, and vehicle type. There is also a file that contains definitions of data types and descriptions of vehicle types. The accident index connects the databases. We can use that link to create a relationship between the datasets. By using the common fields, we have gathered information to determine the median and average accident severity for each type of motorcycle.

Using this data, we can determine if there is an association between motorcycle type and accident type. 

###  DATASET
We have initially two datasets as :
1.	Accidents_2019.csv
2.	Vehicle_2019.csv

Another dataset is as 'Road-Accident-Safety-Data-Guide.xls', from which a new dataset can be created as 'vehicle_type.csv'.

### Create SCHEMA and Tables
+ The first step was to create a structure for the data. Thus, we have created a new schema 'accident' and set it to be the default schema.
+ Table: accidents_2019
```sql
CREATE TABLE accidents_2019 (
Accident_Index VARCHAR(13),
 Accident_Severity INT );
```
+ Table: vehicles_2019
```sql
CREATE TABLE vehicles_2019 (
Accident_Index VARCHAR(13),
Vehicle_Type VARCHAR(10) );```
+ Table: vehicles_types
```sql
CREATE TABLE vehicles_type (
vehicle_code INT,
Vehicle_Type VARCHAR(100) );
```

