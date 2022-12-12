---
title: "MYSQL Portfolio: MySQL subqueries 'With ...AS' example"
Date: 2022-12-12
tags: [MYSQL]
header:
  image: "/images/mysql.jpg"
excerpt: "MYSQL"
mathjax: "true"
---


## Problem :
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

