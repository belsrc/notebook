---
tags:
  - arrow
  - notes
---
### What is Arrow?

The Apache Arrow specification [defines](https://arrow.apache.org/docs/format/Columnar.html) a common, language-agnostic in-memory representation for columnar data. Since _every_ implementation of Apache Arrow uses the same memory layout, with only a bit of metadata about where in memory to look and what data type an array has, _any Arrow implementation can understand existing memory it hasn't created_. The Arrow memory format also supports zero-copy reads for lightning-fast data access without serialization overhead. Note

---
### But Why?

- Data adjacency for sequential access (scans)
    
- O(1) (constant-time) random access
    
- SIMD and vectorization-friendly
    
- Relocatable without “pointer swizzling”, allowing for true zero-copy access in shared memory


The Arrow columnar format provides analytical performance and data locality guarantees in exchange for comparatively more expensive mutation operations.