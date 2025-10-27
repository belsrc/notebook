---
tags:
  - math
date: 2025-10-27
gardening: 🌱
reference:
---
## PART 1: VECTORS & VECTOR SPACES

### Lesson 1.1: Vectors and Geometric Interpretation

#### Core Concept

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

#### Core Operations

We can perform two fundamental operations on vectors:

### 1. Vector Addition

Given $\mathbf{u} = \begin{bmatrix} u_1 \\ u_2 \\ \vdots \\ u_n \end{bmatrix}$ and $\mathbf{v} = \begin{bmatrix} v_1 \\ v_2 \\ \vdots \\ v_n \end{bmatrix}$

$$\mathbf{u} + \mathbf{v} = \begin{bmatrix} u_1 + v_1 \\ u_2 + v_2 \\ \vdots \\ u_n + v_n \end{bmatrix}$$

**Example:**
$$\begin{bmatrix} 3 \\ 2 \end{bmatrix} + \begin{bmatrix} 1 \\ 4 \end{bmatrix} = \begin{bmatrix} 4 \\ 6 \end{bmatrix}$$

**Geometric interpretation:** Place the tail of $\mathbf{v}$ at the head of $\mathbf{u}$. The sum is the vector from the tail of $\mathbf{u}$ to the head of $\mathbf{v}$ (tip-to-tail method).

![](../../images/math/linear-addition.png)

### 2. Scalar Multiplication

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
