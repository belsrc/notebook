**V8 Switch Statement Optimization Strategies**

V8 uses different strategies depending on switch characteristics:

1. **Jump Table (fastest - O(1))**

Requirements:
- Case values are integers (or can be coerced to integers)
- Values are dense (no large gaps)
- Reasonably small range (typically < 256 range)

```typescript
// ✅ Optimized to jump table
switch (statusCode) {
  case 200: return 'OK';
  case 201: return 'Created';
  case 204: return 'No Content';
  case 400: return 'Bad Request';
  case 404: return 'Not Found';
}
// Range: 200-404 = 204 slots (acceptable)

// ❌ NOT optimized to jump table (sparse)
switch (code) {
  case 1: return 'A';
  case 1000: return 'B';
  case 10000: return 'C';
}
// Range: 1-10000 = 9999 slots (too sparse, wastes memory)
```

2. **Binary Search (O(log n))**

Used when:
- More than ~6-8 cases
- Cases are ordered (or can be sorted)
- Jump table would be too sparse or too large

```typescript
// Binary search strategy
switch (value) {
  case 100: return 'A';
  case 500: return 'B';
  case 1000: return 'C';
  case 5000: return 'D';
  case 10000: return 'E';
}
```

3. **Linear Search (O(n))**

Used when:
- Small number of cases (< 6-8)
- String cases
- Cases cannot be ordered

```typescript
// ❌ Linear search - our formatError case
switch (error.type) {
  case 'FileNotFound': return '...';
  case 'ParseError': return '...';
  // ... more string cases
}
```

4. **Hash Table (O(1) average, but with overhead)**

Used when:
- Many string cases (>8-10)
- TurboFan compiler optimizes hot paths
- Creates inline cache for common cases