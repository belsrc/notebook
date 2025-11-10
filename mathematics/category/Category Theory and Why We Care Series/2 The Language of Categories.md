---
tags:
  - math
  - category-theory
date: 2025-11-08
gardening: 🌳
reference:
  - https://www.youtube.com/watch?v=5Ykrfqrxc8o
---
## 1. Formal Definition of a Category

A **category** $\mathcal{C}$ consists of:

* A **collection of objects**
  (denoted typically $A, B, C, \dots$)

* A **collection of arrows (morphisms)**
  (denoted $f, g, h, \dots$)

Each arrow $f$ has:

* A **domain**: $\text{dom}(f)$
* A **codomain**: $\text{cod}(f)$

We denote arrows as: $$f : A \to B$$
where $A = \text{dom}(f)$ and $B = \text{cod}(f)$.

### 1.1 Composition

If two arrows are *composable* (i.e. codomain of $f$ equals domain of $g$):
$$
f : A \to B, \quad g : B \to C
$$
then there must exist a **composite arrow**
$$
g \circ f : A \to C
$$
Composition is **not defined** when domains/codomains don’t align.

### 1.2 Associativity

Composition must satisfy:
$$
h \circ (g \circ f) = (h \circ g) \circ f
$$
for all composable $f, g, h$.

### 1.3 Identity

For every object $A$, there exists an **identity arrow**
$$
\text{id}_A : A \to A
$$
which acts as a **unit for composition**:
$$
f \circ \text{id}_A = f, \quad \text{id}_B \circ f = f
$$
for any arrow $f : A \to B$.

> [!NOTE]
> The *order of composition* matters — only one of these compositions is valid depending on domain/codomain alignment. 

## 2. Set-Theoretic Subtlety: “Collections” vs “Sets”

The lecturer warns that not all collections of objects/arrows can be formalized as **sets** due to **Russell’s paradox** — constructing “the set of all sets that do not contain themselves” leads to contradiction.

Hence, mathematicians distinguish:

* **Sets** (ordinary, small collections)
* **Classes** (larger, well-defined collections avoiding paradoxes)

So, a category technically consists of **classes** of objects and morphisms.

## 3. Example 1 — The Category $\text{Set}$

| Component       | Definition                                       |
| --------------- | ------------------------------------------------ |
| **Objects**     | All sets (e.g., ℕ, shapes, etc.)                 |
| **Arrows**      | All functions between sets                       |
| **Composition** | Function composition: $(g \circ f)(x) = g(f(x))$ |
| **Identity**    | $\text{id}_A(x) = x$                             |

**Verification:**

* **Associativity:** Function composition is inherently associative.
* **Identity laws:** For any $f: A \to B$,
  $$
  f \circ \text{id}_A = f, \quad \text{id}_B \circ f = f
  $$
  proven by direct substitution: $(f \circ \text{id}_A)(x) = f(\text{id}_A(x)) = f(x)$.

Thus, $\text{Set}$ satisfies all categorical axioms.

## 4. Example 2 — The Category of $\text{Proofs / Logic}$

| Component       | Definition                                                        |
| --------------- | ----------------------------------------------------------------- |
| **Objects**     | Logical propositions (true/false statements)                      |
| **Arrows**      | Proofs or deductions showing one proposition follows from another |
| **Composition** | Chaining deductions together (logical inference)                  |
| **Identity**    | Trivial proof: “Given P, therefore P”                             |

**Explanation:**

* Composition corresponds to combining proofs via **rules of inference**.
  Example: From $A \land B \land C$ infer $A \land B$, then infer $A$; composing gives a deduction $A \land B \land C \Rightarrow A$.
* **Identity proof:** Every proposition $P$ trivially entails itself $(P \Rightarrow P)$.

This forms a valid category, often denoted as $\text{Prop}$ or $\text{Proof}$, closely tied to *intuitionistic logic* and *type theory*.

## 5. Example 3 — Category from **Functional Programming**

| Component       | Definition                                              |
| --------------- | ------------------------------------------------------- |
| **Objects**     | Data types (e.g., `Boolean`, `Integer`, `String`, etc.) |
| **Arrows**      | Functions between types                                 |
| **Composition** | Function composition: `g ∘ f = x => g(f(x))`            |
| **Identity**    | `x => x` for any type                                   |

Parallel to $\text{Set}$, but interpreted in programming semantics:

* Data types correspond to objects.
* Pure (side-effect-free) functions correspond to morphisms.
* Function composition models category composition.

### Example (TypeScript-like):

```ts
type Bool = boolean;
type Int = number;

const isEven: (x: Int) => Bool = x => x % 2 === 0;
const toString: (b: Bool) => string = b => b ? 'true' : 'false';

const composed = (x: Int) => toString(isEven(x)) // g ∘ f
```

## 6. Conceptual Unification

The lecture emphasizes that **categories** emerge in multiple domains:

| Discipline  | Objects      | Arrows    |
| ----------- | ------------ | --------- |
| Mathematics | Sets         | Functions |
| Logic       | Propositions | Proofs    |
| Programming | Data types   | Functions |

These examples are **structurally analogous**, showcasing category theory’s role as a **unifying language** connecting different fields.
