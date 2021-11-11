## General

* Be consistent. Use the same principles for all tables.

* Every table has a primary key: `id` with type `uuid`.

  * If need to preserve insertion order, use `ord bigserial not null` with a unique constraint.

* Columns have explicit `not null` or `null`.

* Prefer `not null` over `null`. Where possible, prefer `not null default X` where X is the zero value.

* Approach to polymorphic relations is described below: [Polymorphic Relations](#polymorphic-relations).

* All tables must have columns `created_at`, `updated_at`, and trigger `touch_updated_at`.

* Tables with user-created data, such as person profiles or articles, include `deleted_at` and shouldn't be actually deleted. Internal tables such as junctions/edges don't include `deleted_at` and should be actually deleted. Versioned tables might use `is_deleted` instead of `deleted_at`.

* Time must be stored as a `timestamptz` rather than `timestamp` in order to avoid a massive Postgres gotcha: when parsing datetime from a string, `timestamp` TRUNCATES the input and completely ignores the timezone offset, parsing it AS IF the datetime part was specified in UTC time rather than local time. `timestamptz` avoids this problem. Note that `timestamptz` doesn't actually store a timezone, it only parses it, then stores UTC time.

* Where possible, use timestamps instead of booleans. For example, instead of `is_deleted`, store `deleted_at`.

* Prefer uint over `bigint`. SQL doesn't provide uints, so you have to define them. Suggestion: `nat` for "natural", `nat_pos` for "natural positive".

* When storing objects from 3rd party services such as Stripe, fields copied from the source must keep the original name prefixed with `external_`, while any fields added or modified by our code must be unprefixed and follow our naming conventions.

* Time offsets and intervals are either:

  * Stored as PG `interval` for better compatibility with civil time calculations.

  * Measured in seconds and stored as `nat` or `nat_pos`, to keep them easily portable between languages and data formats.

* All constraints, including unique indexes, must be named like this:
  `<table_name>_<description>`. This improves error messages when constraint violations are exposed to clients. Named constraints and indexes can be reliably dropped in migrations.

* Names of all non-unique indexes must be UUIDs without dashes. This makes them much easier to define, and also avoids naming collisions.

* Create matching indexes for queries that are known to be slow. Keep in mind that syncing indexes with queries increases the code maintenance burden.

* Foreign keys referencing a primary key _do not_ specify the column name. Foreign keys referencing a non-primary key _do_ specify the column name. Examples:

        not null references timezones        on update cascade on delete cascade
        not null references timezones (name) on update cascade on delete cascade

* Non-nullable foreign keys must use the following:

        on update cascade on delete cascade

* Nullable foreign keys must use one of:

        on update cascade on delete cascade
        on update cascade on delete set null

* Junction/edge tables have their own `id` and a separate unique index for the references they contain, rather than a composite primary key.

* Avoid `create if not exists`, `create or replace`, etc. The schema is applied to an empty database. Migrations must precisely know the schema they're being applied to. Neither the schema nor the migrations should need the "optional" clauses.

* Ensure that invalid data cannot be represented. Use check constraints, exclude constraints, unique indexes, foreign keys, enums, domain constraints, etc., to prevent invalid states. If no other option is available, use triggers.

* When writing a table name with two words, such as `feedback_topics`, convert the less-general word into a "type" column, such as `topics` with `type = 'feedback'`.

## Polymorphic Relations

To represent polymorphic relations, we use a set of foreign key fields
referencing different tables, alongside a polymorphic check constraint.

    create table reactions (
      id                     uuid          primary key default gen_random_uuid(),
      person_id              uuid          not null references persons on update cascade on delete cascade,

      subject_article_id     uuid          null references articles on update cascade on delete cascade,
      subject_service_id     uuid          null references services on update cascade on delete cascade,
      subject_person_id      uuid          null references persons  on update cascade on delete cascade,

      is_like                bool          not null,
      is_fav                 bool          not null,

      created_at             timestamptz   not null default current_timestamp,
      updated_at             timestamptz   not null default current_timestamp

      constraint "db.constraint.reactions_subject_mutex"
      check ((
        (subject_article_id is not null)::int +
        (subject_service_id is not null)::int +
        (subject_person_id  is not null)::int +
        0
      ) = 1)
    );

    -- To make sure the user can place only one like per subject.
    create unique index "db.constraint.reaction_unique_article" on reactions (person_id, subject_article_id);
    create unique index "db.constraint.reaction_unique_service" on reactions (person_id, subject_service_id);
    create unique index "db.constraint.reaction_unique_person"  on reactions (person_id, subject_person_id);

This approach has several advantages over others:

  * This preserves the correct relational data structure; we have foreign key constaints ensuring the integrity of the data; there's no need for a field that would specify the subject's table. Even if all IDs in the system were UUIDs, and there were never any collisions, it's not correct even from a conceptual point of view to mix IDs from different tables in one field.

  * We retain specialized tables for different entity types, with their own structure, constraints, and indexes.
