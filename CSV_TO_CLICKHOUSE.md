```sql
clickhouse-local --query
"DESC file('OBC_1.csv', 'CSVWithNames')" |
awk 'BEGIN {print "CREATE TABLE my_table ("} {print "    `" $1 "` " $2 ","} END {print ")
ENGINE = MergeTree ORDER BY tuple();"}'
```

![allow null](./allow null type inferring.png)

### Better Solution
```sql
CREATE TABLE satellite.OBC_1 ENGINE = File(CSVWithNames)
AS SELECT * FROM file('OBC_1.csv', CSVWithNames)"

```

### Transform
```sql
CREATE TABLE satellite.t_obc
ENGINE = MergeTree
ORDER BY tuple() AS
SELECT * REPLACE(
    parseDateTimeBestEffort(concat(OBC_Date, ' ', OBC_Time)) AS ts1
)
FROM satellite.obc;
```

```sql
CREATE TABLE satellite.t_obc1
ENGINE = MergeTree
ORDER BY tuple() AS
SELECT
    *,
    parseDateTimeBestEffort(concat(OBC_Date, ' ', OBC_Time)) AS ts1
FROM satellite.obc;

```


### Creating Function to Transform separated Date and Time to One Column
```sql
CREATE OR REPLACE FUNCTION createDateTime AS (d, t) ->
    parseDateTimeBestEffortUS(
        formatDateTime(parseDateTimeBestEffortUS(nullIf(d, '')), '%Y-%m-%d') || ' ' || t
    );

SELECT createDateTime(OBC_Date, OBC_Time) AS ts1
FROM satellite.obc
LIMIT 20;
```

### Solving the problem of NAT and NAN in Date and Time
```sql
CREATE OR REPLACE FUNCTION createDateTime AS (d, t) ->
    parseDateTimeBestEffortOrNull(
        concat(
            nullIf(d, 'NaT'),   -- turns 'NaT' into NULL
            ' ',
            nullIf(t, 'NaN')    -- turns 'NaN' into NULL
        )
    );


SELECT *,
       createDateTime(OBC_Date, OBC_Time) AS ts1
FROM satellite.obc;
```

### Create a new table with ts as primary key
```sql
CREATE TABLE satellite.obc_with_ts
ENGINE = MergeTree()
PRIMARY KEY ts1
ORDER BY ts1
SETTINGS allow_nullable_key = 1
AS
SELECT *,
       createDateTime(OBC_Date, OBC_Time) AS ts1
FROM satellite.obc;

```

### Follow Insert, Update, Delete from source table
```sql
CREATE TABLE satellite.obc_with_ts_crud
ENGINE = ReplacingMergeTree(version)
PRIMARY KEY ts1
ORDER BY ts1
SETTINGS allow_nullable_key = 1
AS
SELECT *,
       createDateTime(OBC_Date, OBC_Time) AS ts1,
       1 AS version,
       0 AS is_deleted
FROM satellite.obc;
-----------------------------------
CREATE MATERIALIZED VIEW satellite.mv_obc_with_ts_crud
TO satellite.obc_with_ts_crud
AS
SELECT *,
       createDateTime(OBC_Date, OBC_Time) AS ts1,
       1 AS version,
       0 AS is_deleted
FROM satellite.obc;
------------------------------------
```
