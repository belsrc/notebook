## PART 1: FOUNDATIONS

### Lesson 1.1: Vector Spaces & Properties

#### Core Concept

A **vector space** (or linear space) is a collection of objects called vectors that can be added together and multiplied by scalars (numbers), following specific rules.

**Formal Definition**: A vector space $V$ over a field $\mathbb{F}$ (typically $\mathbb{R}$ or $\mathbb{C}$) consists of:
- A set of vectors
- Two operations: vector addition $(+)$ and scalar multiplication $(\cdot)$

These operations must satisfy **8 axioms** for all vectors $\mathbf{u}, \mathbf{v}, \mathbf{w} \in V$ and scalars $a, b \in \mathbb{F}$:

#### Addition Axioms

1.  **Closure:** $$u + v \in V$$
2.  **Commutativity:** $$u + v = v + u$$
3.  **Associativity:** $$(u + v) + w = u + (v + w)$$
4.  **Identity:** $$\exists \mathbf{0} \text{ such that } v + \mathbf{0} = v$$
5.  **Inverse:** $$\exists -v \text{ such that } v + (-v) = \mathbf{0}$$

#### Scalar Multiplication Axioms

6.  **Closure:** $$a \cdot v \in V$$
7.  **Distributivity:** $$a \cdot (u + v) = a \cdot u + a \cdot v \\ (a + b) \cdot v = a \cdot v + b \cdot v$$
8.  **Compatibility:** $$a \cdot (b \cdot v) = (ab) \cdot v \\ 1 \cdot v = v$$

#### The $\mathbb{R}^n$ Notation

##### What It Means

$\mathbb{R}$ = the set of all real numbers: $\{\ldots, -2.5, -1, 0, 1, \pi, 3.7, \ldots\}$

$\mathbb{R}^n$ = the set of all **ordered n-tuples** of real numbers

More formally:

$$\mathbb{R}^n = \left\{ \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_n \end{bmatrix} \mid x_1, x_2, \ldots, x_n \in \mathbb{R} \right\}$$

##### Concrete Examples

**$\mathbb{R}^1$**: Just the real number line
- Elements: $5$, $-3.2$, $\pi$, etc.
- Dimension: 1

**$\mathbb{R}^2$**: The plane (ordered pairs)
- Elements: $\begin{bmatrix} 3 \\ -2 \end{bmatrix}$, $\begin{bmatrix} 0 \\ 5.7 \end{bmatrix}$, $\begin{bmatrix} \pi \\ 1 \end{bmatrix}$
- Dimension: 2
- Visual: Cartesian coordinate system (x, y)

**$\mathbb{R}^3$**: Three-dimensional space (ordered triples)
- Elements: $\begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}$, $\begin{bmatrix} -5 \\ 0 \\ 2.8 \end{bmatrix}$
- Dimension: 3
- Visual: 3D coordinates (x, y, z)

**$\mathbb{R}^4$**: Four-dimensional space (can't visualize easily, but mathematically valid!)
- Elements: $\begin{bmatrix} 1 \\ 2 \\ 3 \\ 4 \end{bmatrix}$
- Dimension: 4
- Used in: spacetime in physics, machine learning feature vectors

##### Key Insight

The superscript $n$ tells you:
1. **How many components** each vector has
2. **The dimension** of the space
3. **How many coordinates** you need to specify a point

##### Why "Ordered" Matters

In $\mathbb{R}^2$:
- $\begin{bmatrix} 3 \\ 5 \end{bmatrix} \neq \begin{bmatrix} 5 \\ 3 \end{bmatrix}$

The position (order) of each number matters! The first is point (3, 5), the second is point (5, 3).

#### Real-World Example

**Example 1**: $\mathbb{R}^3$ - Three-dimensional Euclidean space

Vectors: $\mathbf{v} = \begin{bmatrix} x \\ y \\ z \end{bmatrix}$ where $x, y, z \in \mathbb{R}$

Addition: $\begin{bmatrix} x_1 \\ y_1 \\ z_1 \end{bmatrix} + \begin{bmatrix} x_2 \\ y_2 \\ z_2 \end{bmatrix} = \begin{bmatrix} x_1 + x_2 \\ y_1 + y_2 \\ z_1 + z_2 \end{bmatrix}$

Scalar multiplication: $c \cdot \begin{bmatrix} x \\ y \\ z \end{bmatrix} = \begin{bmatrix} cx \\ cy \\ cz \end{bmatrix}$

This represents physical space, forces, velocities, etc.

**Example 2**: $P_2$ - Polynomials of degree ≤ 2

Vectors are polynomials like $p(x) = ax^2 + bx + c$

Addition: $(2x^2 + 3x + 1) + (x^2 - x + 4) = 3x^2 + 2x + 5$

Scalar multiplication: $3(x^2 + 2x) = 3x^2 + 6x$

This is used in curve fitting, approximation theory, etc.

##### Exercises

**Q1**: Consider the set of all $2 \times 2$ matrices with real entries, with standard matrix addition and scalar multiplication. Is this a vector space? Why or why not?

Yes, the set of all $2 \times 2$ matrices forms a vector space. All 8 axioms hold with standard matrix addition and scalar multiplication. The zero matrix $\begin{bmatrix} 0 & 0 \\ 0 & 0 \end{bmatrix}$ serves as the identity.

**Q2**: What about the set of all positive real numbers with operations $a \oplus b = ab$ (multiplication as "addition") and $c \odot a = a^c$ (exponentiation as "scalar multiplication")? Does the identity axiom hold?

Yes, the identity axiom holds! The identity element is $1$ because $a \oplus 1 = a \cdot 1 = a$. (Fun fact: this actually *is* a valid vector space - it's isomorphic to $\mathbb{R}$ with normal operations via the logarithm map!)

**Task**: Verify that the zero vector axiom (axiom 4) holds for $\mathbb{R}^2$.

Specifically, show that $\mathbf{0} = \begin{bmatrix} 0 \\ 0 \end{bmatrix}$ satisfies $\mathbf{v} + \mathbf{0} = \mathbf{v}$ for any $\mathbf{v} = \begin{bmatrix} x \\ y \end{bmatrix}$.

$\begin{bmatrix} x \\ y \end{bmatrix} + \begin{bmatrix} 0 \\ 0 \end{bmatrix} = \begin{bmatrix} x+0 \\ y+0 \end{bmatrix} = \begin{bmatrix} x \\ y \end{bmatrix}$


