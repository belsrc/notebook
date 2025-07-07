---
tags:
  - react
gardening: 🌿
date: 2025-06-30
reference:
  - https://www.epicreact.dev/memoization-and-react
  - https://github.com/facebook/react/blob/4c7036e807/packages/react-reconciler/src/ReactFiberHooks.new.js
  - https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberHooks.js
  - https://kentcdodds.com/blog/usememo-and-usecallback
---
React implements memoization through two phases: `mountMemo`/`mountCallback` (first render) and `updateMemo`/`updateCallback` (subsequent renders).

```ts
// Simplified version of React's actual useMemo implementation
// https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberHooks.js

type Hook = {
  memoizedState: any
  next: Hook | null
  // ... other properties
}

// Mount phase (first render)
function mountMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  const hook = mountWorkInProgressHook()
  const nextDeps = deps === undefined ? null : deps
  const nextValue = nextCreate()
  
  // Store ONLY the current value and dependencies
  hook.memoizedState = [nextValue, nextDeps]
  return nextValue
}

// Update phase (subsequent renders)
function updateMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  const hook = updateWorkInProgressHook()
  const nextDeps = deps === undefined ? null : deps
  const prevState = hook.memoizedState  // [prevValue, prevDeps]
  
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps: Array<mixed> | null = prevState[1]
      
      // Compare each dependency individually using Object.is
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0]  // Return cached value
      }
    }
  }
  
  // Dependencies changed or no previous state - recalculate
  const nextValue = nextCreate()
  hook.memoizedState = [nextValue, nextDeps]
  return nextValue
}

// useCallback is just useMemo for functions
function mountCallback<T>(
  callback: T,
  deps: Array<mixed> | void | null,
): T {
  const hook = mountWorkInProgressHook()
  const nextDeps = deps === undefined ? null : deps
  hook.memoizedState = [callback, nextDeps]
  return callback
}

function updateCallback<T>(
  callback: T,
  deps: Array<mixed> | void | null,
): T {
  const hook = updateWorkInProgressHook()
  const nextDeps = deps === undefined ? null : deps
  const prevState = hook.memoizedState
  
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps = prevState[1]
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0]  // Return cached function
      }
    }
  }
  
  hook.memoizedState = [callback, nextDeps]
  return callback
}

// Dependency comparison function
function areHookInputsEqual(
  nextDeps: Array<mixed>,
  prevDeps: Array<mixed> | null,
): boolean {
  if (prevDeps === null) {
    return false
  }
  
  // Compare each dependency using Object.is
  for (let i = 0; i < prevDeps.length && i < nextDeps.length; i++) {
    if (Object.is(nextDeps[i], prevDeps[i])) {
      continue
    }
    return false
  }
  return true
}
```

### What it does

```ts
// EXPENSIVE: For example only
const Component = ({ val }: { val: number }) => {
  const isAnswer = useMemo(() => val === 42, [val])
  
  // What React actually does internally:
  // 1. Call updateMemo()
  // 2. Retrieve hook.memoizedState: [prevResult, [prevVal]]
  // 3. Call areHookInputsEqual([val], [prevVal])
  //    - Loop through dependencies
  //    - Call Object.is(val, prevVal)
  // 4. If different:
  //    - Execute val === 42
  //    - Store [result, [val]] in hook.memoizedState
  // 5. Return result
  
  return <div>{isAnswer ? "Yes!" : "No"}</div>
}
```

## Why This Design?

### Performance Trade-offs

React chose this design because it satisfies the primary use case for memoizing in a React context. The benefits include:

- **Memory efficiency**: No growing cache that could cause memory leaks
- **Predictable behavior**: Cache invalidation is simple - only the current value matters
- **Integration with React's lifecycle**: React can discard cached values when components unmount or during development hot reloads

### Cache Busting

React explicitly states that useMemo and useCallback are subject to cache purging and should be treated as performance optimizations, not semantic guarantees. React may "forget" cached values to free memory for off-screen components.

## Key Differences from Standard Memoization

### 1. **Cache Size = 1**

React's memoization has a cache size of exactly 1 - it only stores the most recent value and dependencies. Unlike traditional memoization that maintains a map of all input → output pairs, React only remembers the last computation.

### 2. **Storage Mechanism**

React doesn't use a cache object or Map. Instead, each hook stores its state in a `memoizedState` property that contains a tuple: `[value, dependencies]`. The hook object's memoizedState property stores both the calculated value and corresponding dependencies.

### 3. **Dependency Comparison**

React compares each dependency individually using `Object.is` comparison rather than hashing the entire dependency array. The `areHookInputsEqual` function performs a shallow comparison of each dependency.

### 4. **Fiber Architecture Integration**

React's memoization is deeply integrated with the Fiber reconciler. Each hook is part of a linked list structure attached to fiber nodes, where each hook maintains its own memoized state.

### Example Behavior

```typescript
const [a, setA] = useState(1);
const [b, setB] = useState(2);

// First render: a=1, b=2
const result1 = useMemo(() => a + b, [a, b]); // Calculates: 3

// Second render: a=1, b=2 (same deps)
const result2 = useMemo(() => a + b, [a, b]); // Returns cached: 3

// Third render: a=2, b=2 (deps changed)
const result3 = useMemo(() => a + b, [a, b]); // Calculates: 4

// Fourth render: a=1, b=2 (back to original)
const result4 = useMemo(() => a + b, [a, b]); // Calculates: 3 (not cached!)
```

In standard memoization, `result4` would return the cached value from `result1`. In React's implementation, it recalculates because only the most recent computation (a=2, b=2 → 4) was cached.

## The Overhead Reality

Performance optimizations ALWAYS come with a cost but do NOT always come with a benefit. For simple calculations, the memoization machinery overhead includes:

1. **Hook management**: Creating/updating hook objects in the fiber linked list
2. **Memory allocation**: Storing `[result, dependencies]` in `memoizedState`
3. **Dependency comparison**: The `areHookInputsEqual` function call and loop
4. **Function call overhead**: `updateMemo` vs direct execution

## When NOT to Use useMemo

Avoid memoizing for:

- Simple comparisons (`val === 42`, `count > 0`)
- Basic arithmetic (`a + b`, `price * 1.1`)
- String operations (`name.toUpperCase()`)
- Boolean logic (`isVisible && hasPermission`)
- Simple object property access (`user.name`)

```ts
// Examples of when NOT to use useMemo
const BadExamples = ({ name, count, isVisible }: Props) => {
  // ❌ BAD: More expensive than direct calculation
  const isEven = useMemo(() => count % 2 === 0, [count])
  const hasName = useMemo(() => !!name, [name])
  const isLongName = useMemo(() => name.length > 10, [name])
  
  // ✅ GOOD: Direct calculations
  const isEven = count % 2 === 0
  const hasName = !!name
  const isLongName = name.length > 10
  
  return <div>...</div>
}

// When useMemo IS worth it
const GoodExamples = ({ items, searchTerm }: Props) => {
  // ✅ GOOD: Expensive calculation
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    ).sort((a, b) => a.name.localeCompare(b.name))
  }, [items, searchTerm])
  
  // ✅ GOOD: Creating objects for referential equality
  const expensiveConfig = useMemo(() => ({
    apiKey: process.env.API_KEY,
    retries: 3,
    timeout: 5000
  }), [])  // Empty deps - only create once
  
  return <ExpensiveChild config={expensiveConfig} items={filteredItems} />
}
```