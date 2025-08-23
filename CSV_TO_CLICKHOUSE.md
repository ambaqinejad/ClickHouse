```sql
clickhouse-local --query
"DESC file('OBC_1.csv', 'CSVWithNames')" |
awk 'BEGIN {print "CREATE TABLE my_table ("} {print "    `" $1 "` " $2 ","} END {print ")
ENGINE = MergeTree ORDER BY tuple();"}'
```

![allow null](./allow null type inferring.png)
