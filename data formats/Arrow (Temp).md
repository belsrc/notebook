---
tags:
  - arrow
  - data
reference:
  - https://arrow.apache.org/docs/format/Columnar.html
  - https://arrow.apache.org/docs/format/Columnar.html#fixed-size-list-layout
  - https://arrow.apache.org/docs/cpp/api/datatype.html
---
## What is Arrow?

The Apache Arrow specification [defines](https://arrow.apache.org/docs/format/Columnar.html) a common, language-agnostic in-memory representation for columnar data. Since _every_ implementation of Apache Arrow uses the same memory layout, with only a bit of metadata about where in memory to look and what data type an array has, _any Arrow implementation can understand existing memory it hasn't created_. The Arrow memory format also supports zero-copy reads for lightning-fast data access without serialization overhead. Note

### But Why?

- Data adjacency for sequential access (scans)
- O(1) (constant-time) random access
- SIMD and vectorization-friendly
- Relocatable without “pointer swizzling”, allowing for true zero-copy access in shared memory

The Arrow columnar format provides analytical performance and data locality guarantees in exchange for comparatively more expensive mutation operations.

## Format

```js
[
  { lon: 1, lat: 0, alt: 10 },
  { lon: 2, lat: 9, alt: 11 },
  { lon: 3, lat: 8, alt: 12 },
  { lon: 4, lat: 7, alt: 13 },
  { lon: 5, lat: 6, alt: 14 },
]

// A standard row format would give you something like
const rowOne = { lon: 1, lat: 0, alt: 10 };
const rowTwo = { lon: 2, lat: 9, alt: 11 };

// Whereas a columnar would be something like
const lonColumn = [1, 2, 3, 4, 5];
const latColumn = [0, 9, 8, 7, 6];
const altColumn = [10, 11, 12, 13, 14];
```

![](../../images/arrow/arrow-column-layout.png)

## Layout

### Validity bitmaps

All array types, with the exception of union types (more on these later), utilize a dedicated memory buffer, known as the validity (or “null”) bitmap, to encode the nullness or non-nullness of each value slot.
`is_valid[j] -> bitmap[j / 8] & (1 << (j % 8))`

```
values = [0, 1, null, 2, null, 3]

bitmap
j mod 8   7  6  5  4  3  2  1  0
          0  0  1  0  1  0  1  1
```
