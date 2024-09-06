---
tags:
  - math
  - algebra
gardening: 🌱
reference:
  - https://en.wikipedia.org/wiki/Identity_element
---
An identity element or neutral element of a binary operation is an element that leaves unchanged every element when the operation is applied. For example, $0$ is an identity element of the addition of real numbers. This concept is used in algebraic structures such as a groups and rings. The identity implicitly depends on the binary operation it is associated with.

## Definition

Let $(S,*)$ be a set $S$ equipped with a binary operation $*$. Then an element $e$ of $S$ is called left identity if $e * s = s \text{ }\forall\text{ } s \in S$, and a right identity if $s * e = s \text{ }\forall\text{ } s \in S$. If $e$ is both a left identity and a right identity, then it is called a two-sided identity, or simply an identity.

An identity with respect to addition is called an additive identity (often denoted as $0$) and an identity with respect to multiplication is called multiplicative identity (often denoted as $1$). In the case of a group for example, the identity element is sometimes simple denoted by the symbol $e$.

## Properties

For two elements $\{e,f\}$ with $*$ defined by $e *e = f * e = e$ and $f *f = e * f = f$ then $S = \{e,f\}$, with the equalities given, $S$ is a semigroup. It demonstrates the possibility for $(S,*)$ to have several left identities. In fact, every element can be a left identity. In a similar manner, there can be several left identities. But there is both a right identity and a left identity, then they must be equal, resulting in a single two-sided identity.

Is is also quite possible for $(S,*)$ to have no identity element, such as the case of even integers under the multiplication operation.

## Examples

| Set                                | Operation                                 | Identity                 |
| ---------------------------------- | ----------------------------------------- | ------------------------ |
| Real Numbers                       | $+$                                       | $0$                      |
|                                    | $\cdot$                                   | $1$                      |
| Complex Numbers                    | $+$                                       | $0$                      |
|                                    | $\cdot$                                   | $1$                      |
| Positive Integers                  | Least Common Multiple                     | $1$                      |
| Non-negative Integers              | Greatest Common Divisor                   | $0$                      |
| Vectors                            | Vector Addition                           | Zero Vector              |
| $m \times n$ matrices              | Matrix Addition                           | Zero Matric              |
| $m \times n$ sq matrices           | Matrix Multiplication                     | $I_n$                    |
| $m \times n$  matrices             | $\bigcirc$ (Hadamard Product)             | $J_{m,n}$                |
| All $f()$ from a set $M$ to itself | $\circ$ (function composition)            | Identity Function        |
| Subsets of a set $M$               | $\cap$ (Intersection)                     | $M$                      |
|                                    | $\cup$ (Union)                            | $\emptyset$ (Empty Set)  |
| Strings, lists                     | Concatenation                             | Empty String, Empty List |
| A Boolean Algebra                  | $\wedge$ (Logical And)                    | $\top$ (Truth)           |
|                                    | $\leftrightarrow$ (Logical Biconditional) | $\top$ (Truth)           |
|                                    | $\vee$ (Logical Or)                       | $\bot$ (Falsity)         |
|                                    | $\oplus$ (Logical XOR)                    | $\bot$ (Falsity)         |
