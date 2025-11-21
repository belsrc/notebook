---
tags:
  - math
  - algebra
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Distributive_property
---
The distributive property of binary operations is a generalization of the distributive law:

$x \cdot (y + z) = x \cdot y + x \cdot z$

For example:

$2 \cdot (1 + 3) = (2 \cdot 1) + (2 \cdot 3)$

One would say that multiplication distributes over addition.

This basic property of numbers is part of the definition of most algebraic structures that have two operations called addition and multiplication. It is also encountered in Boolean algebra and mathematical logic, where each of the logical and (denoted $\wedge$) and the logical or (denoted $\vee$) distributes over the other.

## Definition

Given a set $S$ and two binary operators $*$ and $+$ on $S$:

- the operation $*$ is _left-distributive_ over (or with respect to) $+$ if, given any elements $x,y \text{ and } z$ of $S$.

$x * (y + z) = (x * y) + (x * z)$

- the operation $*$ is _right-distributive_ over $+$ if, given any elements $x,y \text{ and } z$ of $S$.

$(y + z) * x = (y * x) + (z * x)$

- the operation $*$ is _distributive_ over $+$ if it is left- and right-distributive.

When $*$ is commutative, the three conditions above are logically equivalent.

If the operation denoted $\cdot$ is not commutative, there is a distinction between left-distributivity and right-distributivity:

$a \cdot (b \pm c) = a \cdot b \pm a \cdot c \ \text{(left-distributive)}$

$(a \pm b) \cdot c = a \cdot b \pm b \cdot c \ \text{(right-distributive)}$

One example of an operation that is "only" right-distributive is division, which is not commutative:

$(a \pm b) \div c = a \div c \pm b \div c$

In this case, left-distributivity does not apply:

$a \div (b \pm c) \neq a \div b \pm a \div c$

## Truth Functional Connectives

Distributivity is a property of some logical connectives of truth-functional propositional logic. The following logical equivalences demonstrate that distributivity is a property of particular connectives. The following are truth-functional tautologies.

#### Distribution of conjunction over disjunction

$(P \wedge (Q \vee R)) \Leftrightarrow ((P \wedge Q) \vee (P \wedge R))$

#### Distribution of disjunction over conjunction

$(P \vee (Q \wedge R)) \Leftrightarrow ((P \vee Q) \wedge (P \vee R))$

#### Distribution of conjunction over conjunction

$(P \wedge (Q \wedge R)) \Leftrightarrow ((P \wedge Q) \wedge (P \wedge R))$

#### Distribution of disjunction over disjunction

$(P \vee (Q \vee R)) \Leftrightarrow ((P \vee Q) \vee (P \vee R))$

#### Distribution of implication

$(P \to (Q \to R)) \Leftrightarrow ((P \to Q) \to (P \to R))$

#### Distribution of implication over equivalence

$(P \to (Q \leftrightarrow R)) \Leftrightarrow ((P \to Q) \leftrightarrow (P \to R))$

#### Distribution of implication over conjunction

$(P \to (Q \wedge R)) \Leftrightarrow ((P \to Q) \wedge (P \to R))$

#### Distribution of disjunction over equivalence

$(P \vee (Q \leftrightarrow R)) \Leftrightarrow ((P \vee Q) \leftrightarrow (P \vee R))$
