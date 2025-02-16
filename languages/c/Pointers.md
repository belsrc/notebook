Pointers are variables that contain an address of a piece of memory.

Your applications memory can be thought of as a large array of these addresses.

So you can think of these pointers are simple array indexes.

```c
typedef struct Point {
  int x;
  int y;
  int z;
} point_t;

int main() {
  point_t coord = { x: 24, y: 41, z: 87 };
  
  printf("x, y, z: %d, %d, %d\n\n", coord.x, coord.y, coord.z);
  printf("x, y, z addresses: %p, %p, %p\n\n", &coord.x, &coord.y, &coord.z);
  
  int *x_ptr = &coord.x;
  
  printf("at pointer (index): %d - %p\n", *x_ptr, x_ptr);
  
  x_ptr++;
  
  printf("at pointer++: %d - %p\n", *x_ptr, x_ptr);
  
  x_ptr++;
  
  printf("at (pointer++)++ pointer: %d - %p\n", *x_ptr, x_ptr);
}
```

```
x, y, z: 24, 41, 87

x, y, z addresses: 0x7ffcca6f3c0c, 0x7ffcca6f3c10, 0x7ffcca6f3c14

at pointer (index): 24 - 0x7ffcca6f3c0c
at pointer++: 41 - 0x7ffcca6f3c10
at (pointer++)++: 87 - 0x7ffcca6f3c14
```

Granted this only works since we are using the same type of variables. Doing the same thing with interleaved types doesn't work.

```c
typedef struct Mixed {
  int x;
  int y;
  char *tst;
  char *sts;
  int z;
} mixed_t;

int main() {
  mixed_t mix = { x: 24, y: 41, tst: "A", sts: "B", z: 87 };
  
  printf("x, y, tst, sts, z: %d, %d, %s, %s, %d\n", mix.x, mix.y, mix.tst, mix.sts, mix.z);
  printf("x, y, tst, sts, z addresses:\n%p\n%p\n%p\n%p\n%p\n", &mix.x, &mix.y, &mix.tst, &mix.sts, &mix.z);
  
  int *index = &mix.x;
  
  printf("at index: %d - %p\n", *index, index);
  
  index++;
  
  printf("at index++: %d - %p\n", *index, index);
  
  index++;
  
  printf("at (index++)++ index: %d - %p\n", *index, index);
}
```

```
x, y, tst, sts, z: 24, 41, A, B, 87

x, y, tst, sts, z addresses:
0x7ffdf6cbc1b0
0x7ffdf6cbc1b4
0x7ffdf6cbc1b8
0x7ffdf6cbc1c0
0x7ffdf6cbc1c8

at index: 24 - 0x7ffdf6cbc1b0
at index++: 41 - 0x7ffdf6cbc1b4
at (index++)++ index: -980684792 - 0x7ffdf6cbc1b8
```