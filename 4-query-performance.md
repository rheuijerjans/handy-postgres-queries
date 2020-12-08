# Query performance

The `pg_stat_statements` table gives great insight into query performance.
It is recommended to look at the `total_time` column to get an indication of total 
load on the database that a specific query causes.

I found this query very helpful to gain insight in what query is causing load on the database.
```sql
select query
     , round(total_time::numeric, 2)                                                                 as total_time_
     , calls
     , round(rows / calls::numeric, 2)                                                               as rows_per_call
     , round(mean_time::numeric, 2)                                                                  as mean
     , round((100 * total_time / sum(total_time::numeric) over ())::numeric, 2)                      as percentage_total_time
     , round(shared_blks_hit::numeric /
             nullif(pg_stat_statements.shared_blks_hit + pg_stat_statements.shared_blks_read, 0), 2) as cache_hit_ratio
from pg_stat_statements
order by total_time desc;
```
[Hans-Jürgen Schönig - PostgreSQL performance in 5 minutes (Video)](https://www.youtube.com/watch?v=5M2FFbVeLSs)

