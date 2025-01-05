---
tags:
  - comp-sci
  - language
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Referential_transparency
---
Referential transparency and referential opacity are properties of linguistic construction, and by extension of languages. A linguistic construction is called referentially transparent when for any expression built from it, replacing a subexpression with another one that denotes the same value does not change the value of the expression. Otherwise, it is called referentially opaque.

Referential transparency depends on the values associated to expressions, that is on the semantics of the language. So, both declarative languages and imperative languages can be referentially transparent or referentially opaque, according to the semantics they are given.

The importance of referential transparency is that is allows the programmer and the compiler to reason about the program behavior as a rewrite system. This can help is proving correctness, simplifying an algorithm, assisting in modifying code without breaking it, or optimizing code by means of memoization, common subexpression elimination, lazy evaluation, or parallelization.

## Formal Definitions

There are three fundamental properties concerning substitutivity in formal languages: referential transparency, definiteness and unfoldability.

### Referential Transparency

An operator is referentially transparent if it is referentially transparent in all places. Otherwise it is referentially opaque.

A formal language is referentially transparent if all its operators are referentially transparent. Otherwise it is referentially opaque.

### Definiteness

A formal language is definite if all occurrences of a variable within its scope denote the same value. Mathematics is definite: $3x^2 + 2x + 17$. The two occurrences of $x$ denote the same value.

### Unfoldability

A formal language is unfoldable if all expressions are $\beta$-reducible. The lambda calculus is unfoldable.

### Relations Between the Properties

Referential transparency, definiteness and unfoldability are independent. Definiteness implies unfoldability only for deterministic languages. Non-deterministic languages cannot have definiteness and unfoldability at the same time.