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
* Avoid avoidable closures. Define functions in root scope.
  * Move unavoidable closures as close to root scope as possible.
* Reduce the amount of possible states.
  * Simpler software has as fewer states as possible.
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
* Copy-paste is unavoidable, but the only acceptable unit of copy-paste is a single expression, preferably a single function call.
* Prefer synchronize code over asynchronize.
* Only per one name (term) for each thing.
