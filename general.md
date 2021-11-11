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
* API design: implement the general case; optimize for the common case.
