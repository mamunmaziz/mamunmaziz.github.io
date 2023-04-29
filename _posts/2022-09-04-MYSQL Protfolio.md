---
title: "MYSQL Portfolio: MySQL for Studying & Basic Queries"
Date: 2022-06-13
tags: [MYSQL]
header:
  image: "/images/mysql.jpg"
excerpt: "MYSQL"
mathjax: "true"
---


## Introduction 
The MySQL relational database management system (RDBMS) is open-source and well-known to me. My master's in data science programme included a comprehensive MySQL training. I've also finished a few MYSQL courses on Coursera and Linkdin. Moreover, while I was employed by the Bangladesh Telecommunication (BTCL) Company, I used MySQL to query the customers' bills and call information from the switching server.  

## The Key skills and knowledge applied :

- Designed the database architecture
- Designing Data Flow Diagrams, Entity Relationship Diagrams, and Data Structure Diagrams.
- Performed database development and implementation activities.
- Responsible for configuring, tuning, and maintaining MySQL Server database servers.
- Implemented the necessary procedures and packages for extracting, transforming, and loading data.
- Enabled views, stored procedures, and queries in the database



# Example of creating, loading, and searching a database under courses study
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

# Example of JOIN datasets under a course study
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
vcode INT,
vtype VARCHAR(100) );
```


### Load DATA
+ Load data to table ‘accidents_2019’
```sql
LOAD DATA LOCAL INFILE ‘dir:\\<path>\\Accidents_2019.csv' 
INTO TABLE accidents_2019 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n' 
IGNORE 1 LINES
(@col1, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @col2, @dummy, 
@dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, 
@dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, 
@dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy) 
SET accident_index=@col1, accident_severity=@col2;
```
+ Load data to table ‘accidents_2019’
```sql
LOAD DATA LOCAL INFILE dir:\\<path>\\\\Vehicle_2019.csv' 
INTO TABLE vehicles_2019 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n' 
IGNORE 1 LINES
(@col1, @dummy, @dummy,@dummy, @col2, @dummy, @dummy, @dummy, @dummy, @dummy, 
@dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, @dummy, 
@dummy, @dummy, @dummy, @dummy, @dummy, @dummy,@dummy,@dummy) 
SET accident_index=@col1, vehicle_type=@col2;
```
+ Load data to table ‘vehicles_type’
```sql
LOAD DATA LOCAL INFILE 'dir:\\<path>\\vehicle_type.csv' 
INTO TABLE vehicles_type 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n' 
IGNORE 1 LINES;
```

### Show Data in TABLES
```sql
SELECT * FROM accident.accidents_2019;
```

```sql
SELECT * FROM accident.vehicles_2019;
```
```sql
SELECT * FROM accident.vehicles_type;
```

### JOIN and QUERY
Our set target is to show the vehicle type, the average accident severity, and the number of accidents of the vehicle type by join operation in mysql as that data is stored in all three accidents tables.
The following figure shows the relations of three tables and our desired join activities in a line diagram  
<img src="/images/2022-09-04/6sql.JPG" width="850"/>

CODE:
```sql
SELECT 
    v.vtype AS 'Vehicle Type',
    AVG(a.Accident_Severity) AS 'Average Severity',
    COUNT(a.Accident_Severity) AS 'Number of Accidents'
FROM
    accidents_2019 AS a
        JOIN
    vehicles_2019 AS m ON a.Accident_Index = m.Accident_Index
        JOIN
    vehicles_type AS v ON m.Vehicle_Type = v.vcode
WHERE
    v.vtype LIKE '%otorcycle%'
GROUP BY v.vtype;
```
### Query Results
<img src="/images/2022-09-04/7sql.JPG" width="850"/>

The query gives us a result for the average severity and number of accidents per vehicle type in UK on the year 2019.


# :pencil2: A data sample and a basic MYSQL query example:
------------------------------------------------

### Create Tables
```sql 
CREATE TABLE Brands(
  Code INTEGER,
  Name VARCHAR(255) NOT NULL,
  PRIMARY KEY (Code)   
);

CREATE TABLE Products (
  Code INTEGER,
  Name VARCHAR(255) NOT NULL ,
  Price DECIMAL NOT NULL ,
  Brand INTEGER NOT NULL,
  PRIMARY KEY (Code), 
  FOREIGN KEY (Brand) REFERENCES Brands(Code)
) ENGINE=INNODB;

INSERT INTO Brands(Code,Name) VALUES(1,'SONY');
INSERT INTO Brands(Code,Name) VALUES(2,'Creative Labs');
INSERT INTO Brands(Code,Name) VALUES(3,'Hewlett-Packard');
INSERT INTO Brands(Code,Name) VALUES(4,'Iomega');
INSERT INTO Brands(Code,Name) VALUES(5,'Fujitsu');
INSERT INTO Brands(Code,Name) VALUES(6,'Winchester');
INSERT INTO Brands(Code,Name) VALUES(7,'Samsung');
INSERT INTO Brands(Code,Name) VALUES(8,'LG');
INSERT INTO Brands(Code,Name) VALUES(9,'Dell');

INSERT INTO Products(Code,Name,Price,Brand) VALUES(1,'Hard drive',240,5);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(2,'Memory',120,6);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(3,'ZIP drive',150,4);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(4,'Floppy disk',5,6);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(5,'Monitor',240,1);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(6,'DVD drive',180,2);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(7,'CD drive',90,2);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(8,'Printer',270,3);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(9,'Toner cartridge',66,3);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(10,'DVD burner',180,2);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(11,'Curve Monitor',350,7);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(12,'Flat large Monitor',360,8);
INSERT INTO Products(Code,Name,Price,Brand) VALUES(13,'notebook',720,9);
```




---

### Table 1: Brands
   ```sql
   SELECT *
    FROM Brands;
    ```

| Code | Name            |
| ---- | --------------- |
| 1    | SONY            |
| 2    | Creative Labs   |
| 3    | Hewlett-Packard |
| 4    | Iomega          |
| 5    | Fujitsu         |
| 6    | Winchester      |
| 7    | Samsung         |
| 8    | LG              |
| 9    | Dell            |



### Table 1: Products

```sql
SELECT *
    FROM Products;
```

| Code | Name               | Price | Brand |
| ---- | ------------------ | ----- | ----- |
| 1    | Hard drive         | 240   | 5     |
| 2    | Memory             | 120   | 6     |
| 3    | ZIP drive          | 150   | 4     |
| 4    | Floppy disk        | 5     | 6     |
| 5    | Monitor            | 240   | 1     |
| 6    | DVD drive          | 180   | 2     |
| 7    | CD drive           | 90    | 2     |
| 8    | Printer            | 270   | 3     |
| 9    | Toner cartridge    | 66    | 3     |
| 10   | DVD burner         | 180   | 2     |
| 11   | Curve Monitor      | 350   | 7     |
| 12   | Flat large Monitor | 360   | 8     |
| 13   | notebook           | 720   | 9     |
