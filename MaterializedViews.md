# Materialized Views

## Normal View
```sql
Create VIEW avg_price_by_type AS
    SELECT avg(price), type
    FROM uk_price_paid
    GROUP BY type

SELECT * from avg_price_by_type
```

![Materialized View](./MaterializedViews1.PNG)

## Lab 6.1: Defining a Regular View
https://learn.clickhouse.com/learner_module/show/1328927?lesson_id=7671000&section_id=67044210

Define a view on the uk_price_paid table that satisfies the following requirements:

a. The name of the view is london_properties_view 

b. The view only returns properties in the town of London

c. The view only returns the date, price, addr1, addr2 and street columns


```sql
Create View london_properties_view AS
    Select * from uk_price_paid
    where town == 'LONDON'
```

Using your view, compute the average price of properties sold in London in 2022.

```sql
Select avg(price) from london_properties_view
```

Count the number of rows in the view (which should be the number of properties sold in London).

```sql
Select count() from london_properties_view

2188031
```

Create a parameterized view named properties_by_town_view that is identical to your london_properties_view, except instead of filtering by the town equal to 'LONDON', the value of town is defined as a parameter named town_filter.

```sql
Create View properties_view AS
    Select * from uk_price_paid
    where town == {town_filter:String}
```

Write a query using your properties_by_town_view that returns the most expensive property sold in Liverpool, along with the name of the street that the property is on. You should get back a 300M pound property on Sefton Street.

```sql
Select max(price), argMax(street, price)
    from properties_by_town_view(town_filter='LIVERPOOL')
```

## Lab 6.2: Materialized Views
https://learn.clickhouse.com/learner_module/show/1328927?lesson_id=7671000&section_id=67044210

Write a single query on the uk_price_paid table that computes the number of properties sold and the average price of all the properties sold for the year 2020. Notice your query needs to process all the rows in the table.

```sql
SELECT count() as c, avg(price)
    from uk_price_paid
    WHERE toYear(date) == '2020'
```

| count | average |
|-------|---------|
|886642|378060.000030452|


Write a similar query as the one you wrote in step 1, except this time return the year, count and average for all the years in the dataset. (In other words, group the result by toYear(date) instead of filtering by the year 2020). Again, your query will need to process all the rows in the table.

```sql
Select toYear(date), count(), avg(price)
    from uk_price_paid
    GROUP BY toYear(date)
```

Suppose you want to run queries frequently on the yearly historical data of uk_price_paid. Let's define a materialized view that partitions the data by year and sorts the data by town, so that our queries do not need to scan every row each time we run our queries. Let's start by defining the destination table. Define a new MergeTree table that satisfies the following requirements:

a. The name of the table is prices_by_year_dest

b. The table will store the date, price, addr1, addr2, street, town, district and county columns from uk_price_paid

c. The primary key is the town column followed by the date column

d. The table is partitioned by year

```sql
Create table prices_by_year_dest(
    price UInt32,
    date Date,
    addr1 String,
    addr2 String,
    street LowCardinality(String),
    town LowCardinality(String),
    district LowCardinality(String),
    county LowCardinality(String)
)
ENGINE = MergeTree
Primary KEY(town, date)
PARTITION BY toYear(date)
```

Create a materialized view named prices_by_year_view that sends the date, price, addr1, addr2, street, town, district and county columns to the prices_by_year_dest table.

```sql
Create Materialized VIEW prices_by_year_view
TO prices_by_year_dest
AS
    Select  
        price,
        date,
        addr1,
        addr2,
        street,
        town,
        district,
        county
    FROM uk_price_paid

There is no record
```

Populate the prices_by_year_dest table with all of the existing rows in uk_price_paid.

```sql
INSERT INTO prices_by_year_dest
    SELECT 
        price,
        date,
        addr1,
        addr2,
        street,
        town,
        district,
        county
    FROM uk_price_paid
```

Count the number of rows in prices_by_year_dest and verify it's the same number of rows in uk_price_paid - 28,634,236 rows.

```sql
SELECT count() FROM prices_by_year_dest

SELECT count() FROM uk_price_paid
```

Run the following query, which returns the parts that were created for your prices_by_year_dest table. You will see lots of parts, and folder names contain the year

```sql
SELECT * FROM system.parts
WHERE table='prices_by_year_dest';
```

For comparison, notice that uk_price_paid probably only has 1 or 2 parts:

```sql
SELECT * FROM system.parts
WHERE table='uk_price_paid';
```

Notice that partitioning by year created a lot of parts. At a minimum, you need at least one part for each year from 1995 to 2023, but it is possible that some of those years have multiple part folders. This is a cautionary tale about partitioning! Be careful with it - especially when you only have 28M rows. There is really no need for us to partition this dataset by year except for educational purposes. Only for very large datasets do we recommend partitioning, in which case partitioning by month is recommended.


Let's see if we gained any benefits from defining this materialized view. Run the same query from step 1, except this time run it on prices_by_year_dest instead of uk_prices_paid. How many rows were scanned?

```sql
SELECT count() as c, avg(price)
    from prices_by_year_dest
    WHERE toYear(date) == '2020'
```

Use prices_by_year_dest to count how many properties were sold and the maximum, average, and 90th quantile of the price of properties sold in June of 2005 in the county of Staffordshire.

```sql
SELECT 
    count(), 
    max(price), 
    min(price), 
    avg(price), 
    quantile(0.90)(price)
    FROM prices_by_year_dest
WHERE county = 'STAFFORDSHIRE'
    AND date >= toDate('2005-06-01') AND date <= toDate('2005-06-30');
```
|count|max|min|avg|min|
|-----|---|---|---|---|
|1322|745000|23000|160241.94402420576|269670.00000000006|

Let's verify that the insert trigger for your materialized view is working properly. Run the following command, which inserts 3 rows into uk_price_paid for properties in the year 2024. (Right now your uk_price_paid table doesn't contain any transactions from 2024.)

```sql
INSERT INTO uk_price_paid VALUES
    (125000, '2024-03-07', 'B77', '4JT', 'semi-detached', 0, 'freehold', 10,'',	'CRIGDON','WILNECOTE','TAMWORTH','TAMWORTH','STAFFORDSHIRE'),
    (440000000, '2024-07-29', 'WC1B', '4JB', 'other', 0, 'freehold', 'VICTORIA HOUSE', '', 'SOUTHAMPTON ROW', '','LONDON','CAMDEN', 'GREATER LONDON'),
    (2000000, '2024-01-22','BS40', '5QL', 'detached', 0, 'freehold', 'WEBBSBROOK HOUSE','', 'SILVER STREET', 'WRINGTON', 'BRISTOL', 'NORTH SOMERSET', 'NORTH SOMERSET');
```

Verify you have three new rows in prices_by_year_dest where the year is 2024.

```sql
SELECT * from prices_by_year_dest
    where toYear(date) == '2024'
```

You should also see a new part folder for prices_by_year_dest named 2024_0_0_0:

```sql
SELECT * FROM system.parts
WHERE table='prices_by_year_dest';
```


Solution is available [HERE](https://github.com/ClickHouse/clickhouse-academy/tree/main/developer/06_materialized_views).