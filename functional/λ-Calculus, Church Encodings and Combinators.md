---
tags:
  - notes
  - "#functional"
  - lambda
  - combinators
---
> [!CAUTION]
> NEED A INTRO TO LAMBDA CALC.


## Calculus

  First, lets define "calculus"

  > A particular method or system of calculation or reasoning.

  What you're probably used to hearing refered to as just "calculus" is actually refering to differential calculus and integral calculus (or as the group infinitesimal calculus). This is your intro to derivatives, integrals, etc.


## $\lambda$-Calculus Syntax

![](lambda-syntax-dark.png)


### Use

  ```
  Identifier    expression ::= variable
  Application                | expression expression
  Abstraction                | λ variable.expression
  Grouping                   | (expression)
  ```


### Variables

  | $\lambda$ | JS    |
  | --------- | ----- |
  | $x$       | `x`   |
  | $(a)$     | `(a)` |


### Application

  Applying a function to its arguments in λ-calculus is just juxtaposition, a space between. In λ-calculus all functions are unary (/ˈyo͞onərē/), which means they have a single argument. So in the case of applying multiple arguments, the functions are curried.

  | $\lambda$     | JS        |
  | ------------- | --------- |
  | $f~a$         | `f(a)`    |
  | $f ~ a ~ b$   | `f(a)(b)` |


### Grouping

  Parens can be used to disambiguate parts of the function.

  | $\lambda$ | JS          |
  | --------- | ----------- |
  | $(f~a)~b$ | `(f(a))(b)` |

  Since function application is **left-associative** (meaning it's left to right), they are unneeded in this case. But parens can be used to force evaluation in a different order.

  | $\lambda$ | JS        |
  | --------- | --------- |
  | $f~(a~b)$ | `f(a(b))` |


##### Syntax Shorthand

  $\lambda a.\lambda b.\lambda c.b$ can be simplified, visually, by just removing the internal $\lambda$ notation as it is curried by default.

  $\lambda a.\lambda b.\lambda c.b = \lambda abc.b$


### Abstraction Breakdown

  $\quad I := \lambda x.x$ (`const identity = x => x`)

  $\quad \lambda~\rightarrow$  start of abstraction, function signifier, a function

  $\quad x~\rightarrow$ argument

  $\quad .~\rightarrow$ separates the abstraction head from the body (the "=>" in a JS lambda)

  $\quad x~\rightarrow$ the return expression

  $\quad \lambda x~\rightarrow$ function head (`const identity = x`)

  $\quad x~\rightarrow$ function body (`x`)

  $\quad \lambda x.x~\rightarrow$ take an argument, $x$, and then return it

  $\quad \lambda x.x$ is equivalent to, the more common, algebraic version, $f(x) = x$

  The $x$ in this case can be any letter $\{a, b, c ... x, y, z\}$ , so $\lambda x.x$ is the same as $\lambda y.y$

  This is called **alpha equivalence**, as Wikipedia defines it

  > A basic form of equivalence, definable on lambda terms, is alpha equivalence. It captures the intuition that the particular choice of a bound variable, in a lambda abstraction, does not (usually) matter.

  > [!CAUTION]
  > Need an image for $\alpha$-equivalence


### Cross Comparisons

  | Type          | $\lambda$               | JS            | Notes                                |
  |---------------|-------------------------|---------------|--------------------------------------|
  | Variables¹    | $x$                     | `x`           |                                      |
  | Variables²    | $(x)$                   | `(x)`         |                                      |
  | Application¹  | $f\:a$                  | `f(a)`        |                                      |
  | Application²  | $f\:a\:b$               | `f(a)(b)`     | Curried                              |
  | Application³  | $(f\:a)\:b$             | `(f(a))(b)`   | Uneeded. Left Associative            |
  | Application⁴  | $f\:(a\:b)$             | `f((a)(b))`   | Force different order                |
  | Abstractions¹ | $\lambda a.b$           | `a => b`      |                                      |
  | Abstractions² | $\lambda a.b\:x$        | `a => b(x)`   | Is greedy. Swallows all to the right |
  | Abstractions³ | $\lambda a.(b\:x)$      | `a => (b(x))` | Disambiguate. But not needed         |
  | Abstractions⁴ | $(\lambda a.b)\:x$      | `(a => b)(x)` | Apply `a => b` to `x`                |
  | Abstractions⁵ | $\lambda a.\lambda b.a$ | `a => b => a` | Nested functions.                    |


## $\beta$-Reduction

  > A *beta reduction* (also written $\beta$ reduction) is the process of calculating a result from the application of a function to an expression.

  $((\lambda a.a)\lambda b.\lambda c.b)(x)\lambda e.f$

  The function $(\lambda a.a)$ is applied to $\lambda b.\lambda c.b$ , so this replaces every $a$ in the function

  $(\lambda b.\lambda c.b)(x)\lambda e.f$

  $(\lambda b.\lambda c.b)$ is applied to the $(x)$, so we replace all of the $b$ 's in the body

  $(\lambda c.x)\lambda e.f$

  $(\lambda c.x)$ is applied to the $\lambda e.f$ and replace all of the $c$ in the body (there are none)

  $x$ is the $\beta$-normal form

![](lambda-beta-dark.png)


## Combinatory Logic

  Combinatory logic is a theoretical framework in mathematical logic and the study of lambda calculus. It deals with expressing computations using function combinators, which are abstract, pure functions with no free variables. Combinatory logic was developed by Moses Schönfinkel and Haskell Curry in the early 20th century and is a fundamental part of the foundations of functional programming.

  Combinators are pure functions that take other functions as arguments and return new functions. They are used to combine and manipulate functions. Combinators have no free variables. A free variable is some variable in a function body that is not bound to some parameter. In other words, external data or context. 

  Not all parameters HAVE to be used. Both $\lambda b.b$ and $\lambda ab.a$ are combinators. While $\lambda b.a$ is not a combinator. $a$ comes from no where.

  One of the primary operations in combinatory logic is function application, which is achieved by applying one function (combinator) to another function or argument.

  Combinatory logic uses lambda abstraction, represented by the Greek letter $λ$ (lambda), to define functions. Lambda abstraction allows you to create anonymous functions with specified arguments and a body.

  Some combinators are designed to find fixed points of functions, which are inputs that, when applied to the function, result in the same output as the input. The most famous fixed-point combinator is the $Y$ combinator.

  In combinatory logic, the goal is to eliminate variables and represent all operations solely through function combinators. This can lead to concise and purely functional representations of complex computations.


### Bound and Free Variables

  - **Bound variables** are variables used in an abstraction which appear in the head of the abstraction. (parameters)
  - **Free variables** are variables used in an abstraction which do not appear in the head, their value is taken to be unknown within the context of the abstraction. (global variables)


## $\color{#fa8072}\lambda a.a$ Identity

  The $I$ combinator, also known as the "identity combinator," is a function that takes an argument $a$ and simply returns $a$ itself. It's called the "identity combinator" because it leaves the argument unaltered.

  The $I$ combinator is very simple but plays a crucial role in combinatory logic as it allows you to represent and manipulate functions in a way that doesn't rely on variables or specific values. It's used as a foundational component in creating more complex combinators and expressing computations in this formal system.

  #### Lambda:

  $\quad I := \lambda a.a$

  #### JS:

  ```js
  const I = a => a;
  ```

  #### Haskell (built in):

  ```haskell
  id 5 == 5
  ```

  Identity of identity is identity.

  $\quad I I = I$

  ```js
  I(I) === I;
```


## $\color{#fa8072}\lambda f.ff$ Mockingbird

  The $M$ combinator takes a function $f$ and applies it to itself. It's sometimes called the "self-application combinator" because it applies the function to itself.

  The $M$ combinator has interesting and powerful properties. It can be used to represent recursion and fixed-point operators in combinatory logic. By applying a function to itself, it allows for self-reference, enabling the definition of functions that refer to themselves in a recursive manner. This makes the $M$ combinator a fundamental building block in expressing various computations and recursive functions in combinatory logic

  #### Lambda:

  $\quad M := \lambda f.ff$

  #### JS:

  ```js
  const M = f => f(f);
  ```

  It takes a function as input and invokes that function passing in itself. Self-application combinator.

  #### Examples:

  ```js
  M(I);
  //> Function I

  M(M);
  //> Infinite call. Callstack exceeded error.
  // Halting problem.
  ```


## $\color{#fa8072}\lambda ab.a$ Kestrel

  The $K$ combinator, also known as the "constant combinator," is a function that takes two arguments, $a$ and $b$, and simply returns the first argument $(a)$ while ignoring the second argument $(b)$. It's called the "constant combinator" because it always returns the same value $(a)$, regardless of the value of the second argument $(b)$.

  #### Lambda:

  $\quad K := \lambda ab.a$

  $\quad\quad\enspace\color{gray}= \lambda a.\lambda b.a$

  #### JS:

  ```js
  const K = a => b => a;
  ```

  #### Haskell (built in):

  ```haskell
  const 7 2 == 7
  ```

  It takes an `a` and a `b` and then returns the `a`. A function that is fixated on a particular value. This is the Constant combinator.

  #### Examples:

  ```js
  K(I)(M);
  //> Function I

  K(K)(M);
  //> Function K

  const K5 = K(5);

  K5(3);
  //> 5

  K5(9);
  //> 5

  K(I)(x)(y) === y
  //> true

  // K(I)(x) === I so K(I)(x)(y) == I(y) == y
  // So K(I) takes two values and returns the second (Opposite of K).
  // This is a derivation of the next (KI)...
  ```


## $\color{#fa8072}\lambda ab.b$ Kite

  The $KI$ combinator, like the $K$ combinator, is a function that takes two arguments, $a$ and $b$. However, in contrast to the $K$ combinator, which always returns the first argument $(a)$ and ignores the second argument, the $Ki$ combinator always returns the second argument $(b)$ and ignores the first argument $(a)$. Therefore, it's sometimes called the "constant combinator for the second argument."

  In combinatory logic, both $K$ and $KI$ combinators serve as basic building blocks, providing the ability to represent and manipulate functions without relying on variable names or specific values.

  #### Lambda:

  $\quad KI := \lambda ab.b$

  $\quad\quad\quad\color{gray}= \lambda a.\lambda b.b$

  $\quad\quad\quad\color{gray}= K I$

  #### JS:

  ```js
  const KI = a => b => b;
  // const KI = K(I);
  ```

  #### Examples:

  ```js
  KI(4)(9);
  //> 9
  ```


## $\color{#fa8072}\lambda fab.fba$ Cardinal

  Takes a function and two arguments and calls the function with the arguments reversed.

  #### Lambda:

  $\quad C := \lambda fab.fba$

  $\quad\quad\quad\color{gray}= \lambda f.\lambda a.\lambda b.fba$

  #### JS:

  ```js
  const C = f => a => b => f(b)(a);
  ```

  #### Haskell (built in):

  ```haskell
  flip const 1 8 == 8
  ```

  #### Examples:

  ```js
  C(K)(I)(M)
  //> Function M

  // C(K) == KI. Takes two things and returns the second.

  C(K)(1)(2);
  //> 2

  KI(1)(2);
  //> 2

  K(I)(1)(2);
  //> 2
  ```


## Booleans

  ![](lambda-boolean-dark.png)

  Its function application. If its "true" then it selects the first expression. If its "false" it selects the second expression.

  Functions that select either the first or the second expression already exist!

  #### Lambda:

  $\quad TRUE := K$

  $\quad FALSE := KI$ (or $CK$)

  #### JS:

  ```js
  const T = K;
  const F = KI;
  ```

  We can add a little helper to make this easier to read (in JS).

  ```js
  const util = require('node:util');
  T[util.inspect.custom] = () => `True (K)`;
  F[util.inspect.custom] = () => `False (KI)`;
  ```


## Negation

  $!p$ isn't part of the LC syntax. So it needs to be a function. $NOT$.

  > [!CAUTION]
  > Need excalidraw image.

  So this is selecting between two possibilities. We already have that defined. The defined "booleans" themselves already negate themselves.

  #### Lambda:

  $\quad\textcolor{gray}{K}\quad TFT$

  #### JS:

  ```js
  // K
  const T (F)(T);
  ```

  Kestrel ($K$) will always select the first. In this case, $F$.

  #### Lambda:

  $\quad\textcolor{gray}{KI}\quad FFT$

  ```js
  // KI
  const F (F)(T);
  ```

  Meanwhile, the Kite ($KI$) will always select the second.

  So $NOT$ is simple, apply the given boolean to $(F)(T)$.

  #### Lambda:

  $\quad NOT := \lambda p.pFT$

  #### JS:

  ```js
  const NOT = p => p(F)(T);
  ```

  #### Examples:

  ```js
  Not(T)
  //> False (KI)

  Not(F)
  //> True (K)
  ```


## Conjunction (AND)

  So we need a function that takes two arguments. If the first argument is `F` then we can short circuit and return `F`. If it is `T` then we need to check the second argument. Since both `T` and `F` are themselves functions that take two arguments, `T` returning the first and `F` returning the second.

  #### Lambda:

  $\quad AND := \lambda pq.pqp$

  #### JS:

  ```js
  // If p is F, then p. Else whatever q is.
  const AND = p => q => p(q)(p);
  ```

  So if `p` is `F`, it will just return the second argument which should be `F`. But since we just determined that `p` is `F` we can just use itself. If `p` is `T` it will be whatever `q` is. `T` or `F` since they both have to be `T` for the expression to be `T`.

  #### Examples:

  ```js
  AND(T)(T)
  //> True (K)

  AND(T)(F)
  //> False (KI)

  AND(F)(T)
  //> False (KI)

  AND(F)(F)
  //> False (KI)
  ```


## Disjunction (OR)

  Like `AND`, its a function that takes two arguments. If the first argument is `T` then we can short circuit in the `T` direction this time. And like `AND` we can let it determine itself from the `F` path. And, again, since we know `p` is `T` in the first place, we can just use itself.


  #### Lambda:

  $\quad OR := \lambda pq.ppq$

  #### JS:

  ```js
  // If p is T then T. Else, whatever q is.
  const OR = p => q => p(p)(q);
  ```

  #### Examples:

  ```js
  OR(T)(F)
  //> True (K)

  OR(F)(T)
  //> True (K)

  OR(F)(F)
  //> False (KI)
  ```


## Exclusive Or (XOR)

  #### Lambda:

  $\quad XOR := \lambda pq.p(NOT\enspace q)\enspace q$

  #### JS:
  ```js
  const XOR = p => q => p(NOT(q))(q);
  ```

  #### Examples:

  ```js
  XOR(T)(T)
  //> False (KI)

  XOR(F)(F)
  //> False (KI)

  XOR(T)(F)
  //> True (K)

  XOR(F)(T)
  //> True (K)
  ```


## Equality

  #### Lambda:

  $\quad BEQ := \lambda pq.pq(NOT\enspace q) = NOT\enspace XOR$

  #### JS:

  ```js
  const BEQ = p => q => p(q)(NOT(q)) == NOT(XOR);
  ```

  #### Examples:

  ```js
  BEQ(T)(F)
  //> False (KI)

  BEQ(T)(T)
  //> True (K)

  BEQ(F)(F)
  //> True (K)
  ```


## De Morgan's Laws

  > The negation of a disjunction is the conjunction of the negations.

  $\quad\neg(P\lor Q) \iff (\neg P)\land (\neg Q)$

  > The negation of a conjunction is the disjunction of the negations.

  $\quad\neg(P\land Q) \iff (\neg P)\lor (\neg Q)$

  Where:

  - $P$ and $Q$ are propositions
  - $\neg$ is the negation logic operator (NOT)
  - $\land$ is the conjunction logic operator (AND)
  - $\lor$ is the disjunction logic operator (OR)
  - $\iff$ is a metalogical symbol meaning "if and only if"

  #### Lambda:

  $\quad Fst = BEQ(NOT(OR\enspace pq))(AND(NOT\enspace p)(NOT\enspace q))$

  $\quad Snd = BEQ(NOT(AND\enspace pq))(OR(NOT\enspace p)(NOT\enspace q))$

  #### JS:

  ```js
  const Lit_Fst = !(p || q) === (!p) && (!q);

  const Lit_Snd = !(p && q) === (!p) || (!q);

  const Fst = p => q => BEQ(NOT(OR(p)(q)))(AND(NOT(p))(NOT(q)));

  const Snd = p => q => BEQ(NOT(AND(p)(q)))(OR(NOT(p))(NOT(q)));
  ```

  #### Examples:

  ```js
  Fst(T)(T)
  //> True (K)

  Fst(T)(F)
  //> True (K)

  Fst(F)(F)
  //> True (K)

  Snd(T)(T)
  //> True (K)

  Snd(T)(F)
  //> True (K)

  Snd(F)(F)
  //> True (K)
  ```

## Church Numerals

### Zero

  #### Lambda:

  $\quad N0 := \lambda fa.a$

  #### JS:

  ```js
  const N0 = f => a => a;
  ```

  Which is saying, take two things and return the second. Which is the same as $F$ or $KI$. So zero is false.

  #### Examples:

  ```js
  const N0(NOT)(T)
  //> True (K)

  const F(NOT)(T)
  //> True (K)
  ```


### One

  #### Lambda:

  $\quad N1 := \lambda fa.fa$

  #### JS:

  ```js
  const N1 = f => a => f(a);
  ```

  So we are applying a function to a thing. This is also familiar. This is Identity once removed. So 0 is false and 1 is identity.

  #### Examples:

  ```js
  const N1(NOT)(T) // = NOT(T)
  //> False (KI)
  ```


### Two

  #### Lambda:

  $\quad N2 := \lambda fa.f(fa)$

  #### JS:

  ```js
  const N2 = f => a => f(f(a));
  ```

  We are applying a function to the application of a function to a thing.

  #### Examples:

  ```js
  const N2(NOT)(T) // = NOT(NOT(T))
  //> True (K)
```


### Three

  #### Lambda:

  $\quad N3 := \lambda fa.f(f(fa))$

  #### JS:

  ```js
  const N3 = f => a => f(f(f(a)));
  ```

  We are applying a function to the application of an application of a function to a thing.

  #### Examples:

  ```js
  const N3(NOT)(T) // = NOT(NOT(NOT(T)))
  //> False (KI)
  ```

We are just building up function applications on a thing. The announce of having to spell out the fold constantly should be apparent. We need something to do this for us.


## Successor

  We want to be able to do

  $\quad SUCC\space N1 = N2$

  $\quad SUCC\space N2 = N3$

  $\quad SUCC\space (SUCC\space N1) = N3$

  This is actually Peano numbers.

  > Peano numbers are a simple way of representing the natural numbers using only a zero value and a successor function.

  #### Lambda:

  $\quad SUCC := \lambda nfa.f(nfa)$

  #### JS:

  ```js
  const SUCC = n => f => a => f(n(f)(a));
  ```

  #### Examples:

  ```js
  SUCC(N0)
  //> [Function]
  ```

  Church numerals are not intentionally equals, its only extensional equal. So we need to prove that this is the $N1$ function.

  ```js
  SUCC(N0)(NOT)(T)
  //> False (KI)
  // Ergo, N1
  ```

  _[Extensionality, or extensional equality](https://en.wikipedia.org/wiki/Extensionality), refers to principles that judge objects to be equal if they have the same external properties._
  _Intensionality, or intensional equality, is concerned with whether the internal definitions of objects are the same._

  This can get a little annoying to always prove, so we can make a helper to make it easier (in JS).

  ```js
  const toJSNum = n => n(x => x + 1)(0);

  toJSNum(SUCC(N0));
  //> 1
  ```

  So now we can count, and set, our numbers a little easier by just using the `SUCC` of the previous.

  ```js
  const N0 = F;
  const N1 = SUCC(N0);
  const N2 = SUCC(N1);
  const N3 = SUCC(N2);
  const N4 = SUCC(N3);
  const N5 = SUCC(N4);
  ```


## $\color{#fa8072}\lambda fga.f(ga)$ Bluebird

  The $B$ combinator is used to combine and apply two functions to an argument.

  It essentially represents function composition, where $g$ is applied to $a$ first, and then the result is passed to $f$.

  The $B$ combinator is essential in combinatory logic as it allows you to express more complex function composition and application.

  #### Lambda:

  $\quad B := \lambda fga.f(ga)$

  $\quad\quad\enspace\color{gray}= \lambda f.\lambda g.\lambda a.f(ga)$

  #### JS:

  ```js
  const B = f => g => a => f(g(a));
  ```

  This is just simple function composition.

  #### Examples:

  ```js
  B(NOT)(NOT)(T)
  //> True (K)
  ```


## Successor Revisited

  Now that we have the power of the B combinator, we can simplify. Since we are just feeding function results in other functions:

  > [!CAUTION]
  > Need excalidraw image.

  We can just use function composition.

  #### Lambda:

  $\quad SUCC := \lambda nf.Bf(nf)$

  ```js
  const SUCC = n => f => B(f)(n(f));
  ```

  #### Examples:

  ```js
  toJSNum(SUCC(N4))
  //> 5

  toJSNum(SUCC2(SUCC2(N4)))
  //> 6
  ```


## Binary Addition

  So if we say 

  $\quad ADD\space N3\space N5 = N8$ 

  $\quad ADD\space N3\space N5 = SUCC\space (SUCC\space (SUCC\space N5))$ 

  then it is easy to see that we are doing function composition. And Church Numerals are functions that create $n$-fold composition. So we can write it as 

  $\quad ADD\space N3\space N5 = (SUCC \circ SUCC \circ SUCC)\space N5$

  Since it is the 3-fold composition of $SUCC$ we can generate that by simply using the Church Numerals.

  $\quad ADD\space N3\space N5 = N3\space SUCC\space N5$

  #### Lambda:

  $\quad ADD := \lambda nk.n\space SUCC\space k$

  #### JS:

  ```js
  const ADD = n => k => n(SUCC)(k);
  ```

  #### Examples:

  ```js
  toJSNum(ADD(N3)(N4))
  //> 7

  toJSNum(ADD(N1)(N4))
  //> 5
  ```


## Multiplication

  $\quad MULT\space N2\space N3 = N6$

  We know that $N6$ is a six-fold composition

  $\quad MULT\space N2\space N3\space f\space a = (f \circ f \circ f \circ f \circ f \circ f)\space a$ 

  Since multiplication is associative 

  $\quad MULT\space N2\space N3\space f\space a = ((f \circ f \circ f) \circ (f \circ f \circ f))\space a$ 

  We can group them. Theres our three-fold ($N3$)

  $\quad MULT\space N2\space N3\space f\space a = ((N3 \space f) \circ (N3 \space f))\space a$ 

  And since we are doing the two-fold composition on $N3$

  $\quad MULT\space N2\space N3\space f\space a = N2\space (N3 \space f)\space a$ 

  Cancel the $a$

  $\quad MULT\space N2\space N3\space f = N2\space (N3 \space f)$ 

  $\lambda nkf.n(kf)$ would suffice, but we can take it further.

  $\quad MULT\space N2\space N3\space f = N2\space (N3 \space f)$ is just composition

  $\quad MULT\space N2\space N3\space f = (N2 \circ N3) \space f$

  Now we can cancel out the $f$

  $\quad MULT\space N2\space N3 = (N2 \circ N3)$

  So multiplying two number is just getting their composition.

  $\quad MULT\space N2\space N3 = B\space N2 \space N3$

  But now we can cancel the numerals out

  $\quad MULT = B$

  So multiplication is just the $B$ combinator.

  The two definitions are $\alpha$-equivalent. Which means the only difference is the variable names.

  $\quad MULT := \lambda \textcolor{#fa8072}{n}k\textcolor{pink}{f}.\textcolor{#fa8072}{n}(k\textcolor{pink}{f})$

  $\quad\quad\quad\space B := \lambda \textcolor{#fa8072}{f}g\textcolor{pink}{a}.\textcolor{#fa8072}{n}(g\textcolor{pink}{a})$

  #### Lambda:

  $\quad MULT := B$

  $\quad\quad\quad\quad\quad\color{gray}= \lambda nkf.n(kf)$

  #### JS:

  ```js
  const MULT = B;
  ```

  #### Examples:

  ```js
  toJSNum(MULT(N3)(N4))
  //> 12

  toJSNum(MULT(N2)(N3))
  //> 6
  ```


## Exponentiation

  $\quad POW\space N2\space N3 = N8$

  $\quad POW\space N2\space N3 = N2 \circ N2 \circ N2$

  Pattern should be getting more clear.

  $\quad POW\space N2\space N3 = N3\space N2$

  Just takes the numerals and flips them around. Which is itself a common combinator. Called the Thrush combinator (Th).

  #### Lambda:

  $\quad POW := Th$

  $\quad\quad\quad\quad\color{gray}= \lambda nk.kn$

  #### JS:

  ```js
  // const Th = a => f => f(a);

  const POW = Th;
  ```

  #### Examples:

  ```js
  toJSNum(POW(N2)(N3))
  //> 8

  toJSNum(POW(N3)(N3))
  //> 27
  ```


## Is Zero

  #### Lambda:

  $\quad Is0 := \lambda n.n\space (KF)\space T$

  #### JS:

  ```js
  const Is0 = n => n(K(F))(T);
  ```

  Numerals are $n$-folds, but since its folding a constan false $K(F)$ it could fold a billion times and it doesn't matter, it will still be False. Only if the constant false isn't called will it be true. Which is the case when given the $N0$ numeral.

  #### Examples:

  ```js
  Is0(N1)
  //> False (KI)

  Is0(N0)
  //> True (K)

  Is0(N9)
  //> False (KI)
  ```


## $\color{#fa8072}\lambda abf.fab$ Vireo

  You give this function two arguments. Then you can move this box around and pass it where you want. When you are ready to use te arguments, you give it a function to apply to the arguments. This is the smallest data structure. A closure. Can also be called $PAIR$ in the context of Church Pairs.

  #### Lambda:

  $\quad V := \lambda abf.fab$

  #### JS:

  ```js
  const V = a => b => f => f(a)(b);
  ```

  #### Examples:

  ```js
  V(I)(M)(K) // First
  //> Function I

  V(I)(M)(KI) // Second
  //> Function M
  ```


## $FST$ and $SND$

  #### Lambda:

  $\quad FST := \lambda p.pK$

  $\quad SND := \lambda p.p(KI)$

  #### JS:

  ```js
  const FST = p => p(K);

  const SND = p => p(KI);
  ```

  #### Examples:

  ```js
  FST(PAIR(I)(M))
  //> Function I

  SND(PAIR(I)(M))
  //> Function M
  ```


## (Big) $\Phi$

  #### Lambda:

  $\quad \Phi := \lambda p.PAIR\space (SND\space p)\space (SUCC\space (SND\space p))$

  #### JS:

  ```js
  const PHI = p => PAIR(SND(p))(SUCC(SND(p)));
  ```

  #### Examples:

  ```js
  const p0 = PAIR(N0)(N0);
  const p1 = PHI(p0);

  toJSNum(FST(p1))
  //> 0

  toJSNum(SND(p1))
  //> 1

  const p2 = PHI(p1)

  toJSNum(FST(p2))
  //> 1

  toJSNum(SND(p2))
  //> 2
  ```

  Going forward, $PAIR(x)(y)$ will be denoted $(x, y)$ for brevity.

  $\Phi (M, N7) = (N7, N8)$

  So it takes the Second of the input pair and moves it to the First of the output. Then it takes the Second of the input pair and puts the Succ of that as the Second of the output pair.

  $\Phi (N9, N2) = (N2, N3)$

  $\Phi (N0, N0) = (N0, N1)$

  $\Phi (N0, N1) = (N1, N2)$

  $N8\space\Phi (N0, N0) = (N7, N8)$

  $FST\space (N8\space\Phi (N0, N0)) = FST\space (N7, N8) = N7$

  Eureka! We have subtraction!


## Predecessor

  #### Lambda:

  $\quad PRED := \lambda n.FST\space (n\space\Phi\space (PAIR\space N0\space N0))$


  #### JS:

  ```js
  const PRED = n => FST(n(PHI)(PAIR(N0)(N0)));
  ```

  #### Examples:

  ```js
  toJSNum(PRED(N5))
  //> 4
  ```


## Subtraction

  #### Lambda:

  $\quad SUB := \lambda nk.k\space PRED\space n$


  #### JS:

  ```js
  const SUB = n => k => k(PRED)(n);
  ```

  #### Examples:

  ```js
  toJSNum(SUB(N7)(N2))
  //> 5
  ```


## Less Than Equal To

  #### Lambda:

  $\quad LEQ := \lambda nk.Is0\space (SUB\space n\space k)$

  Just SUB $k$ from $n$ and seeing if it bottoms out.

  #### JS:

  ```js
  const LEQ = n => k => Is0(SUB(n)(k));
  ```

  #### Examples:

  ```js
  LEQ(N4)(N2)
  //> False (KI)

  LEQ(N4)(N4)
  //> True (K)

  LEQ(N2)(N4)
  //> True (K)
  ```


## Equal To

  #### Lambda:

  $\quad EQ := \lambda nk.AND(LEQ\space n\space k)(LEQ\space k\space n)$

  Just checks LEQ in both directions.

  #### JS:

  ```js
  const EQ = n => k => AND(LEQ(n)(k))(LEQ(k)(n));
  ```

  #### Examples:

  ```js
  EQ(N4)(N2)
  //> False (KI)

  EQ(N2)(N4)
  //> False (KI)

  EQ(N4)(N4)
  //> True (K)
  ```


## $\color{#fa8072}\lambda fgab.f(gab)$ Blackbird

  Binary function composition.

  #### Lambda:

  $\quad B_{1} := \lambda fgab.f(gab)$

  #### JS:

  ```js
  const B1 = f => g => a => b => f(g(a)(b));
  ```

## Greater Than

  #### Lambda:

  $\quad GT := B_{1}\space NOT\space LEQ$

  #### JS:

  ```js
  const GT = B1(NOT)(LEQ);
  ```

  #### Examples:

  ```js
  GT(N4)(N3)
  //> True (K)

  GT(N3)(N4)
  //> False (KI)
```


## References

### Combinators

| Sym. | Bird        | $\lambda$-Calculus                                                   | JS                                                  | Haskell          | Use                        |
|------|-------------|----------------------------------------------------------------------|-----------------------------------------------------|------------------|----------------------------|
| I    | Idiot       | $\lambda a.a$                                                        | `x => x`                                            | `id`             | identity                    |
| B    | Bluebird    | $\lambda fga.f(ga)$                                                  | `f => g => a => f(g(a))`                            | `(.)`            | unary composition           |
| M    | Mockingbird | $\lambda f.ff$                                                       | `f => f(f)`                                         | _not definable_  | self-application            |
| K    | Kestrel     | $\lambda ab.a$                                                       | `a => b => a`                                       | `const`          | first, const                |
| KI   | Kite        | $\lambda ab.b = KI = CK$                                             | `a => b => b`                                       | `const id`       | second                      |
| C    | Cardinal    | $\lambda fab.fba$                                                    | `f => a => b => f(b)(a)`                            | `flip`           | reverse args                |
| Th   | Thrush      | $\lambda af.fa = CI$                                                 | `a => f => f(a)`                                    | `flip id`        | hold an argument            |
| V    | Vireo       | $\lambda abf.fab = BCT$                                              | `a => b => f => f(a)(b)`                            | `flip . flip id` | hold a pair of args         |
| B1   | Blackbird   | $\lambda fgab.f(gab) = BBB$                                          | `f => g => a => b => f(g(a)(b))`                    | `(.).(.)`        | binary to unary composition |
| Y    |             | $\lambda f.(\lambda x.f(xx))(\lambda x.f(xx))$                       | `f => (x => f(x(x)))(x => f(x(x)))`                 |                  | recursion (for lazy)        |
| Z    |             | $\lambda f.(\lambda x.f(\lambda v.xxv))(\lambda x.f(\lambda v.xxv))$ | `f => (x => f(v => x(x)(v)))(x => f(v => x(x)(v)))` |                  | recursion (for strict, JS)  |


### Church Encodings: Booleans

| Name  | $\lambda$-Calculus             | Use                |
|-------|--------------------------------|--------------------|
| True  | $\lambda ab.a = K$             | Encoding for true  |
| False | $\lambda ab.b = KI$            | Encoding for false |
| Not   | $\lambda p.pFT ~= C$           | Negation           |
| And   | $\lambda pq.pqp$               | Conjunction        |
| Or    | $\lambda pq.ppq = M*$          | Disjunction        |
| Beq   | $\lambda pq.pq(NOT\enspace q)$ | Equality           |

### Church Encodings: Numerals

| Sym. | Name     | $\lambda$-Calculus          | Use                       |
|------|----------|-----------------------------|---------------------------|
| N0   | Zero     | $\lambda fa.a = F$          | apply $f$ no times to $a$ |
| N1   | Once     | $\lambda fa.fa = I*$        | apply $f$ once to $a$     |
| N2   | Twice    | $\lambda fa.f(fa)$          | apply $f$ 2-fold to $a$   |
| N3   | Thrice   | $\lambda fa.f(f(fa))$       | apply $f$ 3-fold to $a$   |
| N4   | Fourfold | $\lambda fa.f(f(f(fa)))$    | apply $f$ 4-fold to $a$   |
| N5   | Fivefold | $\lambda fa.f(f(f(f(fa))))$ | apply $f$ 5-fold to $a$   |

### Church Arithmetic

| Name | $\lambda$-Calculus                                                 | Use                           |
|------|--------------------------------------------------------------------|-------------------------------|
| Succ | $\lambda nf.Bf(nf) = \lambda nfa.f(nfa)$                           | successor of n                |
| Add  | $\lambda nk.n\space SUCC\space k$                                  | addition of $n$ and $k$       |
| Mult | $\lambda nkf.n(kf) = B$                                            | multiplication of $n$ and $k$ |
| Pow  | $\lambda nk.kn = Th$                                               | raise $n$ to the power of $k$ |
| Pred | $\lambda n.FST\space (n\space\Phi\space (PAIR\space N0\space N0))$ | predecessor of $n$            |
| Sub  | $\lambda nk.k\space PRED\space n$                                  | subtract $k$ from $n$         |

### Church Arithmetic: Boolean Operations

| Name | $\lambda$-Calculus                                         | Use                |
|------|------------------------------------------------------------|--------------------|
| Is0  | $\lambda n.n\space (KF)\space T$                           | test if $n = 0$    |
| Leq  | $\lambda nk.Is0\space (SUB\space n\space k)$               | test if $n \leq k$ |
| Eq   | $\lambda nk.AND(LEQ\space n\space k)(LEQ\space k\space n)$ | test if $n = k$    |
| Gt   | $B1\space NOT\space LEQ$                                   | test if $n \gt k$  |

### Church Pairs

| Sym.   | Name | $\lambda$-Calculus                                                    | Use                      |
|--------|------|-----------------------------------------------------------------------|--------------------------|
|        | PAIR | $\lambda abf.fab = V$                                                 | pair two arguments       |
|        | FST  | $\lambda p.pK$                                                        | extract first of a pair  |
|        | SND  | $\lambda p.p(KI)$                                                     | extract second of a pair |
| $\Phi$ | Phi  | $\lambda p.PAIR\space (SND\space p)\space (SUCC\space (SND\space p))$ | copy 2nd to 1st, inc 2nd |