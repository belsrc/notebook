---
tags:
  - math
  - algebra
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Associative_property
---
The associative property is a property of some binary operations that means that rearranging the parentheses in an expression will not change the result. Within an expression containing two or more occurrences in a row of the same associative operator, the order in which the operations are performed does not matter as long as the sequence of the operands is not changed. That is, rearranging the parentheses in such an expression will not change its value.

$(2 + 3) + 4 = 2 + (3 + 4) = 9$

$2 \times (3 \times 4) = (2 \times 3) \times 4 = 24$

Even though the parentheses were rearranged on each line, the values of the expressions were not altered. Since this holds true when performing addition and multiplication on an real number, it can be said that "addition and multiplication of real numbers are associative operations."

Associativity is not the same as [commutativity](../../mathematics/algebra/Commutative%20Property), which addresses whether the order of two operands affects the result.

Many operations are non-associative; some examples include subtraction, exponentiation and the vector cross product. The addition of floating point numbers in computer science is not associative and the choice of how to associate an expression can have significant effect on rounding errors.

## Definition

Formally, a binary operation $*$ on a set $S$ is called associative if it satisfies the associative law:

$(x * y) * z = x * (y * z)\text{ }\forall\text{ } x, y, z \in S$

The associative law can also be expressed in functional notation:

$(f \circ (g \circ h))(x) = ((f \circ g) \circ h)(x)$

If a binary operation is associative, repeated application of the operation produces the same result regardless of how valid pairs of parentheses are inserted in the expression. This is called the generalized associative law.

- $((ab)c)d$
- $(a(bc))d$
- $a((bc)d)$
- $(a(b(cd)))$
- $(ab)(cd)$

If the product operation is associative, the generalized associative law says that all these expressions will yield the same result. So unless the expression with omitted parentheses already has a different meaning, the parentheses can be considered unnecessary and the product can be written unambiguously as:

$abcd$

As the number of elements increases, the number of possible ways to insert the parentheses grows quickly, but they remain unnecessary for disambiguation.

An example where this does not work is the logical biconditional $\leftrightarrow$. It is associative; thus:

$A \leftrightarrow (B \leftrightarrow C) = (A \leftrightarrow B) \leftrightarrow C$

But, $A \leftrightarrow B \leftrightarrow C$ most commonly means $(A \leftrightarrow B)$ and $(B \leftrightarrow C)$ which is not equivalent.

## Examples

- String concatenation is associative (though, not commutative).

```js
("hello" + " ") + "world" === "hello" + (" " + "world")
```

- In arithmetic, addition and multiplication of real numbers are associative.

$$
  \begin{align*}
  \begin{rcases}
    (x + y) + z = x + (y + z) = x + y + x\\
    (xy)z = x(yz) = xyz
  \end{rcases}\text{ }\forall\text{ } x, y, z \in \mathbb{R}
  \end{align*}
$$

$$
  \begin{rcases}
    (x + y) + z = x + (y + z) = x + y + x\\
    (xy)z = x(yz) = xyz
  \end{rcases}\text{ }\forall\text{ } x, y, z \in \mathbb{R}
$$

- Taking the intersection or the union of sets:

$$
  \begin{rcases}
    (A \cap B) \cap C = A \cap (B \cap C) = A \cap B \cap C\\
    (A \cup B) \cup C = A \cup (B \cup C) = A \cup B \cup C \text{ }
  \end{rcases} \text{ }\forall\text{ } A, B, C
$$

- If $M$ is some set and $S$ denotes the set of all functions $M \to M$ , then the operation of function composition on $S$ is associative:

$$
f \circ (g \circ h) = (f \circ g) \circ h = f \circ g \circ h \text{ }\forall\text{ } f, g, h \in S
$$

- In category theory, composition of morphisms is associative by definition. Associativity of functors and natural transformations follows from associativity of morphisms.

- For real numbers (and for any totally ordered set), the minimum and maximum operations is associative:

$$
\text{max}(a, \text{max}(b,c)) = \text{max}(\text{max}(a,b),c)
$$

and

$$
\text{min}(a, \text{min}(b,c)) = \text{min}(\text{min}(a,b),c)
$$

## Truth Functional Connectives

Associativity is a property of some logical connectives of truth-functional propositional logic. The following logical equivalences demonstrate that associativity is a property of particular connectives. The following (and their converses, since $\leftrightarrow$ is commutative) are truth-functional tautologies.

#### Associativity of conjunction

$$
(P \wedge (Q \wedge R)) \leftrightarrow ((P \wedge Q) \wedge R)
$$

#### Associativity of disjunction

$$
(P \vee (Q \vee R)) \leftrightarrow ((P \vee Q) \vee R)
$$

#### Associativity of equivalence

$$
(P \leftrightarrow (Q \leftrightarrow R)) \leftrightarrow ((P \leftrightarrow Q) \leftrightarrow R)
$$

## Non-associative Operations

A binary operation $*$ on a set $S$ that does not satisfy the associative law is called non-associative.

$$
(x \cdot y) \cdot z \neq x \cdot (y \cdot z) \text{ }\exists\text{ } x,y,z \in S
$$

For such an operation the order of evaluation _does_ matter.

#### Subtraction

$$
(5 - 3) - 2 \neq 5 - (3 - 2)
$$

#### Division

$$
(4/2)/2 \neq 4/(2/2)
$$

#### Exponentiation

$$
2^{(1^2)} \neq (2^1)^2
$$

#### Vector Cross Product

$$
i \times (i \times j) = i \times k = -j
$$

$$
(i \times i) \times j = 0 \times j = 0
$$
