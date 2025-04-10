---
tags:
  - comp-sci
gardening: 🌱
date: 2025-04-09
---
A tagged union, also known as a variant type, sum type, or discriminated union, is a data structure designed to hold a value that can take on several different, but fixed, types. Each value is "tagged" with an indicator (or tag) that specifies its type, allowing the program to safely inspect and work with the stored data.

A tagged union typically consists of two parts:

- **The Tag (Discriminator):** This is a small value, often implemented as an enumeration, that indicates which variant the union currently holds. The tag informs you how to interpret the remaining data.
    
- **The Data (Payload):** This is a memory area capable of storing the actual value. The format and interpretation of this data depend on the tag. Only one variant's data is valid at any given time.

One significant benefit of using tagged unions is safety. By explicitly tagging the union, programmers can write code that checks the tag before accessing the data. This prevents type errors that might occur if incorrect assumptions are made about the data type.

Tagged unions also enable the use of pattern matching, a powerful programming feature that simplifies the process of deconstructing data types. Pattern matching automatically checks the tag and binds the corresponding data, leading to cleaner and more readable code.

Without the tag, traditional unions do not carry type information. As a result, developers must rely on external logic or discipline to determine which part of the union is valid, increasing the risk of errors if the wrong type is assumed. Tagged unions eliminate this risk by associating data with its type information.

## C

In C, creating a tagged union involves combining several data types: `enum` for the tag, `struct` as the container, `union` for the union itself, and another `struct` for the union data.

```c
typedef enum {
    SQUARE,
    CIRCLE,
    RECTANGLE,
    TRIANGLE
} ShapeType;

typedef struct {
	ShapeType type;  // The tag tells us what kind of shape is stored.
	union {
		struct { double side; } square;
		struct { double radius; } circle;
		struct { double width, height; } rectangle;
		struct { double base, height; } triangle;
	} u;
} Shape;

double area(Shape s) {
	switch(s.type) {
		case SQUARE:
			return s.u.square.side * s.u.square.side;
		case CIRCLE:
			return M_PI * s.u.circle.radius * s.u.circle.radius;
		case RECTANGLE:
			return s.u.rectangle.width * s.u.rectangle.height;
		case TRIANGLE:
			return 0.5 * s.u.triangle.base * s.u.triangle.height;
		default:
			return 0;
	}
}

int main() {
	Shape s1 = { 
	  .type = SQUARE,
	  .u.square = { .side = 5 }
	};
	Shape s2 = { 
	  .type = CIRCLE,
	  .u.circle = { .radius = 3 }
	};
	Shape s3 = { 
	  .type = RECTANGLE,
	  .u.rectangle = { .width = 4, .height = 6 }
	};
	Shape s4 = {
	  .type = TRIANGLE,
	  .u.triangle = { .base = 4, .height = 7 }
	};

	printf("Square area: %f\n", area(s1));
	printf("Circle area: %f\n", area(s2));
	printf("Rectangle area: %f\n", area(s3));
	printf("Triangle area: %f\n", area(s4));
	
	return 0;
}
```

## Rust

The Rust language adopts a slightly different approach. It uses `enum`s exclusively, which can hold data and effectively transform them into tagged unions. Additionally, Rust allows for the implementation of methods directly on the `enum`.

```rust
enum Shape {
	Square { side: f64 },
	Circle { radius: f64 },
	Rectangle { width: f64, height: f64 },
	Triangle { base: f64, height: f64 },
}

impl Shape {
	fn area(&self) -> f64 {
		match self {
			Shape::Square { side } => side * side,
			Shape::Circle { radius } => std::f64::consts::PI * radius * radius,
			Shape::Rectangle { width, height } => width * height,
			Shape::Triangle { base, height } => 0.5 * base * height,
		}
	}
}

fn main() {
	let s1 = Shape::Square { side: 5.0 };
	let s2 = Shape::Circle { radius: 3.0 };
	let s3 = Shape::Rectangle { width: 4.0, height: 6.0 };
	let s4 = Shape::Triangle { base: 4.0, height: 7.0 };

	println!("Square area: {}", s1.area());
	println!("Circle area: {}", s2.area());
	println!("Rectangle area: {}", s3.area());
	println!("Triangle area: {}", s4.area());
}
```

## Typescript

JavaScript lacks native support for tagged unions, so TypeScript's approach is somewhat more structured. You need to define each individual type of the union first, and then create a new type that combines those types into a union. Finally, you can implement these types on JavaScript objects.

```ts
type Square = {
	kind: 'square';
	side: number;
}

type Circle = {
	kind: 'circle';
	radius: number;
}

type Rectangle = {
	kind: 'rectangle';
	width: number;
	height: number;
}

type Triangle = {
	kind: 'triangle';
	base: number;
	height: number;
}

type Shape = Square | Circle | Rectangle | Triangle;

function area(shape: Shape): number {
	switch (shape.kind) {
		case 'square':
			return shape.side * shape.side;
		case 'circle':
			return Math.PI * shape.radius * shape.radius;
		case 'rectangle':
			return shape.width * shape.height;
		case 'triangle':
			return 0.5 * shape.base * shape.height;
		default:
			throw new Error('Unknown shape type');
	}
}

const s1: Shape = { kind: 'square', side: 5 };
const s2: Shape = { kind: 'circle', radius: 3 };
const s3: Shape = { kind: 'rectangle', width: 4, height: 6 };
const s4: Shape = { kind: 'triangle', base: 4, height: 7 };

console.log('Square area:', area(s1));
console.log('Circle area:', area(s2));
console.log('Rectangle area:', area(s3));
console.log('Triangle area:', area(s4));
```