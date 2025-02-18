---
tags:
  - comp-sci
  - paradigm
gardening: 🌱
date: 2025-02-09
reference:
  - https://en.algorithmica.org/hpc/cpu-cache/alignment/
  - https://www.nic.uoregon.edu/~khuck/ts/acumem-report/manual_html/ch05s01.html
  - https://en.algorithmica.org/hpc/cpu-cache/cache-lines/
---
- Deeper dive into alignment and cache lines (links in refs)

```c
	typedef struct Coordinate {
		int x;
		int y;
		int z;
	} coordinate_t;
	
	typedef struct Human {
		char first_initial;
		int age;
		double height;
	} human_t;

  bool bool_var = true;
  char char_var = 'W';
  int int_var = 235;
  long long_var = 45353;
  float float_var = 232.452;
  double double_var = 23421.435;
  coordinate_t coord = {.x = 23, .y = 123, .z = 23 };
  human_t human = {.first_initial = 'B', .age = 42, .height = 70 };
  
  bool *p_bool = &bool_var;
  char *p_char = &char_var;
  int *p_int = &int_var;
  long *p_long = &long_var;
  float *p_float = &float_var;
  double *p_double = &double_var;
  coordinate_t *p_coord = &coord;
  human_t *p_human = &human;
  
  printf("size of bool var: %zu\n", sizeof(bool_var));
  printf("size of bool pointer: %zu\n", sizeof(p_bool));
  printf("alignment of bool: %zu\n", alignof(bool));
  printf("address of bool var: %p\n\n", &bool_var);
  
  printf("size of char var: %zu\n", sizeof(char_var));
  printf("size of char pointer: %zu\n", sizeof(p_char));
  printf("alignment of char: %zu\n", alignof(char));
  printf("address of char var: %p\n\n", &char_var);
  
  printf("size of int var: %zu\n", sizeof(int_var));
  printf("size of int pointer: %zu\n", sizeof(p_int));
  printf("alignment of int: %zu\n", alignof(int));
  printf("address of int var: %p\n\n", &int_var);
  
  printf("size of long var: %zu\n", sizeof(long_var));
  printf("size of long pointer: %zu\n", sizeof(p_long));
  printf("alignment of long: %zu\n", alignof(long));
  printf("address of long var: %p\n\n", &long_var);
  
  printf("size of float var: %zu\n", sizeof(float_var));
  printf("size of float pointer: %zu\n", sizeof(p_float));
  printf("alignment of float: %zu\n", alignof(float));
  printf("address of float var: %p\n\n", &float_var);
  
  printf("size of double var: %zu\n", sizeof(double_var));
  printf("size of double pointer: %zu\n", sizeof(p_double));
  printf("alignment of double: %zu\n", alignof(double));
  printf("address of double var: %p\n\n", &double_var);
  
  printf("size of coord struct var: %zu\n", sizeof(coord));
  printf("size of coord struct pointer: %zu\n", sizeof(p_coord));
  printf("alignment of coord struct: %zu\n", alignof(coordinate_t));
  printf("address of coord struct: %p\n\n", &coord);
  
  printf("size of human struct var: %zu\n", sizeof(human));
  printf("size of human struct pointer: %zu\n", sizeof(p_human));
  printf("alignment of human struct: %zu\n", alignof(human_t));
  printf("address of human struct: %p\n\n", &human);
  
  printf("size of size_t: %zu\n\n", sizeof(size_t));
```

```
size of bool var: 1
size of bool pointer: 8
alignment of bool: 1
address of bool var: 0x7ffda01ee1eb

size of char var: 1
size of char pointer: 8
alignment of char: 1
address of char var: 0x7ffda01ee1ea

size of int var: 4
size of int pointer: 8
alignment of int: 4
address of int var: 0x7ffda01ee1e4

size of long var: 8
size of long pointer: 8
alignment of long: 8
address of long var: 0x7ffda01ee1d8

size of float var: 4
size of float pointer: 8
alignment of float: 4
address of float var: 0x7ffda01ee1d4

size of double var: 8
size of double pointer: 8
alignment of double: 8
address of double var: 0x7ffda01ee1c8

size of coord struct var: 12
size of coord struct pointer: 8
alignment of coord struct: 4
address of coord struct: 0x7ffda01ee1b8

size of human struct var: 16
size of human struct pointer: 8
alignment of human struct: 8
address of human struct: 0x7ffda01ee1a8

size of size_t: 8
```