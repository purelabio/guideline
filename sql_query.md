(This file is a work in progress.)

### TODO

* Postgres gotcha: `false or` in `where` can make queries slower.

* Never use unqualified names in queries with joins (unless `using`).

  * Hazardous name scoping rules can lead to accidental shadowing.

### Always use arguments; never interpolate into strings

Forbidden:

```golang
query := fmt.Sprintf(`
  select *
  from some_table
  where id = '%v'
`, id)

conn.QueryContext(ctx, query)
```

Allowed:

```golang
query := `
  select *
  from some_table
  where id = $1
`
args := []interface{}{id}

conn.QueryContext(ctx, query, args...)
```

Recommended:

```golang
import s "github.com/mitranim/sqlb"

query := s.DictQ(`
  select *
  from some_table
  where id = :id
`, s.Dict{
  `id`: id,
})

text, args := s.Reify(query)
conn.QueryContext(ctx, text, args...)
```

### Prefer explicit joins over implicit joins

Worse:

```sql
select
from
  table_a,
  table_b
where
  table_a.col = table_b.col
```

Better:

```sql
select
from
  table_a
  inner join table_b on table_a.col = table_b.col
```

Why:

* Can mix and match inner/outer/cross/lateral joins.
* Less error prone.

### Prefer views over subqueries

Worse:

```sql
select
from
  table_a
  left join (
    select some_agg(some_col)
    from table_b
    group by another_col
  ) on some_condition
```

Better:

```sql
create view some_view as (
  select some_agg(some_col)
  from table_b
  group by another_col
);

select
from
  table_a
  left join some_view on some_condition
```

Why:

* Views are automatically reusable.
* The resulting query is simpler.

### Prefer outer joins over inner joins

Inner joins can be slower than equivalent outer joins. When both sides are guaranteed to be non-null, prefer outer:

```sql
create table ones (
  id uuid primary key default gen_random_uuid()
);

create table twos (
  id     uuid primary key default gen_random_uuid()
  one_id uuid not null references ones on update cascade on delete cascade
);

-- Uses `left join`, not `inner join`. May be faster.
select
from
  twos
  left join ones on twos.one_id = ones.id;
```

### Avoid comparing rows to `null`

Do not:

```sql
select
from
  table_a
  left join table_b on some_condition
where
  table_b is not null
```

This doesn't do what you expect. Instead, use an inner join or compare the primary key.

### In queries with multiple tables, qualify all names

Do not:

```sql
select
  some_col
from
  table_a
  inner join table_b on some_condition
where
  another_col is not null
```

Do:

```sql
select
  table_a.some_col
from
  table_a
  inner join table_b on some_condition
where
  table_b.another_col is not null
```

### Avoid unnecessary casts to and from `text`.

To-and-from-text conversions in the wrong places can be expensive.

Worse:

```sql
:some_arg::text[]::some_enum[] is null

some_col = any(:some_arg::text[]::some_enum[])
```

Better:

```sql
:some_arg::actual_type[] is null

some_col = any(:some_arg::actual_type[])
```

### Be careful with `distinct from`

In Postgres, `is [not] distinct from` is not indexable, and performs much slower than an equivalent combination of `=` and `is [not] null`. It should be avoided in queries.

However, `is not distinct from` is the _only_ correct way to detect `null` in composite type constraints:

```sql
create type coin_struct as (currency currency, cents nat_pos);

create domain coin as coin_struct
  constraint "coin.check"
  check (
    value is not distinct from null
    or (
      true
      and (value).currency is not null
      and (value).cents    is not null
    )
  );
```

### Postgres gotcha: prefer `=` over `in` when possible

Postgres (version 14 at the time of writing) has a performance gotcha. Sometimes when `=` and `in` should be equivalent, `in` can be much slower.

Example with `in`:

```sql
explain analyze
with
  pid as (select id from persons where email = $1),
  aid as (select id from fin_accounts where person_id = (table pid))
select *
from fin_account_credit_balances
where id in (table aid)
```

```
Planning Time: 3.821 ms
Execution Time: 63.747 ms
```

Exactly the same but with `=`:

```
explain analyze
with
  pid as (select id from persons where email = 'me@mitranim.com'),
  aid as (select id from fin_accounts where person_id = (table pid))
select *
from fin_account_credit_balances
where id = (table aid)
```

```
Planning Time: 2.890 ms
Execution Time: 9.602 ms
```

### Avoid `limit 1`

`limit 1` is an antipattern. Single entries must _always_ be identified by a unique key, such as the primary key.
