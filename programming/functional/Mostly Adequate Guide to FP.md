---
tags:
  - functional
gardening: 🌱
reference:
  - https://github.com/MostlyAdequate/mostly-adequate-guide
---
## Chapter 1: What Ever Are We Doing?

```js
const conjoin = (flockX, flockY) =>
  flockX + flockY;

const breed = (flockX, flockY) =>
  flockX * flockY;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result = conjoin(
  breed(flockB, conjoin(flockA, flockC)), breed(flockA, flockB)
);
```

There is benefit to calling a spade a spade. Has we scrutinized our custom functions more closely, we would have discovered that we're just working with simple addition (`conjoin`) and multiplication (`breed`).

There's really nothing special about these two functions other than their names.

```js
const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

const flockA = 4;
const flockB = 2;
const flockC = 0;
const result = add(
  multiply(flockB, add(flockA, flockC)), multiply(flockA, flockB)
);
```

And with that, we gain the knowledge of the ancients.

```js
// associative
add(add(x, y), z) === add(x, add(y, z));

// commutative
add(x, y) === add(y, x);

// identity
add(x, 0) === x;

// distributive
multiply(x, add(y, z)) === add(multiply(x, y), multiply(x, z));
```

Let's see if we can use these properties to simplify our little seagull program.

```js
// Original line
add(multiply(flockB, add(flockA, flockC)), multiply(flockA, flockB));

// Apply the identity property to remove the extra add
// (add(flockA, flockC) == flockA)
add(multiply(flockB, flockA), multiply(flockA, flockB));

// Apply distributive property to achieve our result
multiply(flockB, add(flockA, flockA));
```

We'll want to represent our specific problem in terms of generic, composable bits and then exploit their properties for our own selfish benefit. It will take a bit more discipline than the "anything goes" approach of imperative programming. The payoff of working within a principled, mathematical framework will truly astound you.

## Chapter 2: First Class Functions

We can treat functions like any other data type and there is nothing particularly special about them - they may be stored in arrays, passed around as function parameters, assigned to variables, and what have you.

A solid understanding of this is critical before moving on, so let's examine a few more examples.

```js
const getServerStuff = callback => ajaxCall(json => callback(json));

// this line
ajaxCall(json => callback(json));

// is the same as this line
ajaxCall(callback);

// so refactor getServerStuff
const getServerStuff = callback => ajaxCall(callback);

// ...which is equivalent to this
const getServerStuff = ajaxCall;
```

And that, folks, is how it is done.

```js
const BlogController = {
  index(posts) { return Views.index(posts); },
  show(post) { return Views.show(post); },
  create(attrs) { return Db.create(attrs); },
  update(post, attrs) { return Db.update(post, attrs); },
  destroy(post) { return Db.destroy(post); }
};
```

This is 90% fluff.

```js
const BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};
```

## Chapter 03: Pure Happiness with Pure Functions

A pure function is a function that, given the same input, will always return the same output and does not have any observable side effect. Take `slice` and `splice`, are two functions that do the exact same thing - in a vastly different way. We say `slice` is _pure_ because it returns the same output per input every time. `splice`, however, will chew up its array and spit it back out forever changed which is an observable effect.

```js
const xs = [1, 2, 3, 4, 5];

// pure
xs.slice(0, 3); // [1, 2, 3]
xs.slice(0, 3); // [1, 2, 3]
xs.slice(0, 3); // [1, 2, 3]

// impure
xs.splice(0, 3); // [1, 2, 3]
xs.splice(0, 3); // [4, 5]
xs.splice(0, 3); // []
```

In functional programming, we dislike unwieldy functions like `splice` that _mutate_ data. This will never do as we're striving for reliable functions that return the same result every time.

```js
// impure
const minimum = 21;
const checkAge = age => age >= minimum;

// pure
const checkAge = age => {
  const minimum = 21;
  return age >= minimum;
};

// [better]
const compareAge = minimum => age => age >= minimum;
const checkAge = compareAge(21);
```

In the impure portion, it depends on system state which is disappointing because it increases the cognitive load by introducing an external environment. It might not seem like a lot in this example, but this reliance upon state is one of the largest contributors to system complexity. This may return different results depending  on factors external to input, which not only disqualifies if from being pure, but also puts our minds through the ringer each time we're reasoning about the software.

Its pure form, on the other hand, is completely self sufficient. We can also make `minimum` immutable, which preserves the purity as the state will never change.

We'll be referring to _effect_ as anything that occurs in our computation other than the calculation of a result. There's nothing intrinsically bad about effects and we'll be using  them all over the place in the chapters to come. It's that _side_ part that bears the negative connotation.

A _side effect_ is a change of system state or _observable interaction_ with the outside world that occurs during the calculation of a result.

Side effects may include, but are not limited to:

- changing the file system
- inserting a record into a database
- making an http call
- mutations
- printing to the screen/logging
- obtaining user input
- querying the DOM
- accessing system state

Any interaction with the world outside of a function is a side effect. The philosophy of functional programming postulates that side effects are a primary cause of incorrect behavior. It is not that we're forbidden to use them, rather we want to contain them and run them in a controlled way.

A function is a special relationship between values: Each of its input values gives back exactly one output value.

![](../../images/functions/function-assoc-dark.png)

Pure functions _are_ mathematical functions and they're what functional programming is all about. Programming with these little angels can provide huge benefits.

Pure functions can always be cached by input. This is typically done using a technique called memoization.

```js
const squareNumber = memoize(x => x * x);

squareNumber(4); // 16
squareNumber(4); // 16, returns cache for input 4
squareNumber(5); // 25
squareNumber(5); // 25, returns cache for input 5
```

A simplified implementation.

```js
const memoize = f => {
  const cache = {};

  return (...args) => {
    const argStr = JSON.stringify(args);
    cache[argStr] = cache[argStr] || f(...args);
    return cache[argStr];
  };
};
```

Pure functions are completely self contained. Everything the function needs is handed to it on a silver platter. Pure function must be honest about its dependencies and, as such, tell us exactly what it's up to. We're forced to "inject" dependencies, or pass them in  as arguments, which makes our app much more flexible because we've parameterized.

Contrary to "typical" methods and procedures in imperative programming rooted deep in their environment via state, dependencies and available effects, pure functions can be run anywhere our hearts desire.

"The problem with object-oriented languages is they've got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana...and the entire jungle." Erlang creator, Joe Armstrong.

Pure functions make testing much easier. We don't have to mock a "real" payment gateway or setup and assert the state of the world after each test. We simply give the function input and assert output. In fact. We find the functional community pioneering new test tools that can blast our functions with generated input and assert that properties hold on the output.

Many believe the biggest win when working with pure functions is _referential transparency_. Code is referentially transparent when it can be substituted for its evaluated value without changing the behavior of the program.

We can use a technique called _equational reasoning_ wherein one substitutes "equals for equals" to reason about code. This ability to reason about code is terrify for refactoring and understanding code in general.

We can run any pure function in parallel since it does not need access to shared memory and it cannot, by definition, have a race condition due to some side effect.

## Chapter 04: Currying

The concept is simple: You can call a function with fewer arguments than it expects. It returns a function that takes the remaining arguments.

> [!NOTE]
> This isn't, strictly speaking, correct. [Currying](mathematics/Algebra/Curry.md) is always one by one. [Partial](mathematics/Algebra/Curry.md#contrast-with-partial-function-application) is the function being wrapped plus one argument. And then takes the rest one by one.

```js
const add = x => y => x + y;

const increment = add(1);

increment(2); // 3
```

Here we've made a function `add` that takes one argument and returns a function. By calling it, the returned function remembers the first argument from then on via the closure.

```js
const match = curry((what, s) => s.match(what));
const filter = curry((f, arr) => arr.filter(f));
const map = curry((f, arr) => arr.map(f));
```

The pattern I've followed is a simple, but important one. I've strategically positioned the data we're operating on (String, Array) as the last argument. It will become clear as to why upon use.

> [!NOTE]
> Since it wasn't really covered later, the above concept is called "data last". This allows us to add all of the arguments that are more likely to be static first. And then finally, the value that is ultimately going to be acted up.

Currying is useful for many things. We can make new function just by giving our base function some arguments. We also have the ability to transform any function that works on single elements into a function that works on arrays simply by wrapping it with `map`.

Giving a function fewer arguments than it expects is typically called _partial application_. Partially applying a function can remove a lot of boiler plate code.

> [!NOTE]
> Again, this isn't entirely correct.

## Chapter 05: Coding by Composing

```js
const compose = (f, g) => x => f(g(x));
```

