---
tags:
  - math
  - linear-alg
date: 2025-11-22
gardening: 🌱
reference:
  - https://brilliant.org/courses/vectors/?from_llp=advanced-math
---


So you have two vectors: $\vec{a} = \begin{bmatrix} 4 \\ 6 \end{bmatrix}$  and $\vec{b} = \begin{bmatrix} 12 \\ 10 \end{bmatrix}$. You need to find the vector length between the two tips of vectors. So you need to subtract their components. $(12 - 4, 10 - 6) = (8, 4)$  which gives you the vector that joins the other two $\vec{a} - \vec{b}$. Then you need to apply Pythagorean to get the length of that vector $\sqrt{8^2 + 4^2}$.  Or simply use Euclidean distance, which just merges the two steps $\sqrt{(12 - 4)^2 + (10 - 6)^2}$ .

```typescript
const pythagoreanTheoroem = (vec: Vector2D) =>
  Math.sqrt(vec[0]^2 + vec[1]^2); // vector length
	
const euclidDistance = (p1: Point2D, p2: Point2D) =>
  Math.sqrt((p2[0] - p1[0])^2 + (p2[1] - p1[1])^2)
```

So Euclidean distance is the Pythagorean theorem of the difference between vectors (or points).

$\vec{a} = \begin{bmatrix} 4 \\ 6 \end{bmatrix} \ \ \vec{b} = \begin{bmatrix} 12 \\ 10 \end{bmatrix}$

$\sqrt{(12 - 4)^2 + (10 - 6)^2} = 3.4641016151377544$

When we multiply a vector $\vec{a}$ by a positive scalar $k$, the resulting vector $k\vec{a}$ is $k$ times as long and points in the same direction. So,
- when $0 < k < 1$, the scaled vector is shorter, and
- when $k > 1$, the scaled vector is longer.

To find the mid-point between two vectors $\vec{v}$ and $\vec{w}$, multiply the difference by $\frac{1}{2}$.

$\frac{1}{2}(\vec{w} - \vec{v})$.

$\vec{v} = \begin{bmatrix} 2 \\ 5 \end{bmatrix}$  and  $\vec{w} = \begin{bmatrix} 6 \\ 7 \end{bmatrix}$

$\vec{m} = \frac{1}{2} \begin{bmatrix} 6 - 2 \\ 7 - 5 \end{bmatrix}$ = $\begin{bmatrix} 2 \\ 1 \end{bmatrix}$

And if you want to find the vector from the common origin to the $\vec{m}$, you add $\vec{v}$.

$\vec{n} = \vec{v} + \frac{1}{2}(\vec{w} - \vec{v})$

$(2, 5) + \frac{1}{2}(6 - 2, 7 - 5) = (4, 6)$

Or, short-hand to get $\vec{n}$:

$\vec{n} = \frac{1}{2}\vec{v} + \frac{1}{2}\vec{w}$

$\frac{1}{2}\begin{bmatrix} 2 \\ 5 \end{bmatrix} + \frac{1}{2}\begin{bmatrix} 6 \\ 7 \end{bmatrix} = \begin{bmatrix} 1 \\ 2.5 \end{bmatrix} + \begin{bmatrix} 3 \\ 3.5 \end{bmatrix} = \begin{bmatrix} 4 \\ 6 \end{bmatrix}$

Or, simplifying again:

$\vec{n} = \begin{bmatrix} (\vec{v}_1 + \vec{w}_1) \div 2 \\ (\vec{v}_2 + \vec{w}_2) \div 2 \end{bmatrix}$

$\begin{bmatrix} \frac{2 + 6}{2} \\ \frac{5 + 7}{2} \end{bmatrix} = \begin{bmatrix} \frac{8}{2} \\ \frac{12}{2} \end{bmatrix} = \begin{bmatrix} 4 \\ 6 \end{bmatrix}$

mid-point vector $\vec{p} = \frac{1}{2}(\vec{v} + \vec{w})$

```typescript
const scaleVector = (vec: Vector2D, scalar: number) =>
  [vec[0] * scalar, vec[1] * scalar];
	
const midvector = (a: Vector2D, b: Vector2D) =>
  scaleVector([a[0] + b[0], a[1] + b[1]], 0.5);
```
