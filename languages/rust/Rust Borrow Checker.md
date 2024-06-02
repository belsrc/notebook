---
tags:
  - rust
gardening: 🌱
---
**Has only two rules:**
1. Data has one owner
2. Data may have multiple readers or one writer

```rust
let rustGuide = Book {};

// multiple readers
loan_to_foo(&rustGuide); // borrowed (read only)
loan_to_bar(&rustGuide); // borrowed (read only)

// still the owner of `rustGuide`, it was just borrowed

retire_book(rustGuide); // giving up ownership

// not allowed to use `rustGuide` anymore

loan_to_foo(&rustGuide); // err: rustGuide moved
```

**Mutable borrowing:**

```rust
let rustGuide = Manuscript {};

let manning_pub = Editor {};
let fred = Editor {};

edit(&mut rustGuide, manning_pub); // mutable borrow
edit(&mut rustGuide, fred); // err: only one mut borrow

// still the owner of `rustGuide`, it was just borrowed

sell(rustGuide); // pass ownership

// not allowed to use `rustGuide` anymore

loan(&rustGuide); // err: rustGuide moved
```