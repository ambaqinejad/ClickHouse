# Analyzing Data
![Common Table Expression](./CTE.PNG)

```sql
select count() as count, town
from uk_price_paid
GROUP BY town
ORDER BY count DESc
Format JSONEachROW

```

```sql
WITH most_expensive AS
(
    select town, sum(price) as sum
    from uk_price_paid
    GROUP BY town
    ORDER BY sum Desc 
    LIMIT 10
)
select avg(sum)
from most_expensive
```

```sql
select any(town),
    district,
    count() as c
    from uk_price_paid
    group by district
    order by c desc
    limit 20
```

```sql
WITH now() as today
select today - Interval 1 week
```

## USER DEFINED FUNCTION
```sql
-- USER DEFINED FUNCTIONS

Create FUNCTION mergeAddress as 
(addr1, addr2, street, town, country) -> concat(addr1, addr2, street, town, country)

select mergeAddress(addr1, addr2, street, town, county) from uk_price_paid
```

### ClickHouse Developer On-demand: Module 5
[https://learn.clickhouse.com/learner_module/show/1328915?lesson_id=7481737&section_id=81192209](https://learn.clickhouse.com/learner_module/show/1328915?lesson_id=7481737&section_id=81192209)

**Introduction**:  In this lab, you will write some queries to analyze the UK property prices dataset. Keep in mind that the UK uses pounds for currency, and each row in the table represents a transaction when a piece of property was sold.


Find all properties that sold for more than 100,000,000 pounds, sorted by descending price.

```sql
select * from uk_price_paid
WHERE price > 100000000
order by price desc
```

How many properties were sold for over 1 million pounds in 2022?

```sql
select count() as sold_over_one_mil_in_2022
FROM uk_price_paid
where price > 1000000 and toYear(date) == 2022
```

How many unique towns are in the dataset?

```sql
select uniqExact(town) from uk_price_paid
```

Which town had the highest number of properties sold?

# Lab 5.1: Writing Queries
[https://learn.clickhouse.com/learner_module/show/1328915?lesson_id=7481737&section_id=81192209](https://learn.clickhouse.com/learner_module/show/1328915?lesson_id=7481737&section_id=81192209)


Find all properties that sold for more than 100,000,000 pounds, sorted by descending price.

```sql
select * from uk_price_paid
WHERE price > 100_000_000
ORDER BY price DESC
```

How many properties were sold for over 1 million pounds in 2022?

```sql
SELECT count() from uk_price_paid
WHERE 
    price > 1_000_000 AND
    toYear(date) == '2022'

36475
```

How many unique towns are in the dataset?

```sql
select uniq(town) from uk_price_paid

1172
```

Which town had the highest number of properties sold?

```sql
select count() as c, town
from uk_price_paid
GROUP BY town
ORDER BY c DESC
limit 1

2188031 LONDON
```

Using the topK function, write a query that returns the top 10 towns that are not London with the most properties sold.

```sql
select topKIf(10)(town, town != 'London')
from uk_price_paid
WHERE town != 'London'
```

What are the top 10 most expensive towns to buy property in the UK, on average?

```sql
SELECT town, avg(price) as avgPrice from uk_price_paid
GROUP BY town
ORDER BY avgPrice DESC
limit 10
```

What is the address of the most expensive property in the dataset? (Specifically, return the addr1, addr2, street and town columns.)

```sql
Create function mergeAddr as 
(addr1, addr2, street, town) -> concat(addr1, addr2, street, town)

select mergeAddr(addr1, addr2, street, town) as addr
from uk_price_paid
order by price desc 
limit 1
```

Write a single query that returns the average price of properties for each type. The distinct values of type are detached, semi-detached, terraced, flat, and other.

```sql
select avg(price), type
from uk_price_paid
group by type
```

What is the sum of the price of all properties sold in the counties of Avon, Essex, Devon, Kent, and Cornwall in the year 2020?

```sql
SELECT
    formatReadableQuantity(sum(price))
FROM uk_price_paid
WHERE
    county IN ['AVON','ESSEX','DEVON','KENT','CORNWALL']
    AND toYear(date) == '2020'

29.94 billion
```

What is the average price of properties sold per month from 2005 to 2010?

```sql
SELECT 
    avg(price) as soldPerMonth,
    toStartOfMonth(date) as month
FROM uk_price_paid
WHERE 
    date >= toDate('2005-01-01') AND 
    date <= toDate('2010-12-31')
GROUP BY month 
```

How many properties were sold in Liverpool each day in 2020?
```sql
SELECT 
    count() as c,
    toStartOfDay(date) as day
FROM uk_price_paid
WHERE 
    toYear(date) == '2020' AND
    town == 'LIVERPOOL'
GROUP BY day 
ORDER by day asc
```