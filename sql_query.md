(This file is a work in progress.)

### TODO

* Postgres gotcha: `false or` in `where` can make queries slower.
* Never use unqualified names in queries with joins (unless `using`).
* Avoid unnecessary casts to and from `text`.

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
query := sqlb.NamedQuery(`
  select *
  from some_table
  where id = :id
`, map[string]interface{}{
  `id`: id,
})

conn.QueryContext(ctx, query.String(), query.Args...)
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
* Fewer identifiers in scope = less likely to accidentally use the wrong column or table.

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
