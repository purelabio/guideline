## Values / principles

* Simplicity.
* Brevity.
* Flexibility.
* Less is more.
* Smaller bricks.
* Always write tests.

## Misc

* Minimize code.
* Deduplicate code.
  * Deduplicate by breaking things into smaller bricks.
  * Avoid making new abstractions.
  * Making new names is unavoidable.
* Deduplicate costs.
  * Everything that can be done only once, should be done only once.
  * Costs that can be paid at compile time, should be paid at compile time.
* Avoid avoidable closures. Define functions in root scope.
  * Move unavoidable closures as close to root scope as possible.
* Reduce the amount of possible states.
  * Simpler software has as few states as possible.
  * More states = more likely to have undefined behaviors = less reliability.
* Avoid invalid states.
  * In correct software, all states must be intentional.
  * Correct software must handle all possible inputs.
* Minimize dependencies.
  * Fewer external libraries.
  * Fewer function parameters.
* Optimizations.
  * Avoid premature optimizations.
  * Don't optimize without profiling and metrics.
  * Simplicity and clarity > performance.
  * Always optimize the bottleneck.
* Avoid unnecessary over-modularization.
* Libraries should not depend.
  * More specifically: 3rd party libraries should depend only on the standard library, but avoid depending on each other.
  * Libraries should connect with each other only through common interfaces found in the language or in the standard library.
  * Only apps are allowed to depend on 3rd party libraries.
* Avoid code that does nothing.
* Prefer immutability by default, create-once, assign-once, use-once.
* Make zero values useful.
  * Whenever possible, treat nil as the zero value of any given type.
* Copy-paste is unavoidable, but the only acceptable unit of copy-paste is a single expression, preferably a single function call.
* Prefer sync code over async.
* One action, one concept.
  * One thing should perform one action.
  * Its name should contain only one term.
  * When a name contains multiple terms, it's a signal that you should break it up.
* Validate inputs immediately.
* Fail early and loudly.
  * Compile-time errors are better than run-time errors.
  * Run-time errors must be as close to the trouble as possible.
  * Use runtime assertions and invariant checks.
* Prefer early returns over nested conditionals.
* Idempotent operations must return something (like a boolean) that tells you which path was taken: insert or update, delete or skip, etc.
* Avoid `if` whenever possible. Alternative solutions:
  * Choose a different function at a callsite.
  * Use an interface.
* Commit message format: due to UI conventions in GitHub, GitLab, and various Git clients, it's better to limit the first line to 50-60 characters, followed by an empty line, followed by a listing of various changes, aligned to 70-80 columns.
* Default values should always be zero. Never use non-zero default values.

## Naming

### Parameters and variables

Names of function parameters and local variables should communicate _role_, not _type_. It's advantageous to use generic role names. For example, when there's only one parameter which serves as a data source to be transformed, the generic name is `src`. In UI views that display one entity, the generic name can be `ent`. And so on.

Type information can be communicated with type annotations (in statically typed languages), or with type assertions (in dynamically typed languages), which makes it redundant to put it into variable names. Role-based names communicate additional information, and short generic names are shorter to type and to read. They are also more compatible with copy-pasting.

Typical generic names (non-exhaustive):

* `err` ("error")
* `val` ("value")
* `src` ("source")
* `tar` ("target")
* `inp` ("input")
* `out` ("output")
* `ind` ("index")
* `ent` ("entity")
* `buf` ("buffer")
* `key`

### Singular vs plural

When type is singular, name must be singular (`blah`).

When type is plural, name must be either plural (`blahs`), or singular and ending with the name of the collection type (`blah_set`, `blah_list`).

In fields used for network APIs, where the only supported collection type is a list (URL query and JSON have this characteristic), collection names must be plural (`blahs`), without naming the collection type (~~`blah_list`~~).

## API design

* Implement the general case; optimize for the common case.
* Types are more powerful than functions. Consider implementing any given feature as a type rather than a function.
* Instance methods are more powerful than static methods. Consider implementing any given feature as an instance method. Provide static methods or top-level functions as shortcuts.
* Defer choices to the user. Make everything overridable.
* Idempotent functions must return an indicator whether the operation was performed.
* Comments should be "why", not so much "what". Avoid comments which are redundant with the code.
