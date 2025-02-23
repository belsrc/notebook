---
tags:
  - language
  - c
gardening: 🌱
date: 2025-02-12
reference:
---
#### Array Length

```c
int len = sizeof(arr) / sizeof(arr[0]);
```

#### Move string pointer to end (`'\0'`)

```c
// str1 == char *str1
while (*str1) str1++;

// Though this seems less efficient then getting
// strlen of the string and treating like an array
```

