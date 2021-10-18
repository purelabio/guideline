(This file is a work in progress.)

* Avoid pointers when possible.
  * When designing an API, prefer to take and return non-pointers.
* Prefer `var name Type` over `name := new(Type)`.
  * Forcing pointers reduces the usability of an API.
* Embed by value. Never embed by pointer.
* Take single-method interfaces as inputs.
* Return single concrete types.

## Go-tchas

* Nil concrete type -> interface type -> interface is non-nil.
