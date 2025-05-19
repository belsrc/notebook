### Compose: Associativity

```js
compose(f, compose(g, h)) === compose(compose(f, g), h);

compose(toUpperCase, compose(head, reverse));
// or
compose(compose(toUpperCase, head), reverse);
```

### Compose: Identity

```js
compose(id, f) === compose(f, id) === f;
```

### Functor: Identity

$$
A^{\text{ }\underrightarrow{\text{Id}_x}\text{ }} B = A
$$

```js
F(a).map(x => x) === F(a)
```

### Functor: Composition

$$
A^{\text{ }\underrightarrow{g}\text{ }} B^{\text{ }\underrightarrow{f}\text{ }} C = A^{\text{ }\underrightarrow{f(g)}\text{ }} C
$$

```js
F(a).map(g).map(f) === F(a).map(x => f(g(x)))
```

### Apply: Composition

```js
A(a).ap(A(g).ap(A(f).map(f => g => x => f(g(x))))) === A(a).ap(A(g)).ap(A(f))
```

### Applicative: Identity

```js
A.of(a).ap(A.of(x => x)) === A.of(a)
```

### Applicative: Composition

```js
A.of(a).ap(A.of(g).ap(A.of(f).map(f => g => x => f(g(x))))) === A.of(a).ap(A.of(g)).ap(A.of(f))
```

### Applicative: Homomorphism

```js
A.of(a).ap(A.of(f)) === A.of(f(a))
```

### Applicative: Interchange

```js
A.of(a).ap(A.of(f)) === A.of(f).ap(A.of(fn => fn(a)))
```

### Chain: Associativity

```js
C(a).chain(f).chain(g) === C(a).chain(x => C(f(x)).chain(g))
```

### Monad: Left Identity

```js
M.of(a).chain(f) === M.of(f(a))
```

### Monad: Right Identity

```js
M.of(a).chain(M.of) === M.of(a)
```
