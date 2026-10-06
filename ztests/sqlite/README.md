# SQLite

The [ztest](https://github.com/superdb/super/blob/main/ztest/ztest.go) YAML
definitions found below this directory are derived from the corresponding
[sqllogictest](https://sqlite.org/sqllogictest/doc/trunk/about.wiki) scripts
from [SQLite](https://sqlite.org/) found
[here](https://sqlite.org/sqllogictest/dir?ci=tip&name=test). They are
intended to test for [SuperDB](https://superdb.org/)'s backward compatibility with
[PostgreSQL](https://www.postgresql.org/).

## Trimmed Query Set

The queries included in these ztests are trimmed down from the full set
provided with SQLite. The trimming process is as follows:

1. Only the `select[1-5]` and `random` sets are targeted, since SuperDB does
not currently support SQL features exercised by the other test sets (e.g.,
indexing, views, etc.)

2. DDL/DML statements at the top of the sqllogictest scripts (e.g.,
`CREATE TABLE` and `INSERT INTO`) have been previously executed in PostgreSQL
and the resulting tables dumped to Parquet files (as described in the
[top-level README](../../README.md#details)) to be used as query inputs.

3. Any queries hinted with `skipif postgresql`, hinted with `onlyif` for a
non-PostgreSQL system, or otherwise confirmed to be
[rejected by PostgreSQL](https://sqlite.org/forum/forumpost/1b2bbec864)
are not included.

This remaining 3,619,718 total queries are captured in `.yaml` or `.fail`
ztest files below this directory.
