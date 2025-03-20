---
tags:
  - language
  - c
gardening: 🌱
date: 2025-02-10
reference:
  - https://www.youtube.com/watch?v=rJrd2QMVbGM
---
## App Entry

A function names `main` is always the entry point of the application.

```c
int main() {
  return 0;
}
```

The return value of the main function is the exit code of the program. `0` means success, anything else means a failure.

## Basic Types

- `int` - Integer: 4 bytes
- `float` - Floating point number: 4 bytes
- `double` - Double-precision floating point: 8 bytes
- `char` - Character: 1 byte
- `char *` - Array of characters/string: _n_ bytes

## Variables

Variables are declared using `<type> <name> <? = value>;` format. Variables are mutable but can not change types.

```c
int main() {
  int x = 5;
  x = 10; // this is ok
  x = 15; // still ok

  int y = 5;
  float y = 3.14; // error
}
```

You can change a variables type by casting it though.

```c
int x = 5;
float y = (float)x;
```

If you need an immutable variable you can use the `const` keyword.

```c
int main() {
  const int x = 5;
  x = 10; // error
}
```

## Functions

Functions have the signature `<return type> <name>(<args | void>) { ... }`.

```c
float add(int x, int y) {
  return (float)(x + y);
}
```

## Void

`void` is a return type for a function that returns normally but provides no result value. And is similar to the `unit type`. The void type may also replace the argument list of a function prototype to indicate that the function takes no arguments.

```c
int get_integer(void) {  
  return 42;
}

void print_integer(int x) {
  printf("this is an int: %d", x);
}
```

## Printing

Printing is done with the `printf` function from the `stdio.h` library.

```c
#include <stdio.h>

int main() {
  printf("Hello, world/n");
}
```

In C, you have to tell it how to print particular values using format specifiers.

- `%d` - digit
- `%c` - character
- `%f` - floating point
- `%s` - string (char *)

[Full Spec](https://cplusplus.com/reference/cstdio/printf/#:~:text=Parameters-,format,-C%20string%20that)

```c
printf("Hello, %s. You're %d years old.\n", name, age);
```

## Math Operators

Normal list of math and assignment operators.

```c
x + y;
x - y;
x * y;
x / y;

x += 1;
x -= 1;
x *= 1;
x /= 1;

x++; // += 1
x--; // -= 1
```

## If Statements

Similar to pretty much all other languages (based on C).

```c
if (x > 3) {
  printf("x is greater than 3\n");
} else if (x == 3) {
  printf("x is 3\n");
} else {
  printf("x is less than 3\n");
}
```

Ternaries are also supported

```c
int a = 5;
int b = 10;
int max = a > b ? a : b;
```

## For Loop

Normal `for` syntax.

```c
for (<initialization>; <condition>; <final-expression>) {
  // Loop Body
}

for (int i = 0; i < 5; i++) {
	printf("%d\n", i);
}
```

## While Loop

Normal `while` loop syntax

```c
while (<condition>) {
	// Loop Body
}

while (i < 5) {
	printf("%d\n", i);
	i++;
}
```

## Do While Loop

Normal `do...while` loop syntax.

```c
do {
	// Loop Body
} while (<condition>);

do {
	printf("i = %d\n", i);
	i++;
} while (i < 5);
```

## Structs

`struct` is your plain object in other languages. It is a way to group related values together.

```c
struct Human {
	int age;
	char *name;
	int is_alive;
};
```

Structs are just a block of memory. Each field is stored in the order that it was defined. One after the other. And, by default, structs are passed by value. That is, they are copied to the function. 

They can be initialized in a number of ways:

#### Zero Initializer

```c
struct City c = {0};
```

This sets all of the fields to `0` values. _(ZII mentioned?)_

#### Positional Initializer

```c
struct City c = {"San Francisco", 37, -122};
```

#### Designated Initializer

```c
struct City c = {
  .name = "San Francisco",
  .lat = 37,
  .lon = -122
};
```

#### Field access

Accessing a field in a struct is done using the `.` operator.

```c
struct Coord coord = { .x = 12, .y = 23, .z = 13 };
int x = coord.x;
```

If it is a pointer to a `struct` it is slightly different. In this case you use `->`. This effectively dereferences the struct pointer and accesses the field in one step.

```c
struct Coord coord = { .x = 12, .y = 23, .z = 13 };
struct Coord *coord_ptr = &coord;
int x = coord_ptr -> x;
```

## Typedef "Aliases"

You can alias the type definition of a struct using the `typedef` keyword and giving it an aliased name.

```c
typedef struct Coord {
  int x;
  int y;
  int z;
} coord_t;

coord_t coord_one = { .x = 12, .y = 23, .z = 13 };

// Can still use ... but it is simpler to use the type alias
struct Coord coord_one = { .x = 12, .y = 23, .z = 13 };
```

When using `typedef`, you can optionally skip giving the struct a name.

```c
typedef struct {
  int x;
  int y;
  int z;
} coord_t;
```

Omitting the name means that you can only refer to it using the alias `coord_t` as it no longer has a name to use with the `struct <name>` syntax.

## Pointers

`&` address of operator allows you to get the memory address of a variable.

```c
int *num_ptr = &num;
```

`*` is used to declare a pointer. It is also used to dereference a pointer (get value). 

```c
int *num_ptr = &num; // num_ptr holds address at num
int num2 = *num_ptr; // num2 holds the dereferenced num_ptr. num val copy.
```

Pointers always have the same size since they are just memory addresses. And are based on the underlying system architecture.  `4 bytes` on a `32-bit` system and `8 bytes` on a `64-bit` system.

```c
int *intPtr;
char *charPtr;
double *doublePtr;
printf("int pointer: %zu bytes\n", sizeof(intPtr));
printf("char pointer: %zu bytes\n", sizeof(charPtr));
printf("double pointer: %zu bytes\n", sizeof(doublePtr));
// int pointer: 4 bytes
// char pointer: 4 bytes
// double pointer: 4 bytes
```

## Arrays

Array size needs to be specified. Like `TypedArray`. Initialized using `{ ... }`.

```c
int numbers[5] = {1, 2, 3, 4, 5};
```

Array is just a pointer to first element.

```c
int numbers[5] = {1, 2, 3, 4, 5};
int *numbers_ptr = number;
```

That means pointer arithmetic works the same as indexing.

```c
numbers[2] == *(numbers + 2);
```

When you add an integer to a pointer, the resulting pointer is offset by that integer times the size of the data type.

| Address | Element    | Pointer     | Value |
| ------- | ---------- | ----------- | ----- |
| 0x1000  | numbers[0] | numbers + 0 | 1     |
| 0x1004  | numbers[1] | numbers + 1 | 2     |
| 0x1008  | numbers[2] | numbers + 2 | 3     |
| 0x100C  | numbers[3] | numbers + 3 | 4     |
| 0x1010  | numbers[4] | numbers + 4 | 5     |
Array of structures (AoS) are handled as one would expect.

```c
struct Coordinate = {
  int x;
  int y;
  int z;
};

struct Coordinate points[3] = {
  {1, 2, 3},
  {4, 5, 6},
  {7, 8, 9}
};

printf("points[1].x = %d, points[1].y = %d, points[1].z = %d\n",
  points[1].x, points[1].y, points[1].z
);
// points[1].x = 4, points[1].y = 5, points[1].z = 6
```

The memory size of an array is just $\text{data type size} * n\text{ number of elements}$ .

```c
int intArray[10];
char charArray[10];
double doubleArray[10];
printf("int array: %zu bytes\n", sizeof(intArray));
printf("char array: %zu bytes\n", sizeof(charArray));
printf("double array: %zu bytes\n", sizeof(doubleArray));
// int array: 40 bytes
// char array: 10 bytes
// double array: 80 bytes
```

Since arrays are basically just pointers, and we know that structs are contiguous in memory, we can cast the array of structs to an array of integers (convert to an interleaved array).

```c
struct Coordinate = {
  int x;
  int y;
  int z;
};

struct Coordinate points[3] = {
  {1, 2, 3},
  {4, 5, 6},
  {7, 8, 9}
};

int *points_start = (int *)points;

for (int i = 0; i < 9; i++) {
  printf("points_start[%d] = %d\n", i, points_start[i]);
}
```

Arrays decay to pointers when used in expressions containing pointers.

```c
int arr[5];
int *ptr = arr;          // 'arr' decays to 'int*'
int value = *(arr + 2);  // 'arr' decays to 'int*'
```

Array will also decay when they are passed to functions. e.g. the pointer is passed to the function. Unlike primitive types and structs that get copied to the function.

## Strings

Strings are any number of characters terminated by a null character (`\0`). And are a pointer to the first element of a character array.

```c
char *msg = "This is some message in code"
```

Most string manipulation in C is done using pointers to move around the `char` array and the null terminator is critical for determining the end of the string.

The length of a C string is determined by the position of the null terminator (`'\0'`). Functions like `strlen` calculate the length of a string by iterating through the characters until the null terminator is encountered. This lack of length storage requires careful management to avoid issues such as buffer overflows and off-by-one errors during string operations.

[Other string functions in the standard library.](https://en.cppreference.com/w/c/string/byte)

You can declare strings in C using either arrays or pointers. The output is the same.

```c
char str1[] = "Hi";
char *str2 = "Snek";
printf("%s %s\n", str1, str2);
```

## Forward Declaration

#### Recursive Reference

```c
typedef struct Node {
  int value;
  node_t *next; // error: unknown type name 'node_t'
} node_t;
```

In order to get around this 

```c
typedef struct Node node_t;

typedef struct Node {
  int value;
  node_t *next;
} node_t;
```

#### Circular Reference

```c
struct Person {
  char *name;
  computer_t *computer; // error: unknown type name 'computer_t'
} person_t;

struct Computer {
  char *brand;
  person_t *owner; // error: expected specifier-qualifier-list before 'person_t'
} computer_t;
```

Fix

```c
typedef struct Computer computer_t;
typedef struct Person person_t;

struct Person {
  char *name;
  computer_t *computer;
};

struct Computer {
  char *brand;
  person_t *owner;
}; // notice no typedef
```

## Enums

Enums are a list of integers (`0` based) constrained to a new type, where each is given an explicit name. They are _not_ a collection type like a struct or an array.

```c
enum DaysOfWeek {
	MONDAY,
	TUESDAY,
	WEDNESDAY,
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY,
};
```

Like with structs, you can use `typedef` to make using them simpler.

```c
typedef enum DaysOfWeek {
	MONDAY,
	TUESDAY,
	WEDNESDAY,
	THURSDAY,
	FRIDAY,
	SATURDAY,
	SUNDAY,
} days_of_week;
```

You can also give the enum a specific value.

```c
typedef enum {
  HTTP_BAD_REQUEST = 400,
  HTTP_UNAUTHORIZED = 401,
  HTTP_NOT_FOUND = 404,
  HTTP_I_AM_A_TEAPOT = 418,
  HTTP_INTERNAL_SERVER_ERROR = 500
} HttpErrorCode;
```

## Switch Case

Enums can be used with switch  statements in order to avoid magic numbers, use descriptive names, and, with modern tooling, provide exhaustive checks. 

```c
switch (logLevel) {
  case LOG_DEBUG:
    printf("Debug logging enabled\n");
    break;
  case LOG_INFO:
    printf("Info logging enabled\n");
    break;
  case LOG_WARN:
    printf("Warning logging enabled\n");
    break;
  case LOG_ERROR:
    printf("Error logging enabled\n");
    break;
  default:
    printf("Unknown log level: %d\n", logLevel);
    break;
}
```

Enums are generally the same size and `int`. But if one of the values exceeds what an `int` can hold, the compiler will increase the size. To `unsigned int` and `long`. From the compiler's perspective, enums are just fancy integers.

## Union

Unions in C can hold **one** of several types.

```c
typedef union AgeOrName {
  int age;
  char *name;
} age_or_name_t;
```

This union can hold _either_ an `int` or a `char*`, but not both at the same time. We provide the list of possible types so the compiler knows the _maximum_ potential memory required.

```c
age_or_name_t lane = { .age = 29 };
printf("age: %d\n", lane.age);
// age: 29
```

But what happens if we try to access the `name` field?

```c
printf("name: %s\n", lane.name);
// name:
```

We get nothing.

A `union` only reserves enough space to hold the largest type in the union and then **all** of the fields _use the same memory._ Put simply, setting the value of `age` overwrites the value of `name` and vice versa. And you should only access the field that you set.

A downside of unions is that the size of the union is the size of the _largest_ field in the union.

```c
typedef union IntOrErr {
  int data;
  char err[256];
} int_or_err
```

This union is designed to hold an `int` 99% of the time. However, when the program encounters an error, instead of storing an integer here, it will store an error message. The trouble is that it's incredibly inefficient because it allocates 256 bytes for every `int` that it stores!

#### Uses

**Tagged Union**

```c
enum Type { INTS, FLOATS, DOUBLE };
struct S {
  Type s_type;
  union {
    int s_ints[2];
    float s_floats[2];
    double s_double;
  };
};

void do_something(struct S *s) {
  switch(s->s_type) {
    case INTS:  // do something with s->s_ints
      break;
    case FLOATS:  // do something with s->s_floats
      break;
    case DOUBLE:  // do something with s->s_double
      break;
  }
}
```

**Accessing Underlying Memory**

```c
typedef union {
    struct {
        unsigned char byte1;
        unsigned char byte2;
        unsigned char byte3;
        unsigned char byte4;
    } bytes;
    unsigned int dword;
} HW_Register;

HW_Register reg;
reg.dword = 0x12345678;
reg.bytes.byte3 = 4;
```

The struct acts as a lens into the unsigned int's value. Byte by byte.

