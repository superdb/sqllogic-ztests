# sqllogic-ztests

This repository is a collection of
[ztests](https://github.com/superdb/super/blob/main/ztest/ztest.go)
that contain queries derived from
[sqllogictest](https://sqlite.org/sqllogictest/doc/trunk/about.wiki)
scripts. They're intended primarily for use as
[SuperDB](https://superdb.org/) regression tests.

Currently all tests in this repo are based on the sqllogictests from
[SQLite](https://sqlite.org/). More details on these tests can be found
[here](ztests/sqlite/README.md).

## Usage

Checkout this repo as a sibling directory alongside your checkout of the
[super repo](https://github.com/superdb/super), e.g.:

```
$HOME/sqllogic-ztests
$HOME/super
```

Then within the checkout for this repo, do:

```
make test-sqllogic
```

That will run all sqllogictests below `ztests/` using the last built binary at
`dist/super` in your super repo checkout.

You can also run just a single sqllogic ztest, e.g.:

```
make TEST=TestSPQ/ztests/sqlite/select1/q0
```

or groups of ztests by directory, e.g.:

```
make TEST=TestSPQ/ztests/sqlite/select[1-5]
```

In fact, that last one may prove handy if you want a quick "smoke test" on a
super branch, since those ~5000 tests run in just a few minutes and provide a
decent regression suite. Meanwhile, if you run 100% of the available tests
you'll also get the 3+ million fuzz-style ztests below `sqlite/random/`,
which do provide wider coverage but also take hours to complete.

## Details

Test files ending in `.yaml` contain queries known to succeed in recent
SuperDB versions and are executed nightly via a
[GitHub Actions workflow](.github/workflows/test-sqllogic.yaml) to catch
regressions. Queries known to fail in recent SuperDB versions are held in
files ending in `.fail` and are skipped in the nightly CI run. Each `.fail`
file contains a comment describing the root cause of the failure, which is
typically a link to one of the open [issues in the super repo](https://github.com/superdb/super/issues).

As SuperDB does not yet support SQL [DDL](https://en.wikipedia.org/wiki/Data_definition_language)
or [DML](https://en.wikipedia.org/wiki/Data_manipulation_language),
each test subdirectory includes a set of input files in
[Parquet](https://parquet.apache.org/) format that the queries treat as
tables. The numeric ztest YAML filenames in each test subdirectory follow the
order of the eligible test queries (i.e., not DDL/DML statements) as they appeared in the
corresponding sqllogictest scripts, e.g., `q0.yaml` is the first `SELECT` query,
`q1.yaml` is the next, etc.

The modifications in [`super.patch`](super.patch) must currently be applied
to your checkout of the super repo before running `make build`.

```
patch -d super -p1 < sqllogic-ztests/super.patch
make -C super build
```

This is a temporary fix to produce TSV output from `super` that is `diff`-able
against the expected sqllogictest outputs. Patching will not be necessary once
[super/5961](https://github.com/superdb/super/issues/5961) and
[super/6381](https://github.com/superdb/super/issues/6381) are addressed.

## Improving Git Performance

Since there's 3+ million files in this repo, local Git operations on its 
contents can be painfully slow. To improve on this, it's been found that
executing the following inside your local clone of the repo may improve things
significantly:

```
git config feature.manyFiles true
git config core.fsmonitor true
git fsmonitor--daemon start
git maintenance run
```

On my Intel-based Macbook, this brings the time to run `git status` down from
~40 seconds to ~1 second.

More notes regarding these commands:

* For the `core.fsmonitor true` config to have effect, that
  [`fsmonitor--daemon`](https://git-scm.com/docs/git-fsmonitor--daemon) must be
  running, which monitors the working directory for changes.

* The `git maintenance run` is a "one shot" optimization pass, so should be
  re-run periodically to get the best possible performance.
