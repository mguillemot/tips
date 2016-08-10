# PostgreSQL

## psql

Execute a SQL command from shell:

```
psql -U <user> -W -h localhost -d <db> -c 'SELECT NOW()'
```

- `-W` to force password auth.
- `-h localhost` to force connecting through the regular TCP interface, and use the right auth path.

## Backup schema with its content into another schema

```
pg_dump -n <schema> > schema.sql
vi schema.sql # edit occurences of schema in the header
psql -f schema.sql
```

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

## Import CSV data

Import local file in psql:

```PLpgSQL
\copy dest_table (a, b, c) FROM 'source.csv' DELIMITER ',' CSV;
```

Import file on the server:

```PLpgSQL
COPY dest_table (a, b, c) FROM 'source.csv' DELIMITER ',' CSV;
```

Concatenate/filter input files:

```PLpgSQL
COPY dest_table (a, b, c) FROM PROGRAM 'echo *.csv | grep something' DELIMITER ',' CSV;
```

## Pivot tables

*Require the `tablefunc` extension.*

```PLpgSQL
SELECT * FROM crosstab
(
  'SELECT created_at, measure, value FROM log_measures WHERE measure IN (''A'', ''B'', ''C'') ORDER BY 1,2'
)
AS
(
  d timestamptz,
  a float8,
  b float8,
  c float8
)
```

- The query to pivot need to be sent as raw text, not as subquery!
- Returned fields are sorted alphabetically with this method, assuming there are values for all pivots. If that's not the case, we need the 2-param form of `crosstab()`.