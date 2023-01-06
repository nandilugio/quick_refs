ALTER TABLE (almost all, see https://www.postgresql.org/docs/current/sql-altertable.html)
  - acquires AccessExclusive lock (1) on the table

  ADD FOREIGN KEY
    - requires only a SHARE ROW EXCLUSIVE lock, though on both tables

  ADD COLUMN
    - Without DEFAULT
      - Fast: no default to write
      - Subsequent ALTER COLUMN SET DEFAULT
        - Fast: doesn't write the default
    - With DEFAULT
      - Fast: "writes" default as metadata

  DROP COLUMN
    - Fast: metadata change. Disk changes are done on VACUUM

  ADD table_constraint NOT VALID
    - Skips lengthy validation of values
    - Requires subsequent VALIDATE CONSTRAINT (after backfilling data)
    - Only for foreign key and CHECK constraints

1) Table AccessExclusive lock issue
  Problem:
    - From http://www.databasesoup.com/2013/11/alter-table-and-downtime-part-ii.html
      - Bad luck: a long transaction accesses the table with, say, an easy AccessShare lock.
      - The AccessExclusive lock is requested (by eg. an ALTER TABLE). It's queued behind the above.
      - Other reads (AccessShare lock) conflict with the AccessExclusive lock, piling up in the queue too.
        - Requests start timing out, etc.
  Mitigation:
    - From https://gocardless.com/blog/zero-downtime-postgres-migrations-the-hard-parts/
      - Eliminate long-running queries/transactions from your application.
        - Run analytics queries against an asynchronously updated replica.
        - It's worth setting `log_min_duration_statement` and `log_lock_waits` to find these issues in your app before they turn into downtime.
      - Set lock_timeout in your migration scripts to a pause your app can tolerate. It's better to abort a deploy than take your application down.
      - Split your schema changes up.
        - Problems become easier to diagnose.
        - Transactions around DDL are shorter, so locks aren't held so long.
      - Keep Postgres up to date. The locking code is improved with every release.






TODO
====

- From https://gocardless.com/blog/zero-downtime-postgres-migrations-the-hard-parts/
  - Don't rename columns/tables which are in use by the app - always copy the data and drop the old one once the app is no longer using it
  - Don't rewrite a table while you have an exclusive lock on it (e.g. no ALTER TABLE foos ADD COLUMN bar varchar DEFAULT 'baz' NOT NULL)
  - Don't perform expensive, synchronous actions while holding an exclusive lock (e.g. adding an index without the CONCURRENTLY flag)

- From https://www.braintreepayments.com/blog/safe-operations-for-high-volume-postgresql/

- From gems https://github.com/gocardless/nandi, https://github.com/ankane/strong_migrations, https://github.com/braintree/pg_ha_migrations/, https://github.com/doctolib/safe-pg-migrations (quick comparison: https://gocardless.com/blog/fear-free-postgresql-migrations-for-rails/#appendix-why-not-use--)




