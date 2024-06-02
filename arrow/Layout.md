---
tags:
  - arrow
gardening: 🌱
reference:
  - https://arrow.apache.org/docs/format/Columnar.html
---
### Validity bitmaps
All array types, with the exception of union types (more on these later), utilize a dedicated memory buffer, known as the validity (or “null”) bitmap, to encode the nullness or non-nullness of each value slot.
`is_valid[j] -> bitmap[j / 8] & (1 << (j % 8))`

```
values = [0, 1, null, 2, null, 3]

bitmap
j mod 8   7  6  5  4  3  2  1  0
          0  0  1  0  1  0  1  1
```

