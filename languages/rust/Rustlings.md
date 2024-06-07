---
tags:
  - rust
gardening: 🌱
reference:
  - https://github.com/rust-lang/rustlings
---
## Variables

Variables are declared using the `let` keyword.

```rust
let x = 5;
```

The compiler can usually infer the type of a variable with no need for additional type annotations.

```rust
let x = 10;
```

If needed (for compiler reasons or readability), types can be annotated in the `let <var>: <type>` format.

```rust
let x: i32 = 10;
```

Like most languages, you must declare and initialize a variable before it can be used.

```rust
let x:i32;
println!("number {}", x); // error
x = 453;
println!("number {}", x); // fine
```

Variables are immutable by default.

```rust
let y = 3;
y = 4;

// error: cannot assign twice to immutable variable `y`
```

But they can be made mutable using the `mut` keyword.

```rust
let mut x = 76;
println!("number {}", x); // error
x = 453;
println!("number {}", x); // fine
```

Variables can be shadowed. Variable shadowing is when a variable declared within a certain scope has the same name as a variable declared in an outer scope.

```rust
fn main() {
  let number = "T-H-R-E-E";
  println!("Spell a Number : {}", number);
  let number = 3;
  println!("Number plus two is : {}", number + 2);
}
```

Constant variables are declared with the `const` keyword. They are always immutable and must have a type annotation.

```rust
const FOO: i32 = 53;
```

## Functions

Functions can declared using the `fn` keyword.

```rust
fn call_me() {}
```

Function parameters need to have type annotations.

```rust
fn call_me(num: i32) {}
```

As well as the return type of the function.

```rust
fn is_even(num: i32) -> bool {
  num % 2 == 0
}
```

Rust distinguishes between expressions and statements. Expressions return a value, while statements return `()` (which is basically `void`). When in a block, terminating the last line (the tail) with a `;` means that it is seen as a statement. 

```rust
fn square(num: i32) -> i32 {
  num * num; // error: mismatched types
}
// This function is returning `()`

fn square(num: i32) -> i32 {
  return num * num; // Works, return
}

fn square(num: i32) -> i32 {
  num * num // Works, notice the removed `;`. Seen as an expression.
}
```

## If

In Rust, the condition does not need to be surrounded by parentheses.  They are also expressions. Each condition is followed by a `{}` blocks.

```rust
// These are equivalent

pub fn bigger(a: i32, b: i32) -> i32 {
    return if a > b { a } else { b };
}

pub fn bigger(a: i32, b: i32) -> i32 {
  if a > b {
    a
  } else {
    b
  }
}
```

In Rust, each branch of the if expression needs to return the same type.

```rust
pub fn foo_if_fizz(fizzish: &str) -> &str {
  if fizzish == "fizz" {
    "foo"
  } else if fizzish == "fuzz" {
    "bar"
  } else {
    "baz"
  }
}
```

## Primitive Types

#### Booleans

Booleans are `bool`.j

```rust
let is_morning: bool = true;
// Granted, due to the compiler inference, this isn't strictly needed.
```

#### Characters

Characters are `char` and use single quotes `'`. 

```rust
let my_first_initial = 'C';
```

#### Arrays

Arrays are defined with the signature `[T; <length>]`. 

```ts
let xs: [i32; 5] = [1, 2, 3, 4, 5];

let ys: [i32; 500] = [0; 500];
```

The `[0; 500]` is a shorthand to fill the array with the value in front of the `;`.

#### Slices

You can create a `slice` (contiguous sequence of elements in a collection) of an array using the `&` (borrow) symbol followed by the array variable and a range value. Range should specify `[starting_index..ending_index]`, where `starting_index` is the first position in the slice and `ending_index` is one more (exclusive) than the last position in the slice.

```rust
let a = [1, 2, 3, 4, 5];
let nice_slice = &a[1..4];
// [2, 3, 4]
```

#### Tuples

Tuples are defined using `()`. They can also be destructured in the same way.

```rust
let cat = ("Furry McFurson", 3.5);
let (name, age) = cat;
```

Tuples use dot notation for accessing indexed values.

```rust
let numbers = (1, 2, 3);
let second = numbers.1;
```

## Vecs

There are two ways define a Vec. Using a `new` as well as a macro.

```rust
let a = Vec::new(); // a.push(5);
let b = vec![1, 2, 3, 4, 5];
```

In order to mutate a mutable Vec in a loop, you have to dereference the variable. This means that it follows the pointer to that underlying value.

```rust
fn vec_loop(mut v: Vec<i32>) -> Vec<i32> {
  for element in v.iter_mut() {
    *element *= 2;
  }

  v
}
```

But this is only needed for in-place mutations. When using a `map` dereferencing isn't needed.

```rust
fn vec_map(v: &Vec<i32>) -> Vec<i32> {
  v.iter()
    .map(|element| {
      element * 2
    })
    .collect()
}
```

## Move Semantics

As mentioned previously in order to change a value, it must be marked as `mut`. This can be combined with shadowing to "make" arguments mutable.

```rust
fn fill_vec(vec: Vec<i32>) -> Vec<i32> {
  let tst = vec; // cannot borrowimmutable local variable

  tst.push(88);

  tst
}

fn fill_vec(vec: Vec<i32>) -> Vec<i32> {
    let mut tst = vec;

    tst.push(88);

    tst
}
```

When and argument is passed to a function and it is not explicitly returned, you can't use the original variable anymore. This a function of the borrow checker and is called "moving" a variable.

```rust
fn main() {
  let vec0 = vec![22, 44, 66];
  let mut vec1 = fill_vec(tmp); // error: borrow of moved value: `vec0`

  assert_eq!(vec0, vec![22, 44, 66]);
  assert_eq!(vec1, vec![22, 44, 66, 88]);
}

fn fill_vec(vec: Vec<i32>) -> Vec<i32> {
  let mut vec = vec;

  vec.push(88);

  vec
}

```

You can get around this by cloning the value being moved.

```rust
fn main() {
    let vec0 = vec![22, 44, 66];
    let tmp = vec0.clone(); // cloned
    let mut vec1 = fill_vec(tmp);

    assert_eq!(vec0, vec![22, 44, 66]);
    assert_eq!(vec1, vec![22, 44, 66, 88]);
}

fn fill_vec(vec: Vec<i32>) -> Vec<i32> {
    let mut vec = vec;

    vec.push(88);

    vec
}
```

Or by making the accepting function borrow the value.

```rust
fn main() {
  let vec0 = vec![22, 44, 66];
  let mut vec1 = fill_vec(&vec0);

  assert_eq!(vec0, vec![22, 44, 66]);
  assert_eq!(vec1, vec![22, 44, 66, 88]);
}

fn fill_vec(vec: &Vec<i32>) -> Vec<i32> {
  let mut vec = vec.clone();

  vec.push(88);

  vec
}
```







nvim exercises/move_semantics/move_semantics3.rs

rustlings verify

rustlings hint primitive_types4