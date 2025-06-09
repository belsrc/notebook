---
tags:
  - math
  - comp-sci
---
### `Array.every`

$$
A \iff \forall x \in S : P(x)
$$

> Unicode representation:
> 𝐴 ⟺ ∀𝑥 ∈ 𝑆 : 𝑃(𝑥)

or

$$
A = \text{True} \iff \forall x \in S : P(x) = \text{True}
$$

- `A` represents the proposition "A is true"
- `\iff` ($\iff$) means "if and only if" (biconditional)
- `\forall` ($\forall$) means "for all" (universal quantifier)
- `x \in S` ($x \in S$) means "x is an element of set S"
- `P(x)` represents the property/condition that must be true for element x

$$
\text{array.every(callback)} \iff \forall x \in \text{array} : \text{callback}(x) = \text{true}
$$

```javascript
const numbers = [2, 4, 6, 8];
const allEven = numbers.every(x => x % 2 === 0); // true
```

This corresponds to:

$$
\text{allEven} \iff \forall x \in \{2, 4, 6, 8\} : (x \bmod 2 = 0)
$$
---

### `Array.some`

$$
A \iff \exists x \in S : P(x)
$$

> Unicode representation:
> 𝐴 ⟺ ∃𝑥 ∈ 𝑆 : 𝑃(𝑥)

or

$$
A  = \text{True} \iff \exists x \in S : P(x)
$$

- `A` represents the proposition "A is true"
- `\iff` ($\iff$) means "if and only if" (biconditional)
- `\exists` ($\exists$) means "there exists" (existential quantifier)
- `x \in S` ($x \in S$) means "x is an element of set S"
- `P(x)` represents the property/condition that must be true for element x

$$
\text{array.some(callback)} \iff \exists x \in \text{array} : \text{callback}(x) = \text{true}
$$

```javascript
const numbers = [1, 3, 4, 7];
const hasEven = numbers.some(x => x % 2 === 0); // true (because of 4)
```

This corresponds to:

$$
\text{hasEven} \iff \exists x \in \{1, 3, 4, 7\} : (x \bmod 2 = 0)
$$
---

### `Array.filter`

#### Set builder notation.

$$
A = \{x \in S : P(x)\}
$$

- `S` is the original set/array
- `P(x)` is the predicate function
- The result is a new set containing only elements that satisfy the predicate

$$
\text{array.filter(predicate)} = \{x \in \text{array} : \text{predicate}(x) = \text{true}\}
$$

```js
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(x => x % 2 === 0); // [2, 4, 6]
```

This corresponds to:

$$
\text{evens} = \{x \in \{1, 2, 3, 4, 5, 6\} : x \bmod 2 = 0\} = \{2, 4, 6\}
$$

#### Sequence notation

This is, practically, the better notation. As it maintains ordering and duplications.

$$
\text{filter}((x_i)_{i=1}^n, P) \text{ where } P(x) = f
$$

- `x_i` is the element at position `i`
- `i=1` is the starting index (first element)
- `n` is the ending index (last element)
- `P` is the proposition that must be true for the element to be added to the new sequence
- `where P(x) =` is the definition of the proposition

Using the same JS example above:

$$
\text{evens} = \text{filter}((x_i)_{i=1}^6, P) \text{ where } P(x) = (x \bmod 2 = 0) = (2, 4, 6)
$$

---

### `Array.map`

#### Sequence notation

$$
\text{map}((x_i)_{i=1}^n, f) \text{ where } f(x) = g
$$

- `x_i` is the element at position `i`
- `i=1` is the starting index (first element)
- `n` is the ending index (last element)
- `f` is the function given each element of the sequence
- `where f(x) =` is the definition of the function

```ts
const numbers = [3, 7, 2, 9];
const doubled = input.map(x => x * 2);
```

This corresponds to:

$$
\text{doubled} = \text{map}((x_i)_{i=1}^4, f) \text{ where } f(x) = 2x = (6, 14, 4, 18)
$$

---

### `if` Assignment / `case` Fall Through

$$
R := x; \quad \forall i \in \{j, j+1, \ldots, n\} : \text{if } A_i \text{ then } R := B_i
$$

> Unicode representation:
> 𝑅 := 𝑥;  ∀𝑖 ∈ {𝑗, 𝑗+1, …, 𝑛} : if 𝐴ᵢ then 𝑅 := 𝐵ᵢ

where `x` is the initial value and `j = min{i : Aᵢ = true}`.

```js
let R; // undefined
if (A1) R = B1; // skips
if (A2) R = B2; // executes: R = B2
if (A3) R = B3; // skips
if (A4) R = B4; // executes: R = B4
if (A5) R = B5; // executes: R = B5
// Final: R = B5
```

```js
let R; // undefined
switch(true) {
	case A1: R = B1; // skips
	case A2: R = B2; // executes: R = B2
	case A3: R = B3; // skips
	case A4: R = B4; // executes: R = B4
	case A5: R = B5; // executes: R = B5
}
// Final: R = B5
```

Given conditions `A₁=false`, `A₂=true`, `A₃=false`, `A₄=true`, `A₅=true`.

---

### `if`/`case` Return

$$
\forall i \in \{1, 2, \ldots, n\} : \text{if } A_i \text{ then return } B_i
$$

> Unicode representation:
> ∀𝑖 ∈ {1, 2, …, 𝑛} : if 𝐴ᵢ then return 𝐵ᵢ

```js
if (A1) return B1;
if (A2) return B2;  // returns here if A2 is true
if (A3) return B3;  // never reached
if (A4) return B4;  // never reached
```

```js
switch(true) {
	case A1: return B1; // skips
	case A2: return B2; // returns here if A2 is true
	case A3: return B3; // never reached
	case A4: return B4; // never reached
}
```

Given conditions `A₁=false`, `A₂=true`, `A₃=false`, `A₄=true`.

### `if...else`

#### Piecewise Function
$$
G = \begin{cases} \forall x \in \{R_1, R_2, \ldots, R_n\} : P(x) & \text{if } C \\ \exists x \in \{R_1, R_2, \ldots, R_n\} : P(x) & \text{otherwise} \end{cases}
$$

#### Logical Implication
$$
G \iff (C \rightarrow \forall x \in \{R_1, R_2, \ldots, R_n\} : P(x)) \land (\neg C \rightarrow \exists x \in \{R_1, R_2, \ldots, R_n\} : P(x))
$$

#### Set-Based Conditional
$$
G = \begin{cases} \text{True} & \text{if } C \land \forall x \in S : P(x) \\ \text{True} & \text{if } \neg C \land \exists x \in S : P(x) \\ \text{False} & \text{otherwise} \end{cases}
$$
Where $S = \{R_1, R_2, ..., R_n\}$

#### Algorithmic Notation
$$
\text{if } C \text{ then } G := \forall x \in \{R_1, R_2, \ldots, R_n\} : P(x) \text{ else } G := \exists x \in \{R_1, R_2, \ldots, R_n\} : P(x)
$$
