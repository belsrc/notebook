---
tags:
  - math
date: 2025-10-27
gardening: 🌱
reference:
---
## PART 1: VECTORS & VECTOR SPACES

### Lesson 1.1: Vectors and Geometric Interpretation

A **vector** is fundamentally an object that has both *magnitude* (length) and *direction*. We can think of it in multiple equivalent ways:

**Three perspectives on vectors:**

1. **Geometric:** An arrow in space pointing from one location to another
2. **Algebraic:** An ordered list of numbers (coordinates)
3. **Abstract:** An element of a vector space (we'll formalize this later)

Additionally, the Computer Science perspective.
- An ordered list, tuple

#### Notation and Representation

In $\mathbb{R}^n$ (n-dimensional real space), we write vectors as:

$$\mathbf{v} = \begin{bmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{bmatrix}$$

**Example in $\mathbb{R}^2$ (2D):**
$$\mathbf{v} = \begin{bmatrix} 3 \\ 2 \end{bmatrix}$$

This means: starting from the origin (0,0), move 3 units right and 2 units up.

**Example in $\mathbb{R}^3$ (3D):**
$$\mathbf{w} = \begin{bmatrix} 1 \\ -2 \\ 4 \end{bmatrix}$$

This means: from origin, move 1 unit along x-axis, -2 units along y-axis, 4 units along z-axis.

#### Real-World Connections

- **ML/Data Science:** A data point with features `[age, income, credit_score]`
- **Graphics:** Position of a vertex, or direction of light/surface normal
- **Physics:** Velocity, force, displacement
- **Programming:** RGB color values `[255, 128, 0]`

#### Questions

**Q1:** If I have a vector $\mathbf{p} = \begin{bmatrix} 5 \\ 5 \end{bmatrix}$ representing a position, and I draw it starting from the origin, what direction does it point? (Think about the angle or describe it directionally)

**A:** 45° northeast. The components are equal, so it bisects the angle.

**Q2:** Are these two vectors the same?
- Vector A: Arrow from point (1,1) to point (4,3)
- Vector B: Arrow from point (0,0) to point (3,2)

**A:** Vectors represent *displacement* (direction + magnitude), not absolute position. Both arrows have the same "movement" of 3 right, 2 up. We say vectors are **translation-invariant** - they can "slide" anywhere while remaining the same vector.

**Q3:** In machine learning, if you have a data point representing a house as $\begin{bmatrix} 2000 \\ 3 \\ 2 \end{bmatrix}$ (square feet, bedrooms, bathrooms), what does each component represent and why is the vector representation useful?

**A:** Vector representation enables **computation**. Once data is in vector form:
- We can measure *distance* between houses (similarity)
- We can perform *operations* (averaging prices across regions)
- We can apply *transformations* (normalize features for ML)
- We can use *geometric intuition* (houses cluster in feature space)
The vector form makes the data **computable** rather than just descriptive.

#### Exercise

Consider a game where a character is at position $(10, 5)$ and needs to move to position $(13, 9)$.

**Tasks:**
1. Write the displacement vector $\mathbf{d}$ that represents this movement

- $\begin{bmatrix} 13 \\ 9 \end{bmatrix} - \begin{bmatrix} 10 \\ 5 \end{bmatrix} = \begin{bmatrix} 3 \\ 4 \end{bmatrix} = \mathbf{d}$

2. If the character starts at (0, 0) instead, what position would this same displacement vector $\mathbf{d}$ take them to?

- $(3, 4)$

3. What does this tell you about the difference between *position* and *displacement*?

- **Displacement** is *always* a vector (direction + magnitude of movement)
- **Position** is a *point* in space
- **Position vector** is the displacement from origin to that point

```
Position = "where you are" (a point)
Displacement = "how you moved" (a vector)
Position vector = displacement from origin to a point
```

## Lesson 1.2: Vector Operations and Properties

We can perform two fundamental operations on vectors:

#### 1. Vector Addition

Given $\mathbf{u} = \begin{bmatrix} u_1 \\ u_2 \\ \vdots \\ u_n \end{bmatrix}$ and $\mathbf{v} = \begin{bmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{bmatrix}$

$$\mathbf{u} + \mathbf{v} = \begin{bmatrix} u_1 + v_1 \\ u_2 + v_2 \\ \vdots \\ u_n + v_n \end{bmatrix}$$

**Example:**
$$\begin{bmatrix} 3 \\ 2 \end{bmatrix} + \begin{bmatrix} 1 \\ 4 \end{bmatrix} = \begin{bmatrix} 4 \\ 6 \end{bmatrix}$$

**Geometric interpretation:** Place the tail of $\mathbf{v}$ at the head of $\mathbf{u}$. The sum is the vector from the tail of $\mathbf{u}$ to the head of $\mathbf{v}$ (tip-to-tail method).

![](../../images/math/linear-addition.png)

#### 2. Scalar Multiplication

Given scalar $c \in \mathbb{R}$ and vector $\mathbf{v}$:

$$c\mathbf{v} = \begin{bmatrix} cv_1 \\ cv_2 \\ \vdots \\ cv_n \end{bmatrix}$$

**Example:**
$$3 \begin{bmatrix} 2 \\ -1 \end{bmatrix} = \begin{bmatrix} 3 \times 2 \\ 3 \times -1 \end{bmatrix} = \begin{bmatrix} 6 \\ -3 \end{bmatrix}$$

**Geometric interpretation:** 
- If $c > 1$: stretches the vector
- If $0 < c < 1$: shrinks the vector
- If $c < 0$: flips direction and scales
- If $c = 0$: produces the zero vector $\mathbf{0}$

![](../../images/math/linear-multiplication.png)

#### Algebraic Properties

These operations satisfy important properties:

**Commutativity of addition:**
$$\mathbf{u} + \mathbf{v} = \mathbf{v} + \mathbf{u}$$

**Associativity of addition:**
$$(\mathbf{u} + \mathbf{v}) + \mathbf{w} = \mathbf{u} + (\mathbf{v} + \mathbf{w})$$

**Distributivity (scalar over addition):**
$$c(\mathbf{u} + \mathbf{v}) = c\mathbf{u} + c\mathbf{v}$$

**Distributivity (addition over scalar):**
$$(c + d)\mathbf{v} = c\mathbf{v} + d\mathbf{v}$$

**Associativity of scalar multiplication:**
$$c(d\mathbf{v}) = (cd)\mathbf{v}$$

**Identity elements:**
- Additive identity: $\mathbf{v} + \mathbf{0} = \mathbf{v}$
- Multiplicative identity: $1\mathbf{v} = \mathbf{v}$

#### Real-World Applications

**Graphics:** Combining transformations

```typescript
type Vec3 = [number, number, number];

const position: Vec3 = [10, 5, 0];
const velocity: Vec3 = [2, -1, 0];  // units per frame
const gravity: Vec3 = [0, -0.5, 0];

// Update position after one frame
const acceleration = gravity;
const newVelocity: Vec3 = [
  velocity[0] + acceleration[0],
  velocity[1] + acceleration[1],
  velocity[2] + acceleration[2]
];
const newPosition: Vec3 = [
  position[0] + newVelocity[0],
  position[1] + newVelocity[1],
  position[2] + newVelocity[2]
];
```

#### Questions

**Q1:** Compute: $2\begin{bmatrix} 1 \\ 3 \\ -2 \end{bmatrix} + 3\begin{bmatrix} 0 \\ 1 \\ 1 \end{bmatrix}$

**A:** $2\begin{bmatrix} 1 \\ 3 \\ -2 \end{bmatrix} + 3\begin{bmatrix} 0 \\ 1 \\ 1 \end{bmatrix} = \begin{bmatrix} 2 \\ 6 \\ -4 \end{bmatrix} + \begin{bmatrix} 0 \\ 3 \\ 3 \end{bmatrix} = \begin{bmatrix} 2 \\ 9 \\ -1 \end{bmatrix}$

**Q2:** If $\mathbf{u}$ represents velocity "5 m/s east, 3 m/s north" and you double it ($2\mathbf{u}$), what does that mean physically? What about $-1\mathbf{u}$?

**A:** 
- $2\begin{bmatrix} 5 \\ 3 \end{bmatrix} = \begin{bmatrix} 10 \\ 6 \end{bmatrix}$ : It is moving twice as fast.
- $-1\begin{bmatrix} 5 \\ 3 \end{bmatrix} = \begin{bmatrix} -5 \\ -3 \end{bmatrix}$ : It's direction is reversed.

Scalar multiplication scales magnitude and (if negative) reverses direction.

**Q3:** In graphics, if you apply transformation $\mathbf{T_1}$ then $\mathbf{T_2}$ to a vertex, is that the same as applying $\mathbf{T_2}$ then $\mathbf{T_1}$?

**A:** If $\mathbf{T_1}$ and $\mathbf{T_2}$ are **translations** (pure vector additions), then YES, order doesn't matter:

```
vertex + T₁ + T₂ = vertex + T₂ + T₁  ✓ (commutative)
```

BUT if they're general transformations (rotations, scaling, shearing), order DOES matter. Rotate a rectangle 45°, then scale by 2× horizontally gives a different result than scaling first, then rotating. Vector addition is commutative, but **transformation composition** (which uses matrix multiplication, coming later) is NOT commutative.

#### Exercise

Given vectors:
$$\mathbf{a} = \begin{bmatrix} 2 \\ -1 \\ 3 \end{bmatrix}, \quad \mathbf{b} = \begin{bmatrix} 1 \\ 2 \\ 0 \end{bmatrix}, \quad \mathbf{c} = \begin{bmatrix} -1 \\ 1 \\ 2 \end{bmatrix}$$

**Compute:**
1. $3\mathbf{a} - 2\mathbf{b}$

**A:** $3a - 2b = 3\begin{bmatrix} 2 \\ -1 \\ 3 \end{bmatrix} - 2\begin{bmatrix} 1 \\ 2 \\ 0 \end{bmatrix} = \begin{bmatrix} 6 \\ -3 \\ 9 \end{bmatrix} - \begin{bmatrix} 2 \\ 4 \\ 0 \end{bmatrix} = \begin{bmatrix} 4 \\ -7 \\ 9 \end{bmatrix}$

2. $\mathbf{a} + \mathbf{b} + \mathbf{c}$

**A:** $a + b + c = \begin{bmatrix} 2 \\ -1 \\ 3 \end{bmatrix} + \begin{bmatrix} 1 \\ 2 \\ 0 \end{bmatrix} + \begin{bmatrix} -1 \\ 1 \\ 2 \end{bmatrix} = \begin{bmatrix} 3 \\ 1 \\ 3 \end{bmatrix} + \begin{bmatrix} -1 \\ 1 \\ 2 \end{bmatrix} = \begin{bmatrix} 2 \\ 2 \\ 5 \end{bmatrix}$

3. Is there a scalar $k$ such that $k\mathbf{a} = \mathbf{b}$? Why or why not?

**A:** No. For one vector to be a scalar multiple of another, all components must scale by the same factor. When this is true, we say the vectors are parallel or collinear (lie on the same line through the origin).

4. In a physics simulation, you have forces $\mathbf{F_1}$, $\mathbf{F_2}$, $\mathbf{F_3}$ acting on an object. The net force is $\mathbf{F_{net}} = \mathbf{F_1} + \mathbf{F_2} + \mathbf{F_3}$. Why does vector addition model this correctly?

**A:** Forces are vectors because they have:
- **Magnitude** (strength of force)
- **Direction** (which way they push)

Vector addition correctly models force combination because of the **principle of superposition**: the effect of multiple forces acting simultaneously equals the sum of their individual effects. 

Geometrically, the tip-to-tail method shows this: if force $\mathbf{F_1}$ would move the object to point A, and then force $\mathbf{F_2}$ would move it from A to point B, the net effect is moving directly to point B - which is exactly $\mathbf{F_1} + \mathbf{F_2}$.

Component-wise addition works because forces in perpendicular directions don't interfere:

$\begin{bmatrix} 10 \\ 0 \\ 0 \end{bmatrix} + \begin{bmatrix} 0 \\ 5 \\ 0 \end{bmatrix} = \begin{bmatrix} 10 \\ 5 \\ 0 \end{bmatrix}$

The x-forces combine independently from y-forces. This **independence of components** is why vector addition models physical reality correctly.

## Lesson 1.3: Linear Combinations and Span

#### Linear Combinations

A **linear combination** of vectors $\mathbf{v_1}, \mathbf{v_2}, \ldots, \mathbf{v_k}$ is any expression of the form:

$$c_1\mathbf{v_1} + c_2\mathbf{v_2} + \cdots + c_k\mathbf{v_k}$$

where $c_1, c_2, \ldots, c_k$ are scalars (called **coefficients**).

**Example:** Given $\mathbf{v_1} = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$ and $\mathbf{v_2} = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$

These are all linear combinations:
- $2\mathbf{v_1} + 3\mathbf{v_2} = \begin{bmatrix} 2 \\ 3 \end{bmatrix}$
- $-1\mathbf{v_1} + 5\mathbf{v_2} = \begin{bmatrix} -1 \\ 5 \end{bmatrix}$
- $0\mathbf{v_1} + 0\mathbf{v_2} = \begin{bmatrix} 0 \\ 0 \end{bmatrix}$ (the zero vector)

Think of vectors as **ingredients** and scalars as **amounts**. A linear combination is a **recipe**:

$$\text{Recipe} = (\text{amount}_1 \times \text{ingredient}_1) + (\text{amount}_2 \times \text{ingredient}_2) + \cdots$$

**Example in $\mathbb{R}^2$:**

Let's say we have:
- $\mathbf{v_1} = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$ (move 1 unit right)
- $\mathbf{v_2} = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$ (move 1 unit up)

Now we make a linear combination: $3\mathbf{v_1} + 2\mathbf{v_2}$

$$3\begin{bmatrix} 1 \\ 0 \end{bmatrix} + 2\begin{bmatrix} 0 \\ 1 \end{bmatrix} = \begin{bmatrix} 3 \\ 0 \end{bmatrix} + \begin{bmatrix} 0 \\ 2 \end{bmatrix} = \begin{bmatrix} 3 \\ 2 \end{bmatrix}$$

**What happened geometrically?**
1. Scale $\mathbf{v_1}$ by 3: go 3 units right
2. Scale $\mathbf{v_2}$ by 2: go 2 units up
3. Add them: end up at position (3, 2)

![](../../images/math/linear-combination.png)

#### More Examples

**Example 1:** Different coefficients, same vectors

Using $\mathbf{v_1} = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$, $\mathbf{v_2} = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$:

- $1\mathbf{v_1} + 1\mathbf{v_2} = \begin{bmatrix} 1 \\ 1 \end{bmatrix}$
- $-2\mathbf{v_1} + 3\mathbf{v_2} = \begin{bmatrix} -2 \\ 3 \end{bmatrix}$
- $0\mathbf{v_1} + 5\mathbf{v_2} = \begin{bmatrix} 0 \\ 5 \end{bmatrix}$
- $0\mathbf{v_1} + 0\mathbf{v_2} = \begin{bmatrix} 0 \\ 0 \end{bmatrix}$ (zero vector always in span)

**Example 2:** Three vectors in $\mathbb{R}^3$

Let:
- $\mathbf{u} = \begin{bmatrix} 1 \\ 0 \\ 0 \end{bmatrix}$ (x-direction)
- $\mathbf{v} = \begin{bmatrix} 0 \\ 1 \\ 0 \end{bmatrix}$ (y-direction)  
- $\mathbf{w} = \begin{bmatrix} 0 \\ 0 \\ 1 \end{bmatrix}$ (z-direction)

A linear combination: $2\mathbf{u} - 3\mathbf{v} + 4\mathbf{w}$

$$2\begin{bmatrix} 1 \\ 0 \\ 0 \end{bmatrix} - 3\begin{bmatrix} 0 \\ 1 \\ 0 \end{bmatrix} + 4\begin{bmatrix} 0 \\ 0 \\ 1 \end{bmatrix} = \begin{bmatrix} 2 \\ -3 \\ 4 \end{bmatrix}$$

**Example 3:** What if vectors aren't perpendicular?

Let:
- $\mathbf{v_1} = \begin{bmatrix} 2 \\ 1 \end{bmatrix}$
- $\mathbf{v_2} = \begin{bmatrix} 1 \\ 3 \end{bmatrix}$

Compute: $2\mathbf{v_1} + 1\mathbf{v_2}$

$$2\begin{bmatrix} 2 \\ 1 \end{bmatrix} + 1\begin{bmatrix} 1 \\ 3 \end{bmatrix} = \begin{bmatrix} 4 \\ 2 \end{bmatrix} + \begin{bmatrix} 1 \\ 3 \end{bmatrix} = \begin{bmatrix} 5 \\ 5 \end{bmatrix}$$

The process is identical! Scale each vector, then add component-wise.

### The Span

The **span** of a set of vectors is the set of ALL possible linear combinations of those vectors.

$$\text{span}\{\mathbf{v_1}, \mathbf{v_2}, \ldots, \mathbf{v_k}\} = \{c_1\mathbf{v_1} + c_2\mathbf{v_2} + \cdots + c_k\mathbf{v_k} \mid c_i \in \mathbb{R}\}$$

**In plain English:** "The set of all vectors you can create by taking ANY real number coefficients and making a linear combination."

#### Key Word: ALL

The span includes **every possible choice** of coefficients:
- $c_1 = 0, c_2 = 0$ ✓
- $c_1 = 1, c_2 = 1$ ✓
- $c_1 = \pi, c_2 = -\sqrt{2}$ ✓
- $c_1 = 10^{100}, c_2 = -10^{-100}$ ✓

This is typically an **infinite set** of vectors.

#### Geometric Intuition

##### Case 1: Span of the Zero Vector in $\mathbb{R}^2$

$$\text{span}\left\{\begin{bmatrix} 0 \\ 0 \end{bmatrix}\right\} = \left\{c\begin{bmatrix} 0 \\ 0 \end{bmatrix} \;\middle|\; c \in \mathbb{R}\right\} = \left\{\begin{bmatrix} 0 \\ 0 \end{bmatrix}\right\}$$

No matter what scalar you multiply by, you get the zero vector. 

**Result:** Just a single point (the origin).

![](../../images/math/linear-span-1d.png)

##### Case 2: Span of One Non-Zero Vector in $\mathbb{R}^2$

$$\text{span}\left\{\begin{bmatrix} 2 \\ 1 \end{bmatrix}\right\} = \left\{c\begin{bmatrix} 2 \\ 1 \end{bmatrix} \;\middle|\; c \in \mathbb{R}\right\}$$

What does this look like?

- $c = 0$: $\begin{bmatrix} 0 \\ 0 \end{bmatrix}$ (origin)
- $c = 1$: $\begin{bmatrix} 2 \\ 1 \end{bmatrix}$
- $c = 2$: $\begin{bmatrix} 4 \\ 2 \end{bmatrix}$
- $c = 0.5$: $\begin{bmatrix} 1 \\ 0.5 \end{bmatrix}$
- $c = -1$: $\begin{bmatrix} -2 \\ -1 \end{bmatrix}$
- $c = -3$: $\begin{bmatrix} -6 \\ -3 \end{bmatrix}$

All these points lie on a **line** through the origin in the direction of $\mathbf{v}$.

**Result:** A 1-dimensional subspace (a line through origin).

![](../../images/math/linear-span-2d.png)

##### Case 3: Span of Two Vectors (Non-Parallel) in $\mathbb{R}^2$

Let:
- $\mathbf{v_1} = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$
- $\mathbf{v_2} = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$

$$\text{span}\{\mathbf{v_1}, \mathbf{v_2}\} = \left\{c_1\begin{bmatrix} 1 \\ 0 \end{bmatrix} + c_2\begin{bmatrix} 0 \\ 1 \end{bmatrix} \;\middle|\; c_1, c_2 \in \mathbb{R}\right\}$$

This equals:
$$\left\{\begin{bmatrix} c_1 \\ c_2 \end{bmatrix} \;\middle|\; c_1, c_2 \in \mathbb{R}\right\}$$

Since $c_1$ and $c_2$ can be ANY real numbers, this is **every point** in the plane!

**Result:** The entire 2D space $\mathbb{R}^2$.

**Why?** Because $\mathbf{v_1}$ and $\mathbf{v_2}$ point in different (non-parallel) directions. You can "slide" along $\mathbf{v_1}$ to control the x-coordinate and "slide" along $\mathbf{v_2}$ to control the y-coordinate independently.

##### Case 4: Span of Two Vectors (Parallel) in $\mathbb{R}^2$

Let:
- $\mathbf{v_1} = \begin{bmatrix} 1 \\ 2 \end{bmatrix}$
- $\mathbf{v_2} = \begin{bmatrix} 2 \\ 4 \end{bmatrix}$ (note: $\mathbf{v_2} = 2\mathbf{v_1}$)

$$c_1\mathbf{v_1} + c_2\mathbf{v_2} = c_1\begin{bmatrix} 1 \\ 2 \end{bmatrix} + c_2\begin{bmatrix} 2 \\ 4 \end{bmatrix}$$

But since $\mathbf{v_2} = 2\mathbf{v_1}$:

$$= c_1\mathbf{v_1} + c_2(2\mathbf{v_1}) = (c_1 + 2c_2)\mathbf{v_1}$$

Let $k = c_1 + 2c_2$. Then this is just $k\mathbf{v_1}$ for some scalar $k$.

**Result:** Still just a line (same as span of $\mathbf{v_1}$ alone). The second vector adds no "new directions."

![](../../images/math/linear-span-2d-par.png)


**Key insight:** $\mathbf{v_2}$ is **redundant** - it doesn't expand the span because it's already reachable using just $\mathbf{v_1}$.

##### Case 5: Span of Two Vectors (Non-Perpendicular but Non-Parallel) in $\mathbb{R}^2$

Let:
- $\mathbf{v_1} = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$
- $\mathbf{v_2} = \begin{bmatrix} 1 \\ 1 \end{bmatrix}$

These aren't perpendicular (angle ≠ 90°) and aren't parallel. Can we still span the plane?

Try to reach $\begin{bmatrix} 3 \\ 2 \end{bmatrix}$:

$$c_1\begin{bmatrix} 1 \\ 0 \end{bmatrix} + c_2\begin{bmatrix} 1 \\ 1 \end{bmatrix} = \begin{bmatrix} 3 \\ 2 \end{bmatrix}$$

$$\begin{bmatrix} c_1 + c_2 \\ c_2 \end{bmatrix} = \begin{bmatrix} 3 \\ 2 \end{bmatrix}$$

From second component: $c_2 = 2$  
From first component: $c_1 + 2 = 3 \Rightarrow c_1 = 1$

Yes! We can reach it: $1\mathbf{v_1} + 2\mathbf{v_2} = \begin{bmatrix} 3 \\ 2 \end{bmatrix}$

In fact, we can reach ANY point in $\mathbb{R}^2$ this way.

**Result:** Still the entire plane! Vectors don't need to be perpendicular to span a space - they just need to point in **non-parallel directions**.

#### Key Insight

The span tells us the **dimensionality of the space we can reach**:

In $\mathbb{R}^2$:
- 0 non-zero vectors → just origin (0D)
- 1 non-zero vector → line through origin (1D)
- 2 non-parallel vectors → entire plane (2D)

In $\mathbb{R}^3$:
- 0 non-zero vectors → just origin (0D)
- 1 non-zero vector → line through origin (1D)
- 2 non-parallel vectors → plane through origin (2D)
- 3 non-coplanar vectors → entire 3D space (3D)

#### Real-World Applications

**Computer Graphics - Color Mixing:**
```typescript
type RGB = [number, number, number];

const red: RGB = [1, 0, 0];
const green: RGB = [0, 1, 0];
const blue: RGB = [0, 0, 1];

// Any RGB color is a linear combination
const purple: RGB = [0.5 * 1 + 0 * 0 + 0.5 * 1, 
                      0.5 * 0 + 0 * 1 + 0.5 * 0,
                      0.5 * 0 + 0 * 0 + 0.5 * 1];
// = [0.5, 0, 0.5]

// span{red, green, blue} = all possible RGB colors
```