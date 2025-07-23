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


[clickhouse-driver](https://clickhouse-driver.readthedocs.io/en/latest/installation.html)
