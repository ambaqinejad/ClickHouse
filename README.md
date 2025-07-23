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





[clickhouse-driver](https://clickhouse-driver.readthedocs.io/en/latest/installation.html)
