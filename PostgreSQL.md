# PostgreSQL

## Group by day/hour

```PLpgSQL
SELECT DATE_TRUNC('hour', created_at), COUNT(*) FROM table GROUP BY DATE_TRUNC('hour', created_at);
```

## Types

See the type of an expression:

```PLpgSQL
SELECT pg_typeof(expression);
```

## Indexed domain search on email column

Use an expression index on the reversed email field:

```PLpgSQL
CREATE INDEX ix_email_reverse ON accounts (REVERSE(email);
```

And then LIKE can use the index for a suffix search:

```PLpgSQL
SELECT * FROM accounts WHERE REVERSE(email) LIKE REVERSE('%@domain.cmo');
```

## Disk size of all tables

```PLpgSQL
SELECT *, pg_size_pretty(total_bytes) AS total
        , pg_size_pretty(index_bytes) AS index
        , pg_size_pretty(toast_bytes) AS toast
        , pg_size_pretty(table_bytes) AS table
  FROM (
    SELECT *, total_bytes - index_bytes - COALESCE(toast_bytes, 0) AS table_bytes FROM (
      SELECT c.oid,
             nspname AS table_schema,
             relname AS table_name,
             c.reltuples AS row_estimate,
             pg_total_relation_size(c.oid) AS total_bytes,
             pg_indexes_size(c.oid) AS index_bytes,
             pg_total_relation_size(reltoastrelid) AS toast_bytes
          FROM pg_class c
          LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
          WHERE relkind = 'r'
  ) a
) a ORDER BY total_bytes DESC;
```

## Rename a hstore key

```PLpgSQL
CREATE OR REPLACE FUNCTION
change_hstore_key(h hstore, from_key text, to_key text) RETURNS HSTORE AS $$
BEGIN
    IF h ? from_key THEN
        RETURN (h - from_key) || hstore(to_key, h -> from_key);
    END IF;
    RETURN h;
END
$$ LANGUAGE plpgsql;
```
