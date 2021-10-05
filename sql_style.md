* Everything is lower_snake_case.

  In the ancient past, UPPERCASE was used because syntax highlighting wasn't prevalent. In modern editors, it's unnecessary. Even queries embedded as strings inside other code can be syntax-highlighted, if the editor's syntax engine supports it.

  Using capitalization (including camelCase) for non-keywords can lead to gotchas because Postgres lowercases everything internally.

* Table and view names are plural: `table persons`, not `table person`.

* Always use `as` for aliases, for better readability and clarity.

* Time fields must end with `_at`. Example: `created_at`, `updated_at`. The only exceptions are "external" fields of objects from 3rd party services such as Stripe. See `orders` for examples.

* Boolean fields must start with `is_`. Example: `is_hidden`, `is_deleted`. Where possible, booleans should instead be timestamps, see below.

* References must include the entity type in the column name. For example, an article author could be either `person_id` or `author_person_id`, but not `author_id`. The extended article view would contain `person` or `author_person` rather than just `author`.
