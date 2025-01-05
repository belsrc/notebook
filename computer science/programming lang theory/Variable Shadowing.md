---
tags:
  - comp-sci
  - language
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Variable_shadowing
---
In computer science, variable shadowing occurs when a variable declared within a certain scope (decision block, method, or inner class) has the same name as a variable declared in an outer scope. At the level of identifiers (names, rather than variables), this is known as name masking. This outer variable is said to be shadowed by the inner variable, while the inner identifier is said to _mask_ the outer identifier.

```js
function myFunc() {
  let someVar = 'test';

  if (true) {
    let someVar = 'new test';

    console.log(someVar); // new test
  }

  console.log(someVar); // test
}

myFunc();
```