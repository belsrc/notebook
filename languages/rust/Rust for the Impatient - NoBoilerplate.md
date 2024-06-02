---
tags:
  - rust
gardening: 🌿
reference:
  - https://www.youtube.com/watch?v=br3GIIQeefY
  - https://fasterthanli.me/articles/a-half-hour-to-learn-rust
---
## Rust for the Impatient

- `let` introduces a variable binding.

```rust
let x = 42;
```

- Can be given type annotations.

```rust
let x: i32 = 42;
```

- Temporal Dead Zone exists in Rust (JS-familiar).

```rust
let x;

foobar(x);
// borrow of possibly-uninitialized `x`

x = 42;
```

- `_` is a special name, or lack of name.

```rust
let _ = 42;

// throw away result
let _ = get_thing();
```

- Rust has tuples.

```rust
let pair = ('a', 15);
// or let pair: (char, i32) = ('a', 15);

pair.0;
pair.1;
```

- Rust compiler can almost always infer the types that you are using. So you rarely need to provide them. But if it is needed:

```rust
let pair: (char, i32) = ('a', 17);
```

- Tuples can be destructured (JS-familiar).

```rust
let (some_char, some_int) = ('a', 15);
let (l, r) = slice.split_at(middle);
let (_, right) = slice.split_at(middle);
// throw away first item in tuple
```

- Nearly everything in Rust is an expression (expression based, JS-familiar).

- Semicolons marks the end of a statement.

- `fn` declares a function.

```rust
// f -> void
fn greet() {
  println!("Hi there!");
}

// f -> i32
fn dice_roll() -> i32 {
  4
}
```

- Brackets (`{}`) declares a block which has its own scope (JS-familiar).

```rust
let x = "out";
{
  // this is a different `x`
  let x = "in";
  println!("{}", x);
}
println!("{}", x);

//> "in"
//> "out"
```

- Blocks are also expressions (which means they evaluate to a value).

```rust
// these are the same thing
let x = 42;
let x = { 42 };
```

- Can have multiple statements in a block. The final one is called the tail. This is what the whole block will evaluate to.

```rust
let x = {
  let y = 1;
  let z = 2;

  y + z // this is the tail
};

// x = 3
```

- The "tail" is why omitting the semicolon at the end of a function is the same as returning

```rust
// these are equivalent
fn dice_roll() -> i32 {
  return 4;
}

fn dice_roll() -> i32 {
  4 // notice the missing semicolon
}
```

- `if` conditionals are also expressions.

```rust
fn dice_roll() -> i32 {
  if feeling_lucky {
    5
  } else {
    4
  }
}
```

- `match` is also an expression.

```rust
fn dice_roll() -> i32 {
  match feeling_lucky {
    true => 5,
    false => 4,
  }
}
```

- Dots are typically used to access fields or methods on a value.

```rust
let a = (10, 20);
a.0; // 10

let amos = get_some_struct();
amos.nickname;

let nick = "foobar";
nick.len(); // 6
```

- The double colon `::` is similar but operates on namespaces.

```rust
let least = std::cmp::min(3, 8);

// crate::file::function
```

- The `use` directive can bring names from other namespaces into scope.

```rust
use std::cmp::min;

let least = min(7, 1);
```

- Types are namespaces too. And methods can be called as regular functions.

```rust
let x = "amos".len();
let x = str::len("amos");
```

- Structs can be declared with the `struct` keyword.

```rust
struct Number {
  odd: bool,
  value: i32,
}
```

- They can be initialized using literals.

```rust
let x = Number { odd: false, value: 2 };
let y = Number { value: 3, odd: true };
// order doesn't matter
```

- You can spread a struct into another.

```rust
struct Point {
  x: f64,
  y: f64,
}

let p1 = Point { x: 1.0, y: 3.0 };

let p2 = Point {
  x: 14.0,
  ..p1 // can only happen in the last postion
};
```

- Structs can be destructured.

```rust
let p = Point { x: 3.0, y: 6.0 };
let Point { x, y } = p;

let Point { x, .. } = p;
// this throws away `v.y`
```

- `let` patterns can be used as conditions

```rust
fn print_number(n: Number) {
  if let Number { odd: true, value } = n {
    println!("Odd number: {}", value);
  } else if let Number { odd: false, value } = n {
    println!("Even number: {}", value);
  }
}
```

- `match` arms are patterns. A match has to be exhaustive. An `_` can be used as a catch-all.

```rust
fn print_number(n: Number) {
  match n.value {
    1 => println!("One"),
    2 => println!("Two"),
    _ => println!("{}", n.value),
  }
}
```

- You can declare methods on your own types.

```rust
struct Number {
  odd: bool,
  value: i32,
}

impl Number {
  fn is_positive(self) -> bool {
    self.value > 0
  }
}

let minus_two = Number {
  odd: false,
  value: -2,
};

println!("{}", minus_two.is_positive());
```

- Variable bindings are immutable by default. Interior can't be altered.

```rust
let n = Number {
  odd; true,
  value: 7,
};

n.odd = false;

// error: cannot assign to `n.odd` as `n` is not declared to be mutable
```

- `mut` makes a variable binding mutable. (Variable bindings are immutable by default.)

```rust
let mut n = Number {
  odd: true,
  value: 9,
};

n.value = 7; // all good
```

- Functions can be generic.

```rust
fn foobar<T>(arg: T) {
  // work with arg
}

fn left_right<L, R>(left: L, right: R) {
  // do work
};
```

- Type parameters can have constraints.

```rust
fn print<T: Display>(value: T) {
  println!("value = {}", value);
}
```

- Which is shorthand for the longer version

```rust
fn print<T>(value: T)
where
  T: Display,
{
  println!("value = {}", value);
}
```

- They can also be more complicated, requiring a type parameter to implement multiple traits.

```rust
fn compare<T>(left: T, right: T)
where
  T: Debug + PartialEq,
{
  println!("{:?} {} {:?}", left, if left == right { "==" } else { "!=" }, right);
}
```

- Structs can also me generic

```rust
struct Pair<T> {
  a: T,
  b: T,
}
```

- Standard library type `Vec`, which is a **Heap** allocated array, is generic

```rust
let mut v1 = Vec::new();
v1.push(1);
// v1 == Vec<i32>

let mut v2 = Vec::new();
v2.push(false);
// v2 == Vec<bool>
```

- Has a macro which gives, more or less, Vec literals

```rust
let v1 = vec![1, 2, 3];
let v2 = vec![true, false, true];
```

- All of these invoke a macro. 

```rust
name!()
name![]
name!{}
```

- Macros just expand to regular code. Recognizable by the bang `!` at the end of the name.

```rust
println!("{}", "Hello there!");

// expands to roughly

use std::io::{self, Write};
io::stdout()
   .lock()
   .write_all(b"Hello there!\n")
   .unwrap();
```

- Panic violently stops execution with an error message, a file name and a line number.

```rust
panic!("This panics");

// thread 'main' panicked at 'This panics', src/main.rs:3:5
```

- Some methods also Panic

```rust
let o1: Option<i32> = Some(128);
o1.unwrap(); // this is fine

let o2: Option<i32> = None;
o2.unwrap(); // this panics!

// thread 'main' panicked at
// 'called `Option::unwrap()` on a `None` value,
// src/libcore/option.rs:378:21
```

- `Option` is not a `struct`, it's an `enum` with two variants

```rust
enum Option<T> {
  None,
  Some(T),
};

impl<T> Option<T> {
  fn unwrap(self) {
    match self {
      Self::Some(t) => t,
      Self::None => panic!(...)
    }
  }
}
```

- `Result` is also an `enum`. Also Panics when `unwrap` is called on the unhappy path.

```rust
enum Result<T, E> {
  Ok(T),
  Err(E),
};
```

- Functions that can fail typically return a result.

```rust
let s1 = str::from_utf8(
  &[240, 159, 141, 137]
);

println!("{:?}", s1);
// Ok("🍉")

let s2 = str::from_utf8(&[195, 40]);
println!("{:?}", s2);
// OErr(Utf8Error { valid_up_to: 0, error_len: Some(1) })
```

> [!error] Note
> This is kind of a huge thing and the main reason I want control structures in JS apps.

- If you want to Panic in case of failure, you can `unwrap`.

```rust
let s1 = str::from_utf8(
  &[240, 159, 141, 137]
).unwrap();

println!("{:?}", s1);
// "🍉"

let s2 = str::from_utf8(&[195, 40]).unwrap();
// thread 'main' panicked at
// 'called `Result::unwrap()` on an `Err` value: 
// `Utf8Error { valid_up_to: 0, error_len: Some(1) }`'
```

- You can also use `expect` for a custom error message.

```rust
let s = str::from_utf8(&[195, 40]).expect("valid utf-8");
// thread 'main' panicked at 'valid utf-8:
// `Utf8Error{ valid_up_to: 0, error_len: Some(1) }`'
```

- You can also `match` and handle the error.

```rust
let melon = &[240, 159, 141, 137];
match str::from_utf8(melon) {
    Ok(s) => println!("{}", s),
    Err(e) => panic!(e),
}
// 🍉
```

- Or you can use `if let` to safely destructure the inner value, if it is OK.

```rust
let melon = &[240, 159, 141, 137];
if let Ok(s) = str::from_utf8(melon) {
    println!("{}", s);
}
// 🍉
```

- Or, you can bubble up the error

```rust
let melon = &[240, 159, 141, 137];
match std::str::from_utf8(melon) {
    Ok(s) => println!("{}", s),
    Err(e) => return Err(e),
}
Ok(())
```

- The pattern of unwrapping a Result if the value is Ok or returning it if it is an Err is so common that Rust added syntax for it.

```rust
// let melon = &[240, 159, 141, 137];
let s = str::from_utf8(melon)?;  // <- the `?`
// println!("{}", s);
// Ok(())

// Does the same thing as the match example above. And is the normal error handling pattern in Rust. Where you are just trying to write the happy path.
```

- Iterators are computed lazily, on demand. So an iterator from 1...∞ can fit in RAM.

```rust
let natural_numbers = 1..;
```

- This is call a "range", the most basic iterator. They can be open at the bottom or the top. Or you can specify both exactly. Computation only happens when the iterator is called.

```rust
// 0 or greater
(0..).contains(&100);
// 20 or less than 20
(..=20).contains(&20));
// only 3, 4, 5
(3..6).contains(&4));
```

- Anything that is iterable can be used in a for loop.

```rust
// with a vec
for i in vec![52, 49, 21] {
  println!("I like number {}", i);
}

// with a slice
for i in &[52, 49, 21] {
  println!("I like number {}", i);
}

// or an actual iterator
for c in "rust".chars() {
  println!("Give me a {}", c);
}
```

> [!Error] Insert Traits

> [!Error] Lifetimes

> [!Error] Dereference

> [!Error] Closures & Returning Closures
