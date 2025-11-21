---
tags:
  - math
  - algebra
  - comp-sci
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Currying
---
Currying is the technique of translating a function that takes multiple arguments into a sequence of families of functions, each taking a single argument.

In the prototypical example, one begins with a function $f: (X \times Y) \to Z$ that takes two arguments, one from $X$ and one from $Y$, and produces objects in $Z$. The curried form of this function treats the first argument as a parameter, so as to create a family of functions $f_x: Y \to Z$. The family is arranged so that for each object $x$ in $X$, there is exactly one function $f_x$.

In the example, curry itself becomes a function that takes $f$ as an argument and returns a function that maps each $x$ to $f_x$. The function $f$ belongs to the set of functions $(X \times Y) \to Z$. Meanwhile, $f_x$ belongs to the set of functions $Y \to Z$. Thus, something that maps $x$ to $f_x$ will be of the type $X \to [Y \to Z]$. With this notation, $\text{curry}$ is a function that takes objects from the first set and returns objects in the second set.

$\text{curry} : [(X \times Y) \to Z] \to (X \to [Y \to Z])$

```ts
curry: <X, Y, Z>(fn: (x: X, y: Y) => Z) => (x: X) => (y: Y) => Z
```

Currying is related to, but not the same as, partial application. The example above can be used to illustrate partial application; it is quite similar. Partial application is the function $\text{apply}$ that takes the pair $f$ and $x$ together as arguments and returns $f_x$. Using the same notation as above, partial application has the signature:

$\text{apply} : ([(X \times Y) \to Z] \times X) \to [Y \to Z]$

```ts
apply: <X, Y, Z>(fn: (x: X, y: Y) => Z, x: X) => (y: Y) => Z
```

Currying is useful in both practical and theoretical settings. In functional programming languages, it provides a way of automatically managing how arguments are passed to functions and exceptions. In theoretical computer science, it provides a way to study functions with multiple arguments in simpler theoretical models which provide only one argument.

Uncurrying is the dual transformation to currying and can be seen as a form of defunctionalization. It takes a function $f$ whose return value is another function $g$ and yields a new function $f^\prime$ that takes as parameters the arguments for both $f$ and $g$ and returns, as a result, the application of $f$ and subsequently, $g$, to those arguments.

## Definition

First, there is some notation to be established. The notation $X \to Y$ denotes all functions from $X$ to $Y$. If $f$ is such a function, we write $f : X \to Y$. Let $X \times Y$ denote the ordered pairs of the elements of $X$ and $Y$ respectively, that is, the Cartesian product of $X$ and $Y$. Here, $X$ and $Y$ may be sets, or they may be types, or they may be other kinds of objects.

Given a function:

$f : (X \times Y) \to Z$

currying constructs a new function

$g : X \to (Y \to Z)$

That is, $g$ takes an argument of type $X$ and returns a function of type $Y \to Z$. It is defined by

$g(x)(y) = f(x,y)$

for $x$ of type $X$ and $y$ of type $Y$. We then also write

$\text{curry}(f) = g$

#### Lambda Calculi

In theoretical computer science, currying provides a way to study functions with multiple arguments in a very simple theoretical models, such as the lambda calculus, in which functions only take a single argument. Consider a function $f(x,y)$ taking two arguments and having the type $(X \times Y) \to Z$, which should be understood to mean that $x$ must have the type $X$, $y$ must have the type $Y$ and the function itself returns the type $Z$. The curried form of $f$ if defined as 

$\text{curry}(f) = \lambda x.(\lambda y. (f(x,y)))$

Where $\lambda$ is the abstractor of lambda calculus. Since curry takes, as input, functions with the type $(X \times Y) \to Z$, one concludes that the type of curry itself is

$\text{curry} : ((X \times Y) \to Z) \to (X \to (Y \to Z))$

The $\to$ operator is often considered right-associative, so the curried function type $X \to (Y \to Z)$ is often written as $X \to Y \to Z$. Conversely, function application is considered to be left-associative, so that $f(x,y)$ is equivalent to

$((\text{curry}(f)x)y) = \text{curry}(f)xy$

That is, the parenthesis are not required to disambiguate the order of the application.

#### Type Theory

In type theory, the general idea of a type system in computer science is formalized into a specific algebra of types. For example, when writing $f : X \to Y$, the intent is that $X$ and $Y$ are types, while the arrow $\to$ is a type constructor, specifically, the function type or arrow type. Similarly, the Cartesian product $X \times Y$ of types is constructed by the product type constructor $\times$.

The type-theoretical approach provides a natural complement to the language of category theory. This is because categories, and specifically, monoidal categories, have an internal language, with simply-typed lambda calculus being the most prominent example of such a language. It is important in this context, because it can be built from a single type constructor, the arrow type. Currying then endows the language with a natural product type. The correspondence between objects in categories and types then allows programming languages to be re-interpreted as logics, and as other types of mathematical systems.

#### Logic

Under the Curry-Howard correspondence, the existence of currying and uncurrying is equivalent to the logical theorem

$((A \wedge B) \to C) \Leftrightarrow (A \to (B \to C))$

(also known as exportation), as tuples (product type) corresponds to conjunction in logic, and function type corresponds to implication.

The exponential object $Q^P$ in the category of Heyting algebras is normally written as material implication $P \to Q$. Distributive Heyting algebras are Boolean algebras, and the exponential object has the explicit form $\neg P \vee Q$, thus making it clear that the exponential object really is material implication.

## Contrast with partial function application

Currying and partial function application are often conflated. One of the significant differences between the two is that a call to a partially applied function returns the result right away, not another function down the currying chain; this distinction can be illustrated clearly for function whose arity is greater than two.

Given a function of type $f : (X \times Y \times Z) \to N$ currying produces $\text{curry}(f) : X \to (Y \to (Z \to N))$. Evaluation of the first function might be represented as $f(1,2,3)$, evaluation of the curried function would be $f_\text{curried}(1)(2)(3)$, applying each argument in turn to a single-argument function returned by the previous invocation.

Partial function application refers to the process of fixing  a number of arguments to a function, producing another function of smaller arity. Given the definition of $f$ above, we might fix (or "bind") the first argument, production a function of type $\text{partial}(f) : (Y \times Z) \to N$. Evaluation of this function might be represented as $f_\text{partial}(2,3)$.

Intuitively, partial function application says "if you fix the first argument of the function, you get a function of the remaining arguments."

The practical motivation for partial application is that very often the functions obtained by supplying some but not all of the arguments to a function are useful. Partial application makes it easy to define these functions.

Partial application can be seen as evaluating a curried function at a fixed point, e.g. given $f : (X \times Y \times Z) \to N$ and $a \in X$ then $\text{curry}(\text{partial}(f)_a)(y)(z) = \text{curry}(f)(a)(y)(z)$ or simply $\text{partial}(f)_a = \text{curry}_1(f)(a)$ where $\text{curry}_1$ curries f's first parameter. Thus, partial application is reduced to a curried function at a fixed point. Further, a curried function at a fixed point is (trivially), a partial application.