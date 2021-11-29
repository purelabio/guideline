(This file is a work in progress.)

## Misc

* Avoid pointers when possible.
  * When designing an API, prefer to take and return non-pointers.
  * Forcing pointers reduces API usability.
* Prefer `var name Type` over `name := new(Type)`.
* Embed by value. Avoid embedding by pointer.
* Take single-method interfaces as inputs.
* Return single concrete types.
* For non-stack-allocated types, prefer inout parameters (by pointer) over return parameters. The caller should be able to preallocate ("bring your own buffer").
* Types are more powerful than funcs. Instead of writing a func, consider writing a type that implements that func as a method, ideally as a common interface.
  * For example: instead of a pretty-printing func `ƒ(any)->string`, write a single-valued `[1]any` type with method `.String()`. This can be used for both eager _and_ lazy printing.
* Create specialized types for similar, but semantically different units.
  * Different DB models -> specialized ID types.
  * Cents != other integers.
  * Meters != feet.
  * Kilograms != pounds.
  * Etc.

## Go-tchas

* Nil concrete type -> interface type -> interface is non-nil.
