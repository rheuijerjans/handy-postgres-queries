# Cache hits

The cache hit ratio tells how much data the database reads from memory, and how much from disk.
A higher cache hit ratio means that more data comes from memory when sql is executed, 
resulting in lower query times.

Patterns I have observed:
- index cache hit ratio > table cache hit ratio
- cache hit ratio depends on data size and available memory
- access patterns that access more recent data have higher cache hit ratio
- access patters that that access 'old' data, like batch jobs, cause lower cache hit ratio 

## Cache hit ratio for all table data

```sql 
SELECT sum(heap_blks_read)                                             as heap_read,
       sum(heap_blks_hit)                                              as heap_hit,
       sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```
[source: Craig Kerstiens- Understanding Postgres Performance](http://www.craigkerstiens.com/2012/10/01/understanding-postgres-performance/)

## Drill down cache hit ratio per table
```sql
SELECT relname,
       schemaname,
       heap_blks_read                                   as heap_read,
       heap_blks_hit                                    as heap_hit,
       cast(heap_blks_hit as decimal)  / (heap_blks_hit + heap_blks_read) as ratio
FROM pg_statio_user_tables;
```

## Cache hit ratio for all index data
```sql
SELECT sum(idx_blks_read)                                           as idx_read,
       sum(idx_blks_hit)                                            as idx_hit,
       (sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit) as ratio
FROM pg_statio_user_indexes;
``` 
[source: Craig Kerstiens- Understanding Postgres Performance](http://www.craigkerstiens.com/2012/10/01/understanding-postgres-performance/)

## Drill down cache hit ratio per index
```sql
select indexrelname as index_name,
       (idx_blks_hit - idx_blks_read) / (idx_blks_hit ) as ratio
from pg_statio_user_indexes;
```

## Cache hit ratio per query
This query gives more insight in how the cache hit ratio affects query performance. 
```sql
SELECT query,
       calls,
       total_time,
       mean_time,
       cast (shared_blks_hit as decimal) / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_ratio
FROM pg_stat_statements
order by total_time desc;
```