#### Pointers

`&` address at operator.

```c
int *num_ptr = &num;
```

`*` declare pointer/dereference pointer (get value). 

```c
int *num_ptr = &num; // num_ptr holds address at num
int num2 = *num_ptr; // num2 holds the dereferenced num_ptr. num val copy.
```

Pointers always have the same size since they are just memory addresses. And are based on the underlying system architecture.  `4 bytes` on a `32-bit` system and `8 bytes` on a `64-bit` system.

#### Structs

`.` access, or set, struct field.

```c
struct Coord new_coord = { .x = 12, .y = 23, .z = 13 }; // Set named fields
int x = new_coord.x; // get field value
```

`->` struct field access from a pointer. It dereferences the pointer and accesses the field.

```c
struct Coord coord = {12, 23, 13};
struct Coord *coord_ptr = &coord;
int x = coord_ptr -> x;
```

#### Arrays

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

#### Strings

Strings are any number of characters terminated by a null character (`\0`). And are a pointer to the first element of a character array.

```c
char *msg = "This is some message in code"
```

Most string manipulation in C is done using pointers to move around the `char` array and the null terminator is critical for determining the end of the string.

The length of a C string is determined by the position of the null terminator (`'\0'`). Functions like `strlen` calculate the length of a string by iterating through the characters until the null terminator is encountered. This lack of length storage requires careful management to avoid issues such as buffer overflows and off-by-one errors during string operations.

You can declare strings in C using either arrays or pointers. The output is the same.

```c
char str1[] = "Hi";
char *str2 = "Snek";
printf("%s %s\n", str1, str2);
```

