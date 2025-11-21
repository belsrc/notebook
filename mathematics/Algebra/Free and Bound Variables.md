---
tags:
  - math
  - algebra
  - comp-sci
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Free_variables_and_bound_variables
---
In mathematics, and other disciplines involving formal languages, a variable may be said to be either free or bound. A free variable is a notation that specifies places in an expression where substitution may take place and is not a parameter of this or any container expression.

In computer science, the term free variable refers to variables used in a function that are neither local variables nor parameters of that function. In other words, a non-local variable.

A variable is bound, in contrast, if the value of that variable symbol has been bound to a specific value or range of values in the domain of discourse or universe. A variable overall is bound if at least one occurrence of it is bound. For example, consider the following expression in which both variables are bound by logical quantifiers:

$\forall\  y \ \exists\  x \  (x = \sqrt{y})$

This expression evaluates to false if the domain of $x$ and $y$ is the real numbers, but true if the domain is the complex numbers.

## Formal Explanation

Variable-binding mechanisms occur in different contexts in mathematics, logic and computer science. In all cases, however, they are purely syntactic properties of expressions and variables in them. For this section we can summarize syntax by identifying an expression with a tree whose leaf nodes are variables, constants, function constants or predicate constants and whose non-leaf nodes are logical operators. Variable-binding operators are logical operators that occur in almost every formal language.

In the [lambda calculus](../../mathematics/lambda%20calculus/Lambda%20Calculus), $x$ is a bound variable in the term $M = \lambda x.T$ and a free variable in the term $T$. We say $x$ is bound in $M$ and free in $T$. If $T$ contains a subterm $\lambda x. U$ then $x$ is rebound in this term. This nested, inner binding of $x$ is said to "[shadow](../../computer%20science/programming%20lang%20theory/Variable%20Shadowing)" the outer binding. Occurrences of $x$ in $U$ are free occurrences of the new $x$.

Variables bound at the top level of a program are technically free variables within the terms to which they are bound but are often treated specially because they can be compiled as fixed addresses. Similarly, an identifier bound to a recursive function is also technically a free variable within its own body but is treated specially.

A closed term is one containing no free variables.

## Examples

$\sum_{k=1}^{10} f(k,n)$

$n$ is a free variable and $k$ is a bound variable.

$\int_{0}^{\infty} x^{y-1}e^{-x}dx$

$y$ is a free variable and $x$ is a bound variable.

## Variable-binding Operators

$\sum_{x \ \in\  S}$

$\int_{0}^{\infty} ...dx$

$\lim_{x \ \to\  0}$

$\forall\  x$

$\exists\  x$ 

Are some common variable-binding operators. Each of them binds the variable $x$ for some set $S$.
