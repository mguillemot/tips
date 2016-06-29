# PostgreSQL

## Types

See the type of an expression:

`SELECT pg_typeof(expression)`

## Indexed domain search on email column

Use an expression index on the reversed email field:

`CREATE INDEX ix_email_reverse ON accounts (REVERSE(email)`

And then LIKE can use the index for a suffix search:

`SELECT * FROM accounts WHERE REVERSE(email) LIKE REVERSE('%@domain.cmo')`
