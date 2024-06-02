---
tags:
  - functional
  - math
  - lambda
gardening: 🌱
---
First, lets define "calculus"

> A particular method or system of calculation or reasoning.

What you're probably used to hearing refered to as just "calculus" is actually refering to differential calculus and integral calculus (or as the group infinitesimal calculus). This is your intro derivatives, integrals, etc

## $\lambda$ Syntax

```
Identifier    expression ::= variable
Application                | expression expression
Abstraction                | λ variable.expression
Grouping                   | (expression)
```

### Variables

Exactly what you'd expect. Variables are immutable.

| $\lambda$ | JS    |
| --------- | ----- |
| $x$       | `x`   |
| $(a)$     | `(a)` |

### Applications

Applying a function to its arguments in λ-calc is just juxtaposition, a space between. In λ-calc all functions are unary (/ˈyo͞onərē/), which means they have a single argument. So in the case of applying multiple arguments, the functions are curried.

| $\lambda$ | JS        |
| --------- | --------- |
| $f~a$     | `f(a)`    |
| $f~a~b$   | `f(a)(b)` |

### Grouping

Parens can be used to disambiguate parts of the function.

| $\lambda$ | JS          |
| --------- | ----------- |
| $(f~a)~b$ | `(f(a))(b)` |

But, in this case, since function application is **left-associative** (meaning it's left to right), they are unneeded in this case. But parens can be used to force evaluation in a different order.

| $\lambda$ | JS        |
| --------- | --------- |
| $f~(a~b)$ | `f(a(b))` |

## Abstraction Breakdown

$I := \lambda x.x$ (`const identity = x => x`)

$\lambda~\rightarrow$  start of abstraction, function signifier, a function

$x~\rightarrow$ argument, bound to the name $x$ (parameter variable)

$.~\rightarrow$ separates the abstraction head from the body (the "=>" in a JS lambda)

$x~\rightarrow$ the return expression

$\lambda x~\rightarrow$ function head (`const identity = x`)

$x~\rightarrow$ function body (`x`)

$\lambda x.x~\rightarrow$ take an argument, bound the name $x$ and then return it

$\lambda x.x$ is equivalent to, the more common, algebraic version, $f(x) = x$

The $x$ in this case can be any letter $\{a, b, c ... x, y, z\}$ , so $\lambda x.x$ is the same as $\lambda y.y$

This is called **alpha equivalence**, as Wikipedia defines it

> A basic form of equivalence, definable on lambda terms, is alpha equivalence. It captures the intuition that the particular choice of a bound variable, in a lambda abstraction, does not (usually) matter.

## $\beta$-Reduction

As the Haskell Wiki defines it

> A *beta reduction* (also written $\beta$ reduction) is the process of calculating a result from the application of a function to an expression.

$((\lambda a.a)\lambda b.\lambda c.b)(x)\lambda e.f$

The function $(\lambda a.a)$ is applied to $\lambda b.\lambda c.b$ , so this replaces every $a$ in the function

$(\lambda b.\lambda c.b)(x)\lambda e.f$

$(\lambda b.\lambda c.b)$ is applied to the $(x)$, so we replace all of the $b$ 's in the body

$(\lambda c.x)\lambda e.f$

$(\lambda c.x)$ is applied to the $\lambda e.f$ and replace all of the $c$ in the body (there are none)

$x$ is the $\beta$-normal form

## Bound and Free Variables

* **Bound variables** are variables used in an abstraction which appear in the head of the abstraction. (parameters)
* **Free variables** are variables used in an abstraction which do not appear in the head, their value is taken to be unknown within the context of the abstraction. (global variables)

###

### Links While Editing

https://www.willtaylor.blog/an-introduction-to-lambda-calculus-explained-through-javascript/

https://lucasfcosta.com/2018/07/29/An-Introduction-to-Lambda-Calculus-Part-1.html

[https://en.wikipedia.org/wiki/Lambda_calculus#Beta_reduction](https://en.wikipedia.org/wiki/Lambda_calculus#Beta_reduction)

https://prl.ccs.neu.edu/blog/2016/11/02/beta-reduction-part-1/

https://youtu.be/3VQ382QG-y4?list=PL5ef3WuGmIuHMCH3JZLSmbV4FD6CdK1Te&t=863

[https://tadeuzagallo.com/blog/writing-a-lambda-calculus-interpreter-in-javascript/](https://tadeuzagallo.com/blog/writing-a-lambda-calculus-interpreter-in-javascript/)

[https://gist.github.com/Avaq/1f0636ec5c8d6aed2e45](https://gist.github.com/Avaq/1f0636ec5c8d6aed2e45)

[https://hackage.haskell.org/package/data-aviary-0.4.0/docs/Data-Aviary-Birds.html](https://hackage.haskell.org/package/data-aviary-0.4.0/docs/Data-Aviary-Birds.html)

[https://github.com/fantasyland/fantasy-birds](https://github.com/fantasyland/fantasy-birds)

[https://github.com/fantasyland/fantasy-combinators/tree/master/src](https://github.com/fantasyland/fantasy-combinators/tree/master/src)

[https://github.com/fantasyland/fantasy-birds/tree/master/src](https://github.com/fantasyland/fantasy-birds/tree/master/src)