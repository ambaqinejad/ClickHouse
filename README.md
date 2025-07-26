# ClickHouse

## Lab 1.4: Writing Ad-hoc Queries
[Tutorial Link](https://learn.clickhouse.com/learner_module/show/1328489?lesson_id=7667283&section_id=81140575)

We have about 2.3M crypto prices stored in a Parquet file in S3. In this lab, you are going to write queries on that file without ingesting the data into a ClickHouse table.

The crypto prices are saved in a Parquet file in S3. It's a public file and the URL is:

[https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet(opens in a new tab)](https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet)

1
Using either the command-line SQL client or the ClickHouse Cloud SQL Console, write a query that returns all columns of the first 100 rows using the s3 table function.

```sql
select * from
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet')
limit 100
```

Use the count() function to compute the number of rows in the Parquet file. You should get a count of 2,382,643 rows.

```sql
select count() as count from
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet')

```

There is a handy function named formatReadableQuantity that makes it easier to view large numbers. Pass the count() function into formatReadableQuantity to see how it works.

```sql
select formatReadableQuantity(count()) as count from
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet')

```

What is the average volume of Bitcoin trades?

```sql
select formatReadableQuantity(avg(volume)) as bitcoint_average_volume from
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet')
where crypto_name = 'Bitcoin'
```

Write a query that returns the number of trades for each cryptocurrency (i.e., for each value of crypto_name). Sort the results alphabetically by crypto_name.

```sql
select crypto_name, count() from
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet')
GROUP BY crypto_name
ORDER BY crypto_name ASC
```

Notice there are quite a few crypto names that start with one or more blank characters. The trim function would be useful here - it removes whitespace from Strings. Use trim and notice how it affects the results.

```sql
select trim(crypto_name), count() from
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet')
GROUP BY crypto_name
ORDER BY crypto_name ASC
```

## Lab 2.1: Understanding the Primary Keys in ClickHouse
[https://learn.clickhouse.com/learner_module/show/1328854?lesson_id=7790807&section_id=81141679](https://learn.clickhouse.com/learner_module/show/1328854?lesson_id=7790807&section_id=81141679)  

**A granule is a small block of stored data. Choosing the right primary key helps your database engine skip unnecessary granules during queries, which can greatly improve performance.**  

Run the following command, which shows the 16 column names and types that ClickHouse infers from this Parquet file:  
```sql
DESCRIBE s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/pypi/2023/pypi_0_7_34.snappy.parquet');

```

Write a query that returns only the first 10 rows of this file, which will give you an idea of what the dataset looks like.  

```sql
SELECT * from  
s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/pypi/2023/pypi_0_7_34.snappy.parquet') LIMIT 50;
```
How many rows are in the file? (You should get 1,692,671.)

```sql
SELECT count() from
s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/pypi/2023/pypi_0_7_34.snappy.parquet') LIMIT 50;
```

Create a table to store the PyPI data with the following specs: 

a. The table name is pypi

b. The table only has four columns: TIMESTAMP, COUNTRY_CODE, URL, and PROJECT. (If you are unsure of the column types, check the result of the DESCRIBE command above and ignore the Nullable part of it.)

c. The table engine is MergeTree

d. The primary key is the TIMESTAMP column

```sql
USE training;
CREATE TABLE pypi
(
    TIMESTAMP DateTime64(3, 'UTC'),
    COUNTRY_CODE Nullable(String),
    URL Nullable(String),
    PROJECT Nullable(String)
)
ENGINE = MergeTree
PRIMARY KEY (TIMESTAMP);


INSERT INTO pypi
SELECT TIMESTAMP, COUNTRY_CODE, URL, PROJECT
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/pypi/2023/pypi_0_7_34.snappy.parquet');

```


Let's see what happens when we add PROJECT to the primary key. Create the following table named pypi2, and notice that the only change from pypi is that PROJECT was added to the end of the primary key. Run all of the following commands and see what happens:

```sql

SELECT 
    PROJECT,
    count() AS c
FROM pypi2
WHERE PROJECT LIKE 'boto%'
GROUP BY PROJECT
ORDER BY c DESC;
```

## Lab 2.2: Primary Keys and Disk Storage
[https://learn.clickhouse.com/learner_module/show/1328854?lesson_id=7790807&section_id=81141679](https://learn.clickhouse.com/learner_module/show/1328854?lesson_id=7790807&section_id=81141679)

In this lab, you will see how a different primary key can affect the amount of disk space your table uses for storage. Why? Because how your data is sorted can affect how effectively the data can be compressed.

Run the following query, which queries the system.parts table and returns the amount of disk space used by your pypi table:
```sql
SELECT
formatReadableSize(sum(data_compressed_bytes)) as CompressedSize,
formatReadableSize(sum(data_uncompressed_bytes)) as UnComporessedSize,
count() as num_of_active_parts
from system.parts
WHERE(active = 1) and (table = 'pypi')
```

``` sql
SELECT
formatReadableSize(sum(data_compressed_bytes)) as CompressedSize,
formatReadableSize(sum(data_uncompressed_bytes)) as UnComporessedSize,
count() as num_of_active_parts
from system.parts
WHERE(active = 1) and (table LIKE '%pypi%')
GROUP BY table
```
Notice how much better compression you get from your pypi2 table - the compressed size of the pypi2 table is significantly smaller than pypi!

**See All Solutions**
[Soultions](https://github.com/ClickHouse/clickhouse-academy/tree/main/developer/02_clickhouse_architecture)

## Modeling Data
[https://learn.clickhouse.com/learner_module/show/1328860?lesson_id=7951150&section_id=81277855](https://learn.clickhouse.com/learner_module/show/1328860?lesson_id=7951150&section_id=81277855)

### Data Types
```sql
Create TABLE Employee (
    name String,
    hired Date,
    age UInt8
)
ENGINE = MergeTree
PRIMARY KEY (name, hired)

Alter Table Employee 
add Column Meetings Array(DateTime)

Show Create Table Employee
```

```sql
Insert INTO Employee Values
(
    'AmirHosein',
    '2025-06-16',
    28,
    ['2025-07-16 09:00:00']
)
(
    'Ali',
    '2025-06-16',
    28,
    [now(), now() - Interval 1 week]
)
```

**Making a Filed Nullable**
```sql
ALTER TABLE Employee Add COLUMN
my_test Nullable(UInt8)
```

**USING ENUMs**
```sql
ALTER TABLE Employee Add COLUMN
shirt_color Enum('Green' = 1, 'Red' = 2, 'White' = 3)

show create table Employee

Insert into Employee values(
    'Ali', '2005-12-12', 45, [], '', 'Green', 0
)

select * from Employee

INSERT INTO Employee (shirt_color) Values (3)

select * from Employee
```

**LowCardinality**
```sql
ALTER TABLE Employee add COLUMN 
position LowCardinality(String)

insert into Employee (position) Values ('Manager')

select * from Employee
```

### Note:
```sql
|    Order By and Primary Key doing the same thing!
```

![Primary Key can not be less than Order By](./primary%20key%20less%20than%20order%20by.PNG)

### Partitioning
![Partitioning](./partitioning.PNG)



## ClickHouse Developer On-demand: Module 3
[Tutorial Link](https://learn.clickhouse.com/learner_module/show/1328860?lesson_id=7951150&section_id=81277855)

Using the uniqExact function, write a query that returns the unique number of values for the COUNTRY_CODE column. 

```sql
select uniqExact(COUNTRY_CODE) from pypi
```

Using uniqExact again, how many unique values appear in the PROJECT column and the URL column?

```sql

```


Create a new table named pypi3 that is identical to pypi2 but uses LowCardinality for the COUNTRY_CODE and PROJECT columns. Insert all the rows from pypi2 into pypi3.

```sql
DESCRIBE pypi2

show create table pypi2

CREATE TABLE training.pypi3
(
    `TIMESTAMP` DateTime,
    `COUNTRY_CODE` LowCardinality(String),
    `URL` String,
    `PROJECT` LowCardinality(String)
)
PRIMARY KEY (TIMESTAMP, PROJECT)

insert into pypi3 select * from pypi2
```

Let's see how much disk space we saved:

```sql
SELECT
    table,
    formatReadableSize(sum(data_compressed_bytes)) AS compressed_size,
    formatReadableSize(sum(data_uncompressed_bytes)) AS uncompressed_size,
    count() AS num_of_active_parts
FROM system.parts
WHERE (active = 1) AND (table LIKE 'pypi%')
GROUP BY table;
```

Notice we went from 60.54MiB in pypi to 14.90MiB in pypi2 by sorting the data better, and now down to 13.98MiB in pypi3 by using LowCardinality. More importantly, our queries should run faster when filtering on the LowCardinality columns.

Let's see if the queries are faster. Run the following query on both pypi2 and pypi3. Which result is faster? 

```sql
SELECT
    toStartOfMonth(TIMESTAMP) AS month,
    count() AS count
FROM pypi2
WHERE COUNTRY_CODE = 'US'
GROUP BY
    month
ORDER BY
    month ASC,
    count DESC;



SELECT
    toStartOfMonth(TIMESTAMP) AS month,
    count() AS count
FROM pypi3
WHERE COUNTRY_CODE = 'US'
GROUP BY
    month
ORDER BY
    month ASC,
    count DESC;
```

Notice the queries are considerably faster on pypi3 (almost twice as fast)...and this is a small dataset! We could see big improvements in both compression and query performance doing this over billions and billions of rows. 

### Lab 3.2: Defining a MergeTree Table

Run the following command, which shows the data types that ClickHouse infers from the Parquet file:

```sql
DESCRIBE s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet');

```

Define a new table named crypto_prices that satisfies the following requirements:

- a. Use the column names and data types above, except: 

- i. do not use Nullable on any columns

- ii. change trade_date to a Date

- iii. use LowCardinality for the crypto_name

- b. The table engine is MergeTree

- c. The primary key is the crypto_name followed by the trade_date

```sql
DESCRIBE s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet');


Create Table crypto_prices (
    trade_date Date,
    crypto_name LowCardinality(String),
    volume Float32,
    price Float32,
    maket_cap Float32,
    change_1_day Float32
)
ENGINE = MergeTree
PRIMARY KEY (crypto_name, trade_date)
```

Insert all of the data from the parquet file in S3 into your new crypto_prices table.

```sql
INSERT INTO crypto_prices 
select * from 
s3('https://learnclickhouse.s3.us-east-2.amazonaws.com/datasets/crypto_prices.parquet');
```

Verify you have all the data by counting the number of rows (you should get 2,382,643 rows).

```sql
select count() from crypto_prices
```

Run a query that returns the count of rows where volume >= 1000_000. How many rows of the table were read to compute this result?

```sql

```

Using the avg function on the price column, find the average price of 'Bitcoin'. How many granules were processed, and why is it smaller than the previous query that filtered on the volume column?

```sql
select avg(price), crypto_name 
from crypto_prices
where crypto_name = 'Bitcoin'
GROUP BY crypto_name
```

What is the average price of all trades where the name of the cryptocurrency name starts with a capital 'B'? You can use the standard LIKE from SQL. Notice how many granules were processed. Clearly the primary index was also useful in this query.

```sql
select avg(price)
from crypto_prices
where crypto_name LIKE 'B%'
```