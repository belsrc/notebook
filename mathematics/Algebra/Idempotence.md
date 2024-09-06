---
tags:
  - math
  - algebra
  - comp-sci
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Idempotence
---
Idempotence is the property of certain operations whereby they can be applied multiple times without changing the result beyond the initial application. The term was introduced  in the context of elements of algebras that remain invariant when raised to a positive integer power and literally means "(the quality of having) the same power."

## Definition

An element $x$ of a set $S$ equipped with a binary operator $*$ is said to be idempotent under $*$ if:

$$
x \cdot x = x
$$

The binary operation $*$ is said to be idempotent if:

$$
x \cdot x = x \text{ }\forall\text{ } x \in S
$$

## Idempotent Functions

In the monoid $(E^E,\circ)$ of the functions from a set $E$ to itself with function composition $\circ$, idempotent elements are the functions $f : E \to E$ such that $f \circ f = f$, that is such that $f(f(x)) = f(x)$ for all $x \in E$. For example:

The absolute value is idempotent. $\text{abs} \circ \text{abs} = \text{abs}$, that is $\text{abs}(\text{abs}(x)) = \text{abs}(x)$ for all $x$.

Constant functions are idempotent.

The identity functions are idempotent. 

The floor $\lfloor\text{ }\rfloor$ , ceiling $\lceil\text{ }\rceil$, and fractional part functions are idempotent.

Neither the property of being idempotent nor that of being not is preserved under function composition. As an example for the former, $f(x) = x \bmod 3$ and $g(x) = \max (x,5)$ are both idempotent, but $f \circ g$ is not, although $g \circ f$ happens to be. As an example for the latter, the negation function $\neg$ on the Boolean domain is not idempotent, but $\neg\circ\neg$ is. Similarly, unary negation $-(\cdot)$ of real numbers is not idempotent, but $-(\cdot) \circ -(\cdot)$ is.

## Examples

In the monoid $(\mathbb{N},\times)$ of the natural numbers with multiplication, only $0$ and $1$ are idempotent. $0 \times 0 = 0$ and $1 \times 1 = 1$

In the monoid $(\mathbb{N},+)$ of the natural numbers with addition, only $0$ is idempotent. $0 + 0 = 0$

In a magma $(M,\cdot)$, an identity element $e$ or an absorbing element $a$, if it exists, is idempotent. $e \cdot e = e$ and $a \cdot a = a$

In a group $(G,\cdot)$, the identity element $e$ is the only idempotent element.

## Computer Science

In computer science, the term idempotence may have a different meaning depending on the context in which it is applied:

- In imperative programming, a subroutine with side effects is idempotent if multiple calls to the subroutine have the same effect on the system state as a single call.

- In functional programming, a pure function is idempotent if it is idempotent in the mathematical sense.

## Computer Science Examples

A request for changing a customer's address to XYZ is typically idempotent, because the final address will be the same no matter how many times the request is submitted. However, a customer request for placing an order is typically not idempotent since multiple requests will lead to multiple orders being placed.