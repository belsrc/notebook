---
tags:
  - notes
  - functional
  - talk
---
## Functional Design: Applying past software architecture and design by Janet Carr
https://www.youtube.com/watch?v=hYSxlN_d2-g

- Recursion over looping (via recursion for low loops or trampolines for large loops)

- Functions over objects
	- HOFS
	- Function composition over Object composition
	- Purity over side-effects

- Transformations over instructions
	- Imperative is a series of instructions
	- Declarative is a series of transformations on data

### Classic Design
(OOP => FP)
- High Cohesion => WomM (works on my machine)

- Low Coupling => Programming to an interface

- Delegation => Callback/HOFs

- State Encapsulation => Persistent data structures

- Object Composition => Function Composition

- OO Interface (method signatures) => FP Interface (function signatures)
	
- Obj Delegation (give work to another obj) => Func Delegation (give work to another func)

### Don't disregard design patterns as "simply OOP"

#### State Pattern

Problem: Modeling Finite State Machines
- OOP:
	- Uses delegate obj as current state
	- Delegate transitions to states
	- Only current state knows when to transition
- FP:
	- Uses a function delegate and recursion
	- State is a function (input => state)

#### Observer Pattern

Problem: Decouple state changes from state usage

- OOP:
	- Notify a collection of observer objs
- FP:
	- Notify a collection of observer funcs

#### Strategy Pattern

Problem: Decouple algorithm from its context

- OOP:
	- Delegate algorithm to obj
- FP:
	- Delegate algorithm to funcs