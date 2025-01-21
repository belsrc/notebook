---
tags:
  - functional
gardening: 🌳
---
"Data last" is a design approach in functional programming where the data operated on is passed as the last argument to a function. In other words, the arguments are ordered from the least to the most significant.

Instead of a data first approach:

```js
transform(data, config);
```

One would instead write:

```js
transform(config, data);
```

## Benefits

### Enables Function Composition

By having data last, functions can be chained or composed more effectively.

```js
const f = curry((d, c) => e);
const g = curry((a, b) => c);

const transform = compose(f(x), g(y));

transform(data);
```

### Facilitates Partial Application

Functions can be [curried](../../mathematics/algebra/Curry.md), making it easy to pass initial parameters to configure the function, which will then be ready to accept data later.

```js
const transformData = curry((conf, data) => { /*...*/ });

const configuredTransform = transformData(fnConfig);

const result = configureTransform(myData);
```

### Improved Readability

Having the data at the end of each function call makes it easier to follow the transformation flow when reading a sequence of operations.

```js
const result = process(data, fn1, fn2, fn3);

// vs

const result = fn3(fn2(fn1(data)));
// Enables natural nesting. Which can then be easily composed.
```

### Interoperability with Other Functions

Data last functions integrate well with other higher-order functions like `map` and `filter`, as these functions expect data as their final parameter.

```js
const transformData = curry((conf, data) => { /*...*/ });

const results = list.map(transformData(config));
```

### Consistency with Point-Free Style

Point-free programming minimizes explicit data references, and "data last" complements this approach.

```js
const processList = compose(map(f), filter(g));
```

### Improves Reusability

Functions designed with a "data last" approach are more reusable because they can be easily reconfigured with different initial arguments without requiring the data upfront.