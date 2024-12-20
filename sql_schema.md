## General

* Be consistent. Use the same principles for all tables.

* Every table has a primary key.

  * For "primary" tables: column `id`, type `uuid`, randomly generated.

  * For "junction" tables: composite key: ids of connected entities.

  * For "secondary" tables that correspond 1-to-1 to primary tables: reuse the id of the connected primary entity.

    * Example: primary table `persons` with PK `id`, secondary table `person_prefs` with PK `person_id`.

  * For tracking insertion order, use either `ord bigserial not null` with a unique constraint, or precise timestamps with a unique constraint. Note that not all SQL databases support precise timestamps.

  * For hardcoded pseudo-enum tables such as currencies, it's okay to use non-uuid as primary key. For example, currencies would use `char(3)`.

* Columns have explicit `not null` or `null`.

* For non-unique, non-reference columns, prefer `not null` over `null`.

  * Where possible, prefer `not null default X` where X is the zero value.

  * Exception: dates and date-times should be nullable because they don't have a useful zero value.

* For unique columns, disallow the zero value such as `0` or `''`. If the value may be missing, make the column nullable. Unique indexes automatically ignore nulls, making it easy to define unconditional unique constraints compatible with upsert queries.

* Approach to polymorphic relations is described below: [Polymorphic Relations](#polymorphic-relations).

* All tables must have columns `created_at`, `updated_at`, and trigger `touch_updated_at`.

* All constraints, including primary keys, foreign keys, unique indexes, and simple checks, must be named manually. This prevents accidental double creation of the same constraint, makes it possible to reliably rename and drop constraints in migrations without "guessing" their names, and exposes naming inconsistencies caused by table renaming.

* Tables with user-created data, such as person profiles or articles, include `deleted_at` and shouldn't be actually deleted. Internal tables such as junctions/edges don't include `deleted_at` and should be actually deleted. Versioned tables might use `is_deleted` instead of `deleted_at`.

* Time must be stored as a `timestamptz` rather than `timestamp` in order to avoid a massive Postgres gotcha: when parsing datetime from a string, `timestamp` TRUNCATES the input and completely ignores the timezone offset, parsing it AS IF the datetime part was specified in UTC time rather than local time. `timestamptz` avoids this problem. Note that `timestamptz` doesn't actually store a timezone, it only parses it, then stores UTC time.

* Where possible, use timestamps instead of booleans. For example, instead of `is_deleted`, store `deleted_at`.

* Prefer uint over `bigint`. SQL doesn't provide uints, so you have to define them. Suggestion: `nat` for "natural", `nat_pos` for "natural positive".

* When storing objects from 3rd party services such as Stripe, fields copied from the source must keep the original name prefixed with `external_`, while any fields added or modified by our code must be unprefixed and follow our naming conventions.

* Time offsets and intervals are either:

  * Stored as PG `interval` for better compatibility with civil time calculations.

  * Measured in seconds and stored as `nat` or `nat_pos`, to keep them easily portable between languages and data formats.

* Constraint names should follow a convention similar to this: `<table_name>.<description>` or `<table_name>.<category>.<description>`. This improves error messages when constraint violations are exposed to clients.

* Names of all non-unique indexes should be UUIDs without dashes. This makes them much easier to define, and also avoids naming collisions.

  * When possible, comment on an index what is it _for_. Provide the name of the relevant SQL view, query-generating function, etc.

* Create matching indexes for queries that are known to be slow. Keep in mind that syncing indexes with queries increases the code maintenance burden.

* Foreign keys referencing a primary key _do not_ specify the column name. Foreign keys referencing a non-primary key _do_ specify the column name. Examples:

        not null references timezones        on update cascade on delete cascade
        not null references timezones (name) on update cascade on delete cascade

* Non-nullable foreign keys must use the following:

        on update cascade on delete cascade

* Nullable foreign keys must use one of:

        on update cascade on delete cascade
        on update cascade on delete set null

* Avoid `create if not exists`, `create or replace`, etc. The schema is applied to an empty database. Migrations must precisely know the schema they're being applied to. Neither the schema nor the migrations should need the "optional" clauses.

* Ensure that invalid data cannot be represented. Use check constraints, exclude constraints, unique indexes, foreign keys, enums, domain constraints, etc., to prevent invalid states. If no other option is available, use triggers.

* When writing a table name with two words, such as `feedback_topics`, convert the less-general word into a "type" column, such as `topics` with `type = 'feedback'`.

## Polymorphic Relations

To represent polymorphic relations, we use a set of foreign key fields
referencing different tables, alongside a polymorphic check constraint.

```sql
create table reactions (
  id                 uuid        not null default gen_random_uuid(),
  person_id          uuid        not null,
  subject_article_id uuid            null,
  subject_service_id uuid            null,
  subject_person_id  uuid            null,
  is_like            bool        not null,
  is_fav             bool        not null,
  created_at         timestamptz not null default now(),
  updated_at         timestamptz not null default now(),

  constraint "reactions.key"                    primary key (id),
  constraint "reactions.key.person_id"          foreign key (person_id) references persons on update cascade on delete cascade,
  constraint "reactions.key.subject_article_id" foreign key (articles)  references persons on update cascade on delete cascade,
  constraint "reactions.key.subject_service_id" foreign key (services)  references persons on update cascade on delete cascade,
  constraint "reactions.key.subject_person_id"  foreign key (persons)   references persons on update cascade on delete cascade,

  constraint "reactions.mutex.subject"
  check ((
    (subject_article_id is not null)::int +
    (subject_service_id is not null)::int +
    (subject_person_id  is not null)::int +
    0
  ) = 1),

  -- To make sure the user can place only one like per subject.
  constraint "reactions.unique.article" unique (person_id, subject_article_id),
  constraint "reactions.unique.service" unique (person_id, subject_service_id),
  constraint "reactions.unique.person"  unique (person_id, subject_person_id)
);

```

This approach has several advantages over others:

  * This preserves the correct relational data structure; we have foreign key constaints ensuring the integrity of the data; there's no need for a field that would specify the subject's table. Even if all IDs in the system were UUIDs, and there were never any collisions, it's not correct even from a conceptual point of view to mix IDs from different tables in one field.

  * We retain specialized tables for different entity types, with their own structure, constraints, and indexes.

## Enums

Sometimes you want to use native enums. Sometimes you want to use pseudo-enum tables. See below.

Example with enum:

```sql
create type gender as enum ('female', 'male', 'other');

create table persons (
  id     uuid   not null default gen_random_uuid(),
  gender gender     null,

  constraint "persons.key" primary key (id)
);
```

Example with pseudo-enum table:

```sql
create domain gender as text;

create table genders (id gender primary key);

insert into genders values ('female'), ('male'), ('other');

create table persons (
  id     uuid   not null default gen_random_uuid(),
  gender gender     null,

  constraint "persons.key" primary key (id),

  constraint "persons.key.gender"
    foreign key (gender)
    references genders on update cascade on delete set null
);
```

Enums are supposed to be more compact. Internally they're stored as numbers. The text representation is used only when interfacing with clients. This may give them an advantage, but you should measure and compare the difference in your particular DB.

An enum type carries its constraints with itself, while pseudo-enums require a foreign key constraint in each table that references them. Enums require less code, and you can't forget to add a foreign key.

However, at least in Postgres, enums are difficult to extend. You can add a value, but can't use it in the same transaction. You can't remove values. Meanwhile, pseudo-enum tables have no such restrictions. Therefore, a pseudo-enum is a safer choice for future extensibility.

Note that even gender may be extensible!

```sql
insert into genders values ('none'), ('prefer_not_to_say');
```
