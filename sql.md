## Critical

* Prevent invalid states.
  * Reduce the amount of states.
* Prevent duplication.

Any data state that wasn't _explicitly intended_ should be considered invalid, and must be forbidden through constraints.

Preventing invalid states:

  * Use constraints.
  * Use foreign key constraints.
  * Use exclusion constraints.
  * Use unique indexes.
  * Use domain types with constraints.
  * Use triggers for validation.

Preventing duplication:

  * Normalize your data.
  * Avoid caching (unless it auto-updates synchronously).

## General

* Source of truth = schema file.
  * Always up to date.
  * Reflects the _current_ state of the schema.
  * Can be used to create an empty DB, with no migrations.
  * When writing migrations, also modify the schema file.
* Each schema = 1 file (not a folder).
  * SQL definitions are order-sensitive.
  * Much easier than automatic reordering of definitions.
* Two schemas:
  * Main schema for persistent data (usually `public`).
    * Extensions.
    * Tables.
    * Triggers.
    * Types used by tables.
    * Functions used by tables and triggers.
    * Views used by triggers.
    * Modified via migrations.
  * Ephemeral schema for views and functions (`eph`).
    * Views.
    * Functions.
    * Dropped before each migration, re-created after running migrations.
* Restrict maximum size of resizable columns such as text, byte array, JSON.
* Use indexes.

## Migrations

* Use a single transaction for the entire migration process.
* Avoid `cascade`.
  * Explicitly drop the dependencies.
* Prefer `drop` + `create` over `create or replace`.
  * Fewer surprises.
* Avoid `if not exists`.
  * Migrations must exactly know the schema they're being applied to.
  * Migrations must be applied only once.
  * Migrations are not idempotent.
  * Migrations are not reversible.
  * Sometimes OK for schemas and extensions.
* Migrations must have unique _hardcoded_ keys to avoid collisions with _future_ migrations after removing old migrations from the codebase.
  * Can use hardcoded UUIDs.
  * Can use hardcoded dates.
* Migrations must have a hardcoded order, because they often depend on each other.
* Write migrations in SQL, but run them from your main language, in a single transaction.
  * Allows hybrid migrations, e.g. partially written in Go.
  * Allows better code reuse.
  * Allows accessing 3rd party services.
* Drop the "ephemeral" schema before migrating, and re-create it after migrating.

## Additional

* See [schema guidelines](./sql_schema.md).
* See [query guidelines](./sql_query.md).
