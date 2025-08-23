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
