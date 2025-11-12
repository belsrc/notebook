---
tags:
  - math
  - category-theory
date: 2025-11-11
gardening: 🌳
reference:
  - https://www.youtube.com/watch?v=HJ9-yvZ-Toc
---
## Reasoning Without “Looking Inside” Objects

* A group of people (Alice, Bob, Charlie, Eve) with friendship arrows.
* Even without knowing “what’s inside” Alice, we can tell she’s the most popular by **how others relate to her** — everyone has an arrow pointing to her.

This introduces a **core idea in category theory**:

> We study objects **by their relationships (morphisms)** rather than internal structure.

Because category theory abstracts over many fields (sets, types, propositions, etc.), we can’t inspect what’s “inside” objects — we can only reason in terms of arrows.

## Example: Isomorphisms in $\text{Set}$

In the category $\text{Set}$:

* **Objects:** sets
* **Arrows:** functions

Two sets $A$ and $B$ are said to have the same structure (same number of elements) if there exist functions:
$$
f: A \to B \quad \text{and} \quad f^{-1}: B \to A
$$
such that:
$$
f^{-1} \circ f = \mathrm{id}_A \quad \text{and} \quad f \circ f^{-1} = \mathrm{id}_B
$$
This defines **isomorphism**: $A \cong B$.

An **isomorphism** expresses “sameness” of structure *purely in terms of morphisms*, not internal data.
Thus, **isomorphic objects are indistinguishable categorically**.

## Example: Terminal Objects via Functions

A **singleton set** (a set with one element) can be characterized *categorically*:

* For any set $A$, there exists exactly **one function** $f: A \to 1$.
* This uniqueness defines a **terminal object** in $\text{Set}$.

### Definition

An object $1$ in a category $\mathcal{C}$ is **terminal** if:
$$
\forall A \in \mathcal{C}, ; \exists! , f: A \to 1
$$
(i.e., exactly one arrow from any object to it).

### Properties

* In $\text{Set}$, all singleton sets are terminal and **isomorphic** to each other.
* In any category, **all terminal objects are isomorphic** (proof by uniqueness of arrows).

## Examples of Terminal Objects in Other Categories

| Category                           | Objects      | Arrows    | Terminal Object     | Reason                                              |
| ---------------------------------- | ------------ | --------- | ------------------- | --------------------------------------------------- |
| **Set**                            | Sets         | Functions | Any singleton set   | Exactly one function from any set to it             |
| **Proofs / Logic**                 | Propositions | Proofs    | **True** $\top$     | There’s exactly one proof from any proposition to ⊤ |
| **Functional Programming (Types)** | Types        | Functions | Singleton type `()` | Only one term `()` of that type                     |

Terminal objects are denoted $1$ because of their role as a categorical "unit".

## Universal Constructions

The **method of universal construction**, which defines mathematical structures by *how they uniquely relate to others*.

### Process

1. **Define a construction** (e.g., an object with certain arrows).
2. **Find a universal property** — a unique mapping that every other candidate must factor through.
3. The resulting object is **unique up to isomorphism**.

Such definitions are **portable** across categories (e.g., same definition of singleton applies to sets, types, propositions, etc.).

## Universal Construction of **Conjunction (Product)**

### In Logic (Proof Category)

* **Objects:** Propositions
* **Arrows:** Proofs
* **Goal:** Define the conjunction $A \land B$.

#### Step 1: The Construction

We want an object $A \land B$ with arrows:
$$
\pi_1: A \land B \to A, \quad \pi_2: A \land B \to B
$$
representing that from a proof of $A \land B$, we can extract proofs of both $A$ and $B$.

#### Step 2: The Universal Property

For any other proposition $X$ with proofs:
$$
f_1: X \to A, \quad f_2: X \to B
$$
there must exist a **unique** arrow $f: X \to A \land B$ such that:
$$
\pi_1 \circ f = f_1 \quad \text{and} \quad \pi_2 \circ f = f_2
$$
This makes the following diagram **commute**:

> [!note]
> Need image

When all paths from top to bottom yield the same result, we say the diagram **commutes**.

#### Step 3: Generalization

This construction defines the **product** $A \times B$ in any category.

## The Product in Functional Programming

In the **category of types and functions**, the **product type** is defined via the same universal property.

Let:
* $A = \text{String}$
* $B = \text{Integer}$

Then the product $A \times B$ is the **pair type** `(String, Integer)` with:
$$
\pi_1(\text{str}, \text{int}) = \text{str}, \quad \pi_2(\text{str}, \text{int}) = \text{int}
$$
For any type $X$ with:
$$
f_1: X \to \text{String}, \quad f_2: X \to \text{Integer}
$$
there exists a **unique function**:
$$
f(x) = (f_1(x), f_2(x))
$$
satisfying:
$$
\pi_1 \circ f = f_1, \quad \pi_2 \circ f = f_2
$$
Thus, `(String, Integer)` is the **categorical product** of `String` and `Integer`.

By symmetry, `(B, A)` is **isomorphic** to `(A, B)` via the swap function.
