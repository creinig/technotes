# PostgreSQL Diagnostics Queries

## SQL statements by execution time (requires pg_stat_statements extension!)

```sql
SELECT query, calls, total_time, max_time, mean_time, "rows"
FROM pg_stat_statements
WHERE (dbid IN (SELECT OID FROM pg_database WHERE datname = current_database()))
ORDER BY total_time DESC;
```

## Active Sessions

```sql
SELECT pid,usename,application_name,client_addr,client_hostname,query_start,
       (NOW() - query_start) AS query_age,wait_event_type,wait_event,state,query
  FROM pg_stat_activity
  WHERE datname = 'mydatabase' AND   state <> 'idle';
```

## Database Statistics

```sql
select *, pg_size_pretty(temp_bytes) from pg_stat_database where (datname = current_user);
```

## Database sizes

Run as superuser for info on all databases.

```sql
select t1.datname AS db_name, 
       pg_size_pretty(pg_database_size(t1.datname)) as db_size
from pg_database t1
order by pg_database_size(t1.datname) desc;
```

## Schema sizes

```sql
WITH schemas AS (
    SELECT
        nspname,
        sum(pg_relation_size(pg_class.oid)) AS schema_size,
        sum(pg_total_relation_size(reltoastrelid)) AS toast_size
    FROM
        pg_class
        INNER JOIN pg_namespace ON relnamespace = pg_namespace.oid
    GROUP BY
        nspname
)
SELECT
    nspname,
    pg_size_pretty(schema_size) as schema_size,
    pg_size_pretty(toast_size) as toast_size
FROM
    schemas
ORDER BY
    (schema_size + toast_size) DESC;
```


## Current Conflicts

```sql
select * from pg_stat_database_conflicts where (datname = current_user);
```

## Table Size, Dead Rows & (Auto)Vacuum, partitioning safe

```sql
SELECT schemaname as "schema", relbasename as "table", count(*) as npart,
    sum(n_live_tup) as nlivetuples, sum(n_dead_tup) as ndeadtuples,
    max(last_autovacuum) as last_autovacuum, max(last_vacuum) as last_vacuum,
    pg_size_pretty(sum(total_bytes)) AS total_bytes,
    pg_size_pretty(sum(index_bytes)) AS index_bytes,
    pg_size_pretty(sum(toast_bytes)) AS toast_bytes
from (
    SELECT t.schemaname, t.relname, regexp_replace(t.relname, '_(p[0-9]+.*|default)$', '') as relbasename,
        t.n_live_tup, t.n_dead_tup, t.last_autovacuum, t.last_vacuum,
        pg_total_relation_size(c.oid) AS total_bytes,
        pg_indexes_size(c.oid) AS index_bytes,
        pg_total_relation_size(reltoastrelid) AS toast_bytes
    FROM pg_stat_user_tables t, pg_class c, pg_namespace n
    where (n.oid = c.relnamespace)
      and (c.relkind = 'r')
      and (n.name in ('public'))
      and (t.schemaname = n.nspname)
      and (t.relname = c.relname)
) details
group by schemaname, relbasename
ORDER BY sum(n_dead_tup)
    / (sum(n_live_tup)
       * current_setting('autovacuum_vacuum_scale_factor')::float8
          + current_setting('autovacuum_vacuum_threshold')::float8)
     DESC,
     sum(total_bytes) desc;
```

## Analyze all tables in a given schema

From https://stackoverflow.com/a/42689285/1814922

```sql
DO $$
DECLARE
  tab RECORD;
  schemaName VARCHAR := 'your_schema';
BEGIN
  for tab in (select t.relname::varchar AS table_name
                FROM pg_class t
                JOIN pg_namespace n ON n.oid = t.relnamespace
                WHERE t.relkind = 'r' and n.nspname::varchar = schemaName
                order by 1)
  LOOP
    RAISE NOTICE 'ANALYZE %.%', schemaName, tab.table_name;
    EXECUTE format('ANALYZE %I.%I', schemaName, tab.table_name);
  end loop;
end
$$;
```

## Progress of running vacuum jobs

```sql
-- run as superuser to get meaningful info!
SELECT v.pid, v.datname, relid, phase,
       heap_blks_total, heap_blks_scanned, heap_blks_vacuumed,
       ((heap_blks_scanned*1.0) / heap_blks_total)*100 AS "scan%",
       ((heap_blks_vacuumed*1.0) / heap_blks_total)*100 AS "vac%",
       a.query
FROM pg_stat_progress_vacuum v, pg_stat_activity a
WHERE (v.pid = a.pid);
```

## SSL information on current connections

```sql
select s.*, a.usename, a.application_name, a.client_addr, a.backend_type from pg_stat_ssl s, pg_stat_activity a where (s.pid = a.pid);
```

## Grant / Permission / User diagnostics

psql metacommands:

* `\dp` : List privileges
* `\ddp` : list default privileges

List all users and their privileges for a table (from https://stackoverflow.com/a/42108936/1814922):

```sql
select a.schemaname, a.tablename, b.usename,
      HAS_TABLE_PRIVILEGE(usename, quote_ident(schemaname) || '.' || quote_ident(tablename), 'select') as has_select,
      HAS_TABLE_PRIVILEGE(usename, quote_ident(schemaname) || '.' || quote_ident(tablename), 'insert') as has_insert,
      HAS_TABLE_PRIVILEGE(usename, quote_ident(schemaname) || '.' || quote_ident(tablename), 'update') as has_update,
      HAS_TABLE_PRIVILEGE(usename, quote_ident(schemaname) || '.' || quote_ident(tablename), 'delete') as has_delete, 
      HAS_TABLE_PRIVILEGE(usename, quote_ident(schemaname) || '.' || quote_ident(tablename), 'references') as has_references 
    from pg_tables a, pg_user b 
where a.schemaname = 'your_schema_name' and a.tablename='your_table_name';
```

