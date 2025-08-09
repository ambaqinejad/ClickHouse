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