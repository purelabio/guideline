(This file is a work in progress.)

* Avoid pointers when possible.
  * When designing an API, prefer to take and return non-pointers.
* Prefer `var name Type` over `name := new(Type)`.
  * Forcing pointers reduces the usability of an API.
* Embed by value. Never embed by pointer.
* Take single-method interfaces as inputs.
* Return single concrete types.
* Types are more powerful than funcs. Instead of writing a func, consider writing a type that implements that func as a method, ideally as a common interface.
  * For example: instead of a pretty-printing func `ƒ(any)->string`, write a single-valued `[1]interface{}` type with method `.String()`. This can be used for both eager _and_ lazy printing.

## Go-tchas

* Nil concrete type -> interface type -> interface is non-nil.
