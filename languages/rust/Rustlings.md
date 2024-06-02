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




nvim exercises/functions/functions3.rs