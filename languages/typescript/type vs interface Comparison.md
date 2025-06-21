In TypeScript, `type` and `interface` serve similar purposes but have important semantic and behavioral differences that impact correctness.

## Fundamental Differences

**Type aliases** (`type`) create a new name for an existing type, while **interfaces** define the shape of an object and support declaration merging.

```typescript
// Type alias - creates an alias for an existing type
type UserID = string;
type Point = { x: number; y: number };

// Interface - declares a new object shape
interface IPoint {
  x: number;
  y: number;
}
```

### 1. Declaration Merging

Interfaces support declaration merging, which can lead to unexpected behavior:

```typescript
interface User {
  name: string;
}

interface User {  // This merges with the above
  age: number;
}

// User now has both name and age
const user: User = { name: 'Alice', age: 30 }; // Valid

// With types, this would cause a compilation error:
type UserType = { name: string };
// type UserType = { age: number }; // Error: Duplicate identifier
```
[Playground](https://www.typescriptlang.org/play/#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChnIhwC2EAXMumFKAOYDcuAvrrqJLIihltgQPT9kAFQAWwdMlJRaESQHdgYUcmUo4AIwD2ANwj5kcWRRABXYhuiMWuQWkxRCW+clFxJ25YRLqQAE0NZXAQtECpkUwcKHkcAXhxvUgoAIgBBABtgJGSAGkDyZABmAAZkJnpkOwA1OEy-VjsAdSUVMABPAAc5POUJZHktU3SAhDhI9WQQ4g7gdLgwYFDkaCgtKDJcdq77aGFOlHi+IiTKajoyxi3uBz3tw-yTc0tHcsqhAFEoVfXkABFTDqZUaQZDAPwQcDAGDAaBAA)

### 2. Union and Intersection Types

Types can represent unions and intersections directly, interfaces cannot:

```typescript
type Status = 'loading' | 'success' | 'error';
type Result<T> = { data: T } | { error: string };

// Interfaces require intersection for composition
type Combined = User & Settings;

interface ICombined extends User, Settings {} // Equivalent but more verbose
```
[Playground](https://www.typescriptlang.org/play/#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChnIhwC2EAXMumFKAOYDcuAvrrqJLIigMoRhh10OfJQAWAewDuABSjiYwADblkAI3HjlcEIxa4wATwAOPMHDABXIQF5kAIkXi4AEzp3kAH3voLCJOnR3LztoOSg7RkMTZAAlCB9FMAAeABUAPmRbbGRnczgKFOQmTxxkUPEoCioaEFoixlwAekbkAElwaHh-ZCgIAEcLYF7kdmhMBAFxEGQYCuQEcWIjcXRgSZB9YxQAYUXVUAhnTLRMKGQAMmRefkEG0c4kNt3ifZBDsoAPSBBnIQxoAA0Vz4AlqQmwxWayAAogNgAA3ODKcBqCxgZDECooeHQdSYVgLEBUeZ7A7OCjPV7vLIiIikCgAclm4gZAJE6AkMjkCmUFHgikwbKYDUJxOACxeZIorUpZOOeAIdJUTI0rPZnNk8iUKn5guY9CAA)

### 3. Primitive and Computed Types

Types can alias primitives and computed types:

```typescript
type ID = string | number;
type EventHandler<T> = (event: T) => void;
type Keys<T> = keyof T;
type Partial<T> = { [P in keyof T]?: T[P] };

// Interfaces cannot do this - they're only for object shapes
```

## Type Theory

### Type Identity vs Type Equivalence

From a type theory perspective, `type` creates **type aliases** [\(🞵)](https://en.wikipedia.org/wiki/Type_aliasing) while `interface` creates **new type declarations** [\(🞵)](https://en.wikipedia.org/wiki/Declaration_(computer_programming)):

```typescript
type UserId = string;
type ProductId = string;

interface IUserId { value: string; }
interface IProductId { value: string; }

// These are the SAME type (both are string)
const userId: UserId = 'user123';
const productId: ProductId = userId; // No error - they're both just string

// These are DIFFERENT types (structural equivalence but nominal distinction)
const userObj: IUserId = { value: 'user123' };
const productObj: IProductId = userObj; // No error in TS, but semantically different

// The critical difference:
type AliasComparison = UserId extends ProductId ? true : false; // true
type InterfaceComparison = IUserId extends IProductId ? true : false; // true (structural)
```
[Playground](https://www.typescriptlang.org/play/#code/C4TwDgpgBAqgzhATgSQCZQLxTsRBLAOwHMBuAKFEigAVEB7VAVwGNg1NtdDSyzDgkAMwCGzaMnhJ2AbygA3YQBtGEAFyd8xElAC+fAgMQixUZLQYs26WQuVqN3bXrIB6F1AAqACwgIowxGhgHygAZQBBAFkAUShKaAAKACM6YP9Ah2IASjJmOgIcKEYEFFR1SVKOAHJipABGACYAZiryPILgKDB6JlY0dXNeqw5a0u03KAA5OigkekQoAFo4nxAqjJS0gCtizpxNIl4J719oAOgAEWQAMWvogCVoyY848F8oBP3LRkQlWYBHRh4WwQAgmJKMToEOgAW0If1QeBwhFYeHyOXahVGAHkklt1BISuwsDYlCp1DUSo0Wro2vlCt0LKxcfjTINLMSiiUWeN3NNZoh5lBCJ5QgAaKAQvYQGHCAx4ZhKRQgKCIwSCJCg4BHdwnKDMfDABUIvDqzVgtQUN5QcKKPDCOAAYVhYACSPyHAq7AgAA8BARUHAaD0OegAPxxRAqKDqESKBC8yMqK1UZAGISiCDOmGu-BwD1YQlSdC+-2BtkhvrhpPQWNKBNQCa4aOfZusH5KLJAA)

### Type Constructor Semantics

Types are **type-level functions** [\(🞶)](https://stackoverflow.com/questions/24481113/what-are-some-examples-of-type-level-programming#answer-24481747), interfaces are **type declarations**:

```typescript
// Type as a type constructor - this is a function at the type level
type Maybe<T> = T | null;
type Either<L, R> = { left: L } | { right: R };
type State<S, A> = (state: S) => readonly [A, S];

// Interface cannot express these type constructors directly
// This doesn't work:
// interface IMaybe<T> = T | null; // Syntax error

// You'd need:
interface IMaybe<T> {
  value: T | null;
}
// But this changes the runtime representation!
```

## Correctness Considerations

### 1. Exact Type Definitions

Types are more precise about what they represent:

```typescript
// Clear that this is an alias for a specific structure
type ApiResponse<T> = {
  readonly data: T;
  readonly status: number;
  readonly timestamp: number;
};

// Less clear - could be extended elsewhere
interface IApiResponse<T> {
  readonly data: T;
  readonly status: number;
  readonly timestamp: number;
}
```

### 2. Exact vs Inexact Types

```typescript
type ExactUser = {
  readonly name: string;
  readonly id: number;
};

interface IUser {
  readonly name: string;
  readonly id: number;
}

// Type alias is "closed" - it refers to exactly this shape
const processExactUser = (user: ExactUser) => {
  // TypeScript knows this has exactly name and id
  const keys = Object.keys(user); // string[]
  return user;
};

// Interface might have additional properties due to declaration merging
const processIUser = (user: IUser) => {
  // TypeScript cannot guarantee this only has name and id
  const keys = Object.keys(user); // string[]
  return user;
};

// Somewhere else in the codebase:
declare module './types' {
  interface IUser {
    email?: string; // This changes ALL IUser references retroactively
  }
}
```
[Playground](https://www.typescriptlang.org/play/#code/C4TwDgpgBAogHgQwMbAKoGcICcoF4oDeAUFFFhAgCYD2AdgDYhS0IC2EAXFOsFgJa0A5gG4SZCjQZM+lLrQCurAEbZRAX1FEBwbADNk0AJIZshMeSp1GzNp268BI8xKvTZzRSqzqiRAPR+UAAq4NAI9HwI6FB80QBESPTUmJRxUAC0McDiutjRwNRQEIgo1sAAFrHc5QiQREh0PFBgWNRIEOjo8MhomDj4ABTyfVzdKCZYAJR4AHxmpAHBoQDKSPxg2QDWtNQA7vmV0TXRxT3WLOxQCLSUMZRiDbRNmxAg0fgA8koAVhAoAHQvN5DPqTYRQRY8fhCADaAF1nMB5FhaFBhqoiBpfItDLQdFh9O0oKw+IJytkagA3MKUSh8YB8OjhZqtSBYBkdKCUeTQApcv70BBYBAMujE7CCRz1RrZFptDroYx9PBQEHYLhK7DTXBzYgLQIhSCrdbZJDXHbZQTyIXXHS8w5QVxQY42S7XW4yB4yqBA95QL6-AG+tVTcGQhywhGkchIlFovrqTSLZbUdi7crYaAQeiYGKoirQBqUCBKKKcfx+YuJIXQVjUbn0aBxf5+UCQdBpPV5-GEowTeakK6CCAAfjknlUEINDqQNSEnIAggAZJdQTU4ci5ci0drRGOtHp8amMMRqCtqIA)

## Codebase Considerations

### 1. Data Structures as Values

```typescript
// Represent data as immutable values
type Vec3 = readonly [number, number, number];
type Matrix4x4 = readonly [
  readonly [number, number, number, number],
  readonly [number, number, number, number],
  readonly [number, number, number, number],
  readonly [number, number, number, number]
];

// Functions operating on these types
const addVec3 = (a: Vec3, b: Vec3): Vec3 =>
  [a[0] + b[0], a[1] + b[1], a[2] + b[2]];

const dotProduct = (a: Vec3, b: Vec3): number =>
  a[0] * b[0] + a[1] * b[1] + a[2] * b[2];
```

### 2. Higher-Order Function Types

```typescript
type Fold<T, R> = (acc: R, current: T) => R;
type Unfold<T, R> = (seed: T) => readonly [R, T] | null;

const reduce = <T, R>(arr: readonly T[], initial: R, folder: Fold<T, R>): R =>
  arr.reduce(folder, initial);

const unfold = <T, R>(seed: T, generator: Unfold<T, R>): readonly R[] => {
  const result: R[] = [];
  let current = seed;
  
  while (true) {
    const next = generator(current);
    if (next === null) break;
    
    const [value, nextSeed] = next;
    result.push(value);
    current = nextSeed;
  }
  
  return result;
};
```

### 3. Tagged Unions (Sum types)

```typescript
type Result<T, E = Error> = 
  | { readonly tag: 'ok'; readonly value: T }
  | { readonly tag: 'error'; readonly error: E };

const ok = <T>(value: T): Result<T> => ({ tag: 'ok', value });
const error = <E>(error: E): Result<never, E> => ({ tag: 'error', error });

const map = <T, U, E>(
  result: Result<T, E>, 
  fn: (value: T) => U
): Result<U, E> => 
  result.tag === 'ok' ? ok(fn(result.value)) : result;

const flatMap = <T, U, E>(
  result: Result<T, E>,
  fn: (value: T) => Result<U, E>
): Result<U, E> =>
  result.tag === 'ok' ? fn(result.value) : result;
```

### 4. Type-Level Computation Correctness

```typescript
// Conditional types work more predictably with type aliases
type NonNull<T> = T extends null | undefined ? never : T;
type ElementType<T> = T extends readonly (infer U)[] ? U : never;

// These computations are deterministic with types
type Result1 = NonNull<string | null>; // string
type Result2 = ElementType<readonly number[]>; // number

// With interfaces, you lose this computational power
interface INonNullable<T> {
  // Cannot express conditional logic
  value: T; // This doesn't eliminate null/undefined
}
```
[Playground](https://www.typescriptlang.org/play/#code/PTAEGEHsDsBMEsAu8YEMA2pEE8AOBTAZ1AHdIAnAa1AFsL9Rdz8EBjRVAI3W1KQAsseBhnipCRAFA4CoAHIw5AV3ToAPABUAfKAC8oDaHwAPRPjjFoKzAB9QSuPgBm8aC1AB+UG4Bu+cqAAXAYA3NLCoACi6Pg05ogawpo6+oYmZhagzKiwMDygABSuTv6gAKoAlADaALqe5UHe+H7kYZIgBvxEDKyQNLhKHMgwxKjMoLD4ZuQ0rvCEyKx8iIIyUmugAEpEKogAjHryitZqC+SuAOagdlaqWiGgHWeX4bLbhLsATIfRsfGJBDU2Vy0HyVhonH8tXujzA4Mh5Ek7TAAHUBKBXNMnKhWEQADSgbCQJSgdCQCRYfjzUC9fqDVDDaAYRiQEj+SSY-zY3GgACSCmgylUXBiyVAAG9JKBYRBUNBoJBEEZjEwiMRenAkCgmZgyRd4KwpaAfBglPhghoHh0NFTiLkiNAAORK-DoeCzJlmbzWYAOSYuNywSQAXyAA)

### 5. Functional Composition

Types compose better with functional patterns:

```typescript
type Predicate<T> = (value: T) => boolean;
type Transformer<T, U> = (value: T) => U;
type AsyncTransformer<T, U> = (value: T) => Promise<U>;

// Compose types functionally
type Pipeline<T> = {
  filter: (pred: Predicate<T>) => Pipeline<T>;
  map: <U>(transform: Transformer<T, U>) => Pipeline<U>;
  collect: () => T[];
};
```

### 6. Higher-Kinded Type Approximation

Types better support higher-kinded type patterns:

```typescript
// Type-level computation
type Apply<F, T> = F extends (arg: any) => infer R ? R : never;
type Compose<F, G> = <T>(x: T) => Apply<F, Apply<G, T>>;

// Functor-like pattern
type Functor<F> = {
  map: <A, B>(fa: F, f: (a: A) => B) => F; // F is a type constructor
};

// This kind of abstraction is harder with interfaces
```

### 7. Branded Types for Compile-Time Safety

[\(Branded types🞵)](https://blog.logrocket.com/leveraging-typescript-branded-types-stronger-type-checks/)

```typescript
// Branded types - the type parameter doesn't appear in the runtime value
type Brand<T, B> = T & { readonly __brand: B };
type Validated<T> = Brand<T, 'validated'>;
type Sanitized<T> = Brand<T, 'sanitized'>;

type Email = Validated<Sanitized<string>>;

const validateEmail = (input: string): Email | null => {
  // validation logic
  return input.includes('@') ? (input as Email) : null;
};

const sendEmail = (email: Email) => {
  // Guaranteed to be validated and sanitized
  console.log(`Sending to: ${email}`);
};

// This phantom type pattern is impossible with interfaces
// because interfaces always represent runtime structure
```

## Reasons to Prefer `type`

### 1. Referential Transparency

Types are referentially transparent - `type A = B` means A and B are interchangeable everywhere:

```typescript
type UserId = string;
// Wherever you see UserId, you can mentally substitute string

interface IUserId { value: string; }
// IUserId is a new type declaration, not substitutable
```

### 2. Compositionality

Types compose algebraically [\(🞶¹)](https://en.wikipedia.org/wiki/Algebra_of_sets#Fundamental_properties_of_set_algebra) [\(🞶²)](https://library.fiveable.me/lists/set-identities):

```typescript
type Sum<A, B> = A | B;            // Sum type (coproduct)
type Product<A, B> = A & B;        // Product type
type Function<A, B> = (a: A) => B; // Function type (exponential)

// These follow algebraic laws (Set theory):
// A | never ≡ A -> Union identity -> A ∪ ∅ ≡ A
// A & unknown ≡ A -> Intersection identity -> A ∩ 𝑈 ≡ A
// (A | B) & C ≡ (A & C) | (B & C) -> Distributive property
```

### 3. Curry-Howard Correspondence

Types correspond more directly to logical propositions [\(🞶)](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence):

```typescript
// Type as proposition: "if P then Q"
type Implies<P, Q> = P extends true ? Q : true;

// Type as proof term
type Modus_Ponens<P, Q> = 
  P extends true 
    ? (Implies<P, Q> extends true ? Q : never)
    : never;
```

## Recommendation

### Use `type` when you want:

- **True type aliases** (referential transparency)
- **Type-level computation** (conditional types, mapped types)
- **Algebraic composition** (unions, intersections)
- **Phantom (Branded) types** for compile-time guarantees
- **Exact type definitions** (no declaration merging)
- **Functional programming patterns**
- **Maximum correctness guarantees**

### Use `interface` only when you specifically need:

- **Open/extensible object types** (intentional declaration merging)
- **Class implementation contracts** (implements clause)
- **Legacy compatibility** with existing interface-based APIs

## Conclusion

The fundamental difference is that `type` operates at the **type level** (meta-programming), while `interface` operates at the **value level** (object shape description). This distinction provides `type` with superior semantic control and correctness guarantees.