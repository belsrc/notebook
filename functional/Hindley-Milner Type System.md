---
tags:
  - functional
gardening: 🌿
---
"As a type inference method, Hindley-Milner is able to deduce the types of variables, expressions and functions from programs written in an entirely untyped style." [(\*)](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system) It is the core type system for several functional languages.

[Need more details. More learning required]


Basic example

```js
// upperCase :: String -> String
const upperCase = str => str.toUpperCase();
```

Basic syntax

```
<fn name> :: <arg type> -> <return type>
```

Assumed to be curried.

```
add :: Number -> Number -> Number
```

```js
const add = x => y => x + y;
```

## Variable Types

Type variables can be denoted by lower case letters. With similar types in the signature using the same letter. The variable types can not be specific (unconstrained), they must work informally on all types. This is also called _Parametric polymorphism_. And in languages like TS, a generic.

```
identity :: a -> a
```

```js
const identity = a => a;
```

```ts
const identity = <A>(a: A) => a;
```

## Array and Higher Order Function Types

Parameters that are arrays are simply wrapped by square brackets. While, for higher order functions, the signature of the function is wrapped with parentheses.

```
map :: (a -> b) -> [a] -> [b]

filter :: (a -> Bool) -> [a] -> [a]
```

```ts
type map = <A, B>(f: ((a: A) => B)) => (a: A[]) => B[];
```

It should be noted. When there are multiple type variables, like `map` in this instance, the type of `b` may or may _not_ be the same type as `a`.


## Type Constraints

You can constrain the type to those that implement a particular interface. This is denoted by the constraint, followed by a fat arrow (_=>_) and then the definition. This is also called _Ad-hoc polymorphism_. And in TS, is like using the `extends` on a generic.

```
<fn name> :: <constraint variable> => <arg type> -> <return type>
```

```
hasElem :: (Eq a) => a -> [a] -> Bool
```

```ts
type ErrMessage = <A extends Error>(a: A) => string;
```

#### See More

[Parametric polymorphism - Wikipedia](https://en.wikipedia.org/wiki/Parametric_polymorphism)

[Ad-hoc polymorphism - Wikipedia](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism)

[Polymorphism - Haskell](https://wiki.haskell.org/Polymorphism)