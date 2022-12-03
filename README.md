# Wrappers

[![crates.io badge](https://img.shields.io/crates/v/supabase-wrappers.svg)](https://crates.io/crates/supabase-wrappers)
[![docs.rs badge](https://docs.rs/supabase-wrappers/badge.svg)](https://docs.rs/supabase-wrappers)
[![CI](https://github.com/supabase/wrappers/workflows/CI/badge.svg)](https://github.com/supabase/wrappers/actions?query=workflow%3ACI)
[![MIT/Apache-2 licensed](https://img.shields.io/crates/l/supabase-wrappers.svg)](./LICENSE)
[![Contributors](https://img.shields.io/github/contributors/supabase/wrappers)](https://github.com/supabase/wrappers/graphs/contributors)

`Wrappers` is a development framework for Postgres Foreign Data Wrappers ([FDW](https://wiki.postgresql.org/wiki/Foreign_data_wrappers)), written in Rust. Its goal is to make Postgres FDW development easier while keeping Rust language's modern capabilities, such as high performance, strong types, and safety.

`Wrappers` is also a collection of FDWs built by [Supabase](https://www.supabase.com). We currently support the following FDWs, with more are under development:

- [HelloWorld](./wrappers/src/fdw/helloworld_fdw): A demo FDW to show how to develop a baisc FDW.
- [BigQuery](./wrappers/src/fdw/bigquery_fdw): A FDW for Google [BigQuery](https://cloud.google.com/bigquery) which supports data read and modify.
- [Clickhouse](./wrappers/src/fdw/clickhouse_fdw): A FDW for [ClickHouse](https://clickhouse.com/) which supports data read and modify.
- [Stripe](./wrappers/src/fdw/stripe_fdw): A FDW for [Stripe](https://stripe.com/) API which supports data read only.
- [Firebase](./wrappers/src/fdw/firebase_fdw): A FDW for Google [Firebase](https://firebase.google.com/) which supports data read only.
- [Airtable](./wrappers/src/fdw/airtable_fdw): A FDW for [Airtable](https://airtable.com/) API which supports data read only.

## Features

- Minimum interface and easy to implement.
- Support for rich data types.
- Support both sync and async backends, such as RDBMS, RESTful APIs, flat files and etc.
- Built on top of [pgx](https://github.com/tcdi/pgx), providing higher level interfaces, without hiding lower-level C APIs.
- `WEHRE`, `ORDER BY`, `LIMIT` pushdown are supported.

## Installation

Wrappers is a pgx extension, so you can follow the [pgx installation steps](https://github.com/tcdi/pgx#system-requirements) to install Wrappers.

## Developing a FDW

To develop a FDW using Wrappers, you only need to implement the [ForeignDataWrapper](./supabase-wrappers/src/interface.rs) trait.

```rust
pub trait ForeignDataWrapper {
    // function for query planning
    fn get_rel_size(...) -> (i64, i32)

    // functions for data scan, e.g. select
    fn begin_scan(...);
    fn iter_scan(...) -> Option<Row>;
    fn re_scan(...);
    fn end_scan(...);

    // functions for data modify, e.g. insert, update and delete
    fn begin_modify(...);
    fn insert(...);
    fn update(...);
    fn delete(...);
    fn end_modify(...);
}
```

In a minimum FDW, which supports data scan only, `begin_scan()`, `iter_scan()` and `end_scan()` are required, all the other functions are optional.

To know more about FDW development, please visit the [Wrappers documentation](https://docs.rs/supabase-wrappers/latest/supabase_wrappers/).

## Basic usage

These steps outline how to use the a demo FDW [HelloWorldFdw](./wrappers/src/fdw/helloworld_fdw):

1. Clone this repo

```bash
git clone https://github.com/supabase/wrappers.git
```

2. Run it using pgx with feature:

```bash
cd wrappers/wrappers
cargo pgx run --features helloworld_fdw
```

3. Create the extension, foreign data wrapper and related objects:

```sql
-- create extension
drop extension if exists wrappers cascade;
create extension wrappers;

-- create foreign data wrapper and enable 'HelloWorldFdw'
drop foreign data wrapper if exists helloworld_wrapper cascade;
create foreign data wrapper helloworld_wrapper
  handler wrappers_handler
  validator wrappers_validator
  options (
    wrapper 'HelloWorldFdw'
  );

-- create server and specify custom options
drop server if exists my_helloworld_server cascade;
create server my_helloworld_server
  foreign data wrapper helloworld_wrapper
  options (
    foo 'bar'
  );

-- create an example foreign table
drop foreign table if exists balance;
create foreign table hello (
  id bigint,
  col text
)
  server my_helloworld_server
  options (
    foo 'bar'
  );
```

4. Run a query to check if it is working:

```sql
wrappers=# select * from hello;
 id |    col
----+-------------
  0 | Hello world
(1 row)
```

## Limitations

- Windows is not supported, that limitation inherits from [pgx](https://github.com/tcdi/pgx).
- Currently only supports PostgreSQL v14.

## Contribution

All contributions, feature requests, bug report or ideas are welcomed.

## License

[Apache License Version 2.0](./LICENSE)

