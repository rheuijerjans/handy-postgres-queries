# The database size

The size of the database is an important factor in database performance.
When the data is smaller then relatively more of it can be kept in cache (memory), speeding up your queries.

## Total database size
```sql
select pg_size_pretty(pg_database_size('your_database_name'))
```

## Table disk space
To get an idea about the size of the tables execute the query below. It also shows how much space is occupied by the actual data, and how much by the indexes on the table.

Disk space per table:
```sql
SELECT *
     , pg_size_pretty(total_bytes) AS total
     , pg_size_pretty(index_bytes) AS INDEX
     , pg_size_pretty(toast_bytes) AS toast
     , pg_size_pretty(table_bytes) AS TABLE
FROM (
         SELECT *, total_bytes - index_bytes - COALESCE(toast_bytes, 0) AS table_bytes
         FROM (
                  SELECT c.oid
                       , nspname                               AS table_schema
                       , relname                               AS TABLE_NAME
                       , c.reltuples                           AS row_estimate
                       , pg_total_relation_size(c.oid)         AS total_bytes
                       , pg_indexes_size(c.oid)                AS index_bytes
                       , pg_total_relation_size(reltoastrelid) AS toast_bytes
                  FROM pg_class c
                           LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
                  WHERE relkind = 'r'
              ) a
     ) a order by total_bytes desc;
```
[source: Postgresql wiki Disk Usage](https://wiki.postgresql.org/wiki/Disk_Usage)

## Index disk space

Big indexes tend to be slower than small indexes. 

```sql
SELECT
    t.schemaname,
    t.tablename,
    indexname,
    c.reltuples AS num_rows,
    pg_size_pretty(pg_relation_size(quote_ident(t.schemaname)::text || '.' || quote_ident(t.tablename)::text)) AS table_size,
    pg_size_pretty(pg_relation_size(quote_ident(t.schemaname)::text || '.' || quote_ident(indexrelname)::text)) AS index_size,
    CASE WHEN indisunique THEN 'Y'
        ELSE 'N'
    END AS UNIQUE,
    number_of_scans,
    tuples_read,
    tuples_fetched
FROM pg_tables t
LEFT OUTER JOIN pg_class c ON t.tablename = c.relname
LEFT OUTER JOIN (
    SELECT
        c.relname AS ctablename,
        ipg.relname AS indexname,
        x.indnatts AS number_of_columns,
        idx_scan AS number_of_scans,
        idx_tup_read AS tuples_read,
        idx_tup_fetch AS tuples_fetched,
        indexrelname,
        indisunique,
        schemaname
    FROM pg_index x
    JOIN pg_class c ON c.oid = x.indrelid
    JOIN pg_class ipg ON ipg.oid = x.indexrelid
    JOIN pg_stat_all_indexes psai ON x.indexrelid = psai.indexrelid
) AS foo ON t.tablename = foo.ctablename AND t.schemaname = foo.schemaname
WHERE t.schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY 1,2;
```
[source: Postgresql wiki Index Maintanance](https://wiki.postgresql.org/wiki/Index_Maintenance)


# Disk space used by a column

Disk space use of a specific column. Note that this query may take a while because it does a table scan.

For types with a fixed size like `bool`, `enum` or `int` you can calculate the total size yourself by multiplying the size with the number of rows. 

Column size:
```sql
select pg_size_pretty(pg_relation_size('your_schema.your_table')) as total_table_size,
       pg_size_pretty(sum(pg_column_size(your_column)))           as total_column_size,
       avg(pg_column_size(your_column))                           as average_column_size_bytes,
       sum(pg_column_size(your_column)) * 100.0 /
       pg_relation_size('your_schema.your_table')                 as percentage
from your_table;
```
