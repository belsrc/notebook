---
tags:
  - functional
  - notes
---
Combinatory logic is a theoretical framework in mathematical logic and the study of lambda calculus. It deals with expressing computations using function combinators, which are abstract, pure functions with no free variables. Combinatory logic was developed by Moses Schönfinkel and Haskell Curry in the early 20th century and is a fundamental part of the foundations of functional programming.

Combinators are pure functions that take other functions as arguments and return new functions. They are used to combine and manipulate functions. Combinators have no free variables, meaning they don't rely on external data or context.

One of the primary operations in combinatory logic is function application, which is achieved by applying one function (combinator) to another function or argument.

Combinatory logic uses lambda abstraction, represented by the Greek letter $λ$ (lambda), to define functions. Lambda abstraction allows you to create anonymous functions with specified arguments and a body.

Some combinators are designed to find fixed points of functions, which are inputs that, when applied to the function, result in the same output as the input. The most famous fixed-point combinator is the $Y$ combinator.

In combinatory logic, the goal is to eliminate variables and represent all operations solely through function combinators. This can lead to concise and purely functional representations of complex computations.

The most fundamental combinators are $S$, $K$, and $I$:
  * The $S$ combinator is used for function composition and represents the application of one function to another.
  * The $K$ combinator is the constant function and takes two arguments, returning the first argument while ignoring the second.
  * The $I$ combinator, also known as the `identity` function, returns its argument as is.

> [!note]
> For info regarding Lambda Calculus check: [[Lambda Calculus]]


## A Combinator

The $A$ combinator, often referred to as the "application" or "apply" combinator, takes two functions, $f$ and $x$, and applies function $f$ to the argument $x$. It effectively represents function application.

$A = λf.λx.(f x)$

If you have a function $f$ that takes an argument $x$, you can apply $f$ to $x$ using the $A$ combinator as $A f x$. It takes the place of the usual function application operator, such as `f(x)` in many programming languages.

The $A$ combinator is a fundamental concept in combinatory logic and helps to simplify expressions and reason about functions and their applications in a purely functional way.


## B<sub>1</sub> Combinator

The $B_1$ combinator is more specialized than the general $B$ combinator and has specific use cases where you need to apply one function to the result of another function while passing a fixed argument, such as 1, to one of them.


## B Combinator

The $B$ combinator is used to combine and apply two functions to an argument.

$B = λf.λg.λx.(f (g x))$

Given two functions, $f$ and $g$, and an argument $x$, you can use the $B$ combinator as $B f g x$.

It essentially represents function composition, where $g$ is applied to $x$ first, and then the result is passed to $f$.

The $B$ combinator is essential in combinatory logic as it allows you to express more complex function composition and application.


## C Combinator

$C = λx.λy.λz.(xz)(yz)$

The $C$ combinator takes three arguments, $x$, $y$, and $z$, and applies them in the following way:
  * It applies $x$ to $z (xz)$.
  * It applies $y$ to $z (yz)$.
  * Then, it applies the result of the first step ($xz$) to the result of the second step ($yz$).

The $C$ combinator represents a function that takes three arguments and returns the result of applying the first argument to the third argument and the second argument to the third argument.

$C$ is often used in combinatory logic to demonstrate how various functions can be expressed using a minimal set of combinators.


## I Combinator

$I = λx.x$

The $I$ combinator, also known as the "identity combinator," is a function that takes an argument $x$ and simply returns $x$ itself. It's called the "identity combinator" because it leaves the argument unaltered, and it's a fundamental building block in combinatory logic.

The $I$ combinator is very simple but plays a crucial role in combinatory logic as it allows you to represent and manipulate functions in a way that doesn't rely on variables or specific values. It's used as a foundational component in creating more complex combinators and expressing computations in this formal system.


## K Combinator

$K = λx.λy.x$

The $K$ combinator, also known as the "constant combinator," is a function that takes two arguments, $x$ and $y$, and simply returns the first argument $(x)$ while ignoring the second argument $(y)$. It's called the "constant combinator" because it always returns the same value $(x)$, regardless of the value of the second argument $(y)$.


## Ki Combinator

The $Ki$ combinator is a variant of the $K$ combinator (also known as the "constant combinator") in combinatory logic.

$Ki = λx.λy.y$

The $Ki$ combinator, like the $K$ combinator, is a function that takes two arguments, $x$ and $y$. However, in contrast to the $K$ combinator, which always returns the first argument $(x)$ and ignores the second argument, the $Ki$ combinator always returns the second argument $(y)$ and ignores the first argument $(x)$. Therefore, it's sometimes called the "constant combinator for the second argument."

In combinatory logic, both $K$ and $Ki$ combinators serve as basic building blocks, providing the ability to represent and manipulate functions without relying on variable names or specific values.


## M Combinator

$M = λf.f f$

The $M$ combinator takes a function $f$ and applies it to itself. It's sometimes called the "self-application combinator" because it applies the function to itself.

The $M$ combinator has interesting and powerful properties. It can be used to represent recursion and fixed-point operators in combinatory logic. By applying a function to itself, it allows for self-reference, enabling the definition of functions that refer to themselves in a recursive manner. This makes the $M$ combinator a fundamental building block in expressing various computations and recursive functions in combinatory logic.


## OR Combinator

The $OR$ combinator is used to represent logical OR operations.

$OR = λx.λy.x \text{ True } y$

The $OR$ combinator takes two arguments, $x$ and $y$, and applies $x$ to the constant "True" and then to $y$. It behaves similarly to the logical OR operation where it returns `True` if at least one of the arguments (`x` or `y`) is `True`, and `False` otherwise.

As it is not usually a part of languages, a slightly altered (`(a → b) → (a → b) → b`) version is used.


## (Big) Phi Combinator

$Φ = λf.((λx.f (xx))(λx.f (xx)))$

The Big Phi combinator is a fixed-point combinator, and its primary role is to find the fixed point of a function. A fixed point of a function is a value that remains unchanged when the function is applied to it.

The expression $λx.f (xx)$ creates a self-application, where the argument $x$ is applied to itself, and then the function $f$ is applied to it. The Big Phi combinator essentially takes a function $f$ and finds a fixed point for it by repeatedly applying the self-application. When a fixed point is reached, applying $f$ to that point results in the same point.

The Big Phi combinator is an essential part of combinatory logic and has deep connections to mathematical logic and the theory of computation. It plays a crucial role in understanding the concept of fixed points, which is fundamental in various areas of computer science, particularly in the theory of recursive functions and functional programming.


## Psi Combinator

The $Psi$ combinator is used to model selection or conditional branching in a similar way to the conditional statements (such as "if-else" statements) in traditional programming languages.

$Psi = λxyz.((xz)y)$

The $Psi$ combinator takes three arguments ($x$, $y$, and $z$) and behaves as follows:
  * It applies the first argument $(x)$ to the third argument $(z)$.
  * The result of this application is then applied to the second argument $(y)$.

The $Psi$ combinator represents conditional selection. If $x$ is a function that returns `true` or `false`, you can think of it as `if x then z else y` in a way similar to conditional statements in traditional programming languages.


## Th Combinator

The $Th$ combinator is used to perform function composition, specifically taking two functions and composing them into a single function.

$Th = λfgh.f(g h)$

The $Th$ combinator takes three functions as arguments: $f$, $g$, and $h$. It applies $h$ to $g$, and then it applies the result to $f$. In essence, it composes the functions $g$ and $h$ and feeds the result into the function $f$.

The $Th$ combinator is used to build more complex functions by combining simpler functions, and it plays a crucial role in understanding how functions can be manipulated in a purely abstract and variable-free manner.


## V Combinator

The $V$ combinator is used to duplicate a single argument, essentially replicating it.

$V = λx.xx$

The $V$ combinator takes a single argument $x$ and applies it to itself. This results in the duplication of the argument. It's essentially equivalent to creating a pair of values, where the two values are the same.