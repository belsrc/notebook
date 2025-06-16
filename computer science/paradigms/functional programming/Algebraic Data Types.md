---
tags:
  - paradigm
  - functional
gardening: 🌱
date: 2025-05-23
reference:
  - https://en.wikipedia.org/wiki/Algebraic_data_type
  - https://cs3110.github.io/textbook/chapters/data/algebraic_data_types.html
  - https://en.wikibooks.org/wiki/Introduction_to_Programming_Languages/Algebraic_Data_Types
  - https://dev.to/gcanti/functional-design-algebraic-data-types-36kf
  - https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/
---
Algebraic Data Types (ADTs) are one of the most powerful and elegant concepts in programming. They provide a mathematical foundation for modeling data, resulting in more robust, maintainable, and expressive code. Although ADTs are traditionally associated with functional programming languages such as Haskell or ML, they have also been adopted in mainstream languages, including TypeScript, through libraries like `true-myth`.

## What Are Algebraic Data Types?

Algebraic Data Types are composite types created by combining other types using two fundamental operations: **sum** (union) and **product** (combination). The term "algebraic" is derived from the mathematical properties of these operations, which resemble addition and multiplication in algebra.

### The Two Building Blocks

**Product Types** refer to a combination of multiple values, similar to a struct, record, or tuple that encapsulates all its constituent parts at once. In TypeScript, this might look like:

```typescript
type User = {
  id: number;
  name: string;
  email: string;
}
```

**Sum Types** signify a selection among alternatives, meaning you have only one of several possible values. TypeScript's union types serve this purpose:

```typescript
type Status = 'loading' | 'success' | 'error';
```

The real strength of these concepts comes into play when you combine them, allowing you to create rich and expressive data models that prevent illegal states from being representable.

## Why Algebraic Data Types Matter

### 1. Type Safety and Correctness

Algebraic Data Types (ADTs) help eliminate many classes of runtime errors by ensuring that invalid states cannot be represented. Instead of relying on documentation or performing runtime checks, the type system enforces correctness at compile time.

### 2. Exhaustive Pattern Matching

When you handle all possible cases of a sum type, the compiler can verify that your implementation is complete. This means that if you add new variants, the compiler will identify every location in your code that needs to be updated.

### 3. Self-Documenting Code

ADTs make the possible states and transitions of your system explicit in the type signatures, serving as living documentation that cannot become outdated.

### 4. Reduced Complexity

By making illegal states unrepresentable, ADTs reduce the cognitive load associated with reasoning about your code. You no longer need to handle edge cases that cannot exist.

## Essential ADTs: Maybe and Result

The two most fundamental and useful algebraic data types are `Maybe` (also called `Option`) and `Result` (also called `Either`). Let's explore these using the `true-myth` library.

### Maybe: Handling Nullable Values

The `Maybe` type represents values that might not exist, eliminating the need for `null` or `undefined` checks scattered throughout your code.

```typescript
import { Maybe } from 'true-myth';

// Creating Maybe values
const someValue = Maybe.of(42);
const noValue = Maybe.nothing<number>();

// Safe operations without null checks
function findUserById(id: number): Maybe<User> {
  const user = database.find(u => u.id === id);
  return Maybe.of(user);
}

// Chaining operations safely - Whether `Some` or `Nothing`
// A `Just` value will go through each processing step
// A `Nothing` value will simply "slide" down to the end
const userEmail = findUserById(123)
  .map(user => user.email)
  .map(email => email.toLowerCase())
  .unwrapOr('no-email@example.com');
```

### Result: Error Handling Without Exceptions

The `Result` type represents operations that can either succeed with a value or fail with an error, providing a functional alternative to exception handling.

```typescript
import { Result } from 'true-myth';

function parseAge(input: string): Result<number, string> {
  const age = parseInt(input, 10);
  
  if (isNaN(age)) {
    return Result.err('Invalid number format');
  }
  
  if (age < 0 || age > 150) {
    return Result.err('Age must be between 0 and 150');
  }
  
  return Result.ok(age);
}

// Composing operations that might fail
function processUserInput(
	ageInput: string,
	nameInput: string
): Result<User, string> {
  return parseAge(ageInput)
    .andThen(age => {
      if (nameInput.trim().length === 0) {
        return Result.err('Name cannot be empty');
      }
      
      return Result.ok({
        age,
        name: nameInput.trim(),
        id: generateId()
      });
    });
}

// Handling the result
const userResult = processUserInput('25', 'Alice');

userResult.match({
  Ok: user => console.log(`Created user: ${user.name}`),
  Err: error => console.error(`Failed to create user: ${error}`)
});
```

## Advanced Patterns and Techniques

### Combining Maybe and Result

Real-world applications often need to combine these patterns. For instance, a database query might return no results (Maybe) or fail entirely (Result):

```typescript
function findUserSafely(id: number): Result<Maybe<User>, DatabaseError> {
  try {
    const user = database.findById(id);
    return Result.ok(Maybe.of(user));
  } catch (error) {
    return Result.err(new DatabaseError(error.message));
  }
}

// Usage with nested pattern matching
findUserSafely(123).match({
  Ok: maybeUser => maybeUser.match({
    Just: user => console.log(`Found user: ${user.name}`),
    Nothing: () => console.log('User not found')
  }),
  Err: error => console.error(`Database error: ${error.message}`)
});
```

### Building Domain-Specific ADTs

You can create custom algebraic data types for your specific domain:

```typescript
// Representing application state
type AppState = 
  | { type: 'loading' }
  | { type: 'loaded'; data: User[]; lastUpdated: Date }
  | { type: 'error'; error: string; retryCount: number };

function handleStateChange(state: AppState): void {
  switch (state.type) {
    case 'loading':
      showSpinner();
      break;
    case 'loaded':
      hideSpinner();
      renderUsers(state.data);
      updateTimestamp(state.lastUpdated);
      break;
    case 'error':
      hideSpinner();
      showError(state.error);
      if (state.retryCount < 3) {
        scheduleRetry();
      }
      break;
  }
}
```

### Functional Composition with ADTs

ADTs shine when combined with functional programming techniques:

```typescript
import { Maybe, Result } from 'true-myth';

// Higher-order function for lifting regular functions into Result context
const liftResult = <A, B>(fn: (a: A) => B) =>
	(result: Result<A, string>): Result<B, string> =>
	  result.map(fn);

// Utility functions for common operations
const validateEmail = (email: string): Result<string, string> =>
  email.includes('@') 
    ? Result.ok(email.toLowerCase())
    : Result.err('Invalid email format');

const validateAge = (age: string): Result<number, string> => {
  const parsed = parseInt(age, 10);
  return isNaN(parsed) || parsed < 0 || parsed > 150
    ? Result.err('Age must be a number between 0 and 150')
    : Result.ok(parsed);
};

const validateName = (name: string): Result<string, string> =>
  name.length < 2
    ? Result.err('Name must be at least 2 characters')
    : Result.ok(name);

// Create composed transformation pipelines
const processEmail = compose(
  liftResult(toLowerCase),
  liftResult(trim)
);

const processName = compose(
  liftResult(trim),
  validateName
);

// Custom applicative-style combiner
function combine<A, B, C, D, E>(
  resultA: Result<A, E>,
  resultB: Result<B, E>,
  resultC: Result<C, E>,
  combiner: (a: A, b: B, c: C) => D
): Result<D, E> {
  return resultA.andThen(a =>
    resultB.andThen(b =>
      resultC.map(c => combiner(a, b, c))
    )
  );
}

// Clean, declarative user creation
function createUser(input: UserInput)
: Result<TimestampedUser, string> {
  const nameResult = processName(input.name);
  const emailResult = processEmail(validateEmail(input.email));
  const ageResult = validateAge(input.age);

  return combine(
    nameResult,
    emailResult,
    ageResult,
    (name, email, age) => ({
      id: crypto.randomUUID(),
      name,
      email,
      age
    })
  ).map(addTimestamp);
}

const userResult = createUser(userData);

userResult.match({
  Ok: user => console.log(`Created user: ${user.name} at ${user.createdAt}`),
  Err: error => console.error(`Validation failed: ${error}`)
});
```

## When to Use Algebraic Data Types

### Ideal Scenarios

**Error-Prone Operations**: Network requests, file input/output, parsing, and validation are ideal candidates for using `Result` types.

**Optional Data**: Instead of using nullable types, utilize `Maybe` to explicitly and safely handle optional values.

**State Machines**: Employ sum types to represent the distinct states of your application, preventing invalid transitions.

**Domain Modeling**: Use a well-defined type system to accurately represent complex business rules and constraints.

### Implementation Considerations

**Team Familiarity**: Adopting Abstract Data Types (ADTs) requires a shift in mindset. Ensure that your team understands the concepts and patterns associated with ADTs.

**Library Ecosystem**: Opt for libraries like `true-myth` that offer comprehensive implementations of ADTs with user-friendly ergonomics.

**Gradual Adoption**: There’s no need to refactor everything at once. Begin with areas that are prone to errors and gradually expand your use of ADTs over time.

**Performance**: Modern JavaScript engines optimize the use of ADT patterns effectively, but be cautious of creating excessive objects in performance-critical sections of your code.

## Best Practices and Common Patterns

### 1. Make Illegal States Unrepresentable

Design your types so that invalid combinations cannot exist:

```typescript
// Instead of this
type UserData = {
  id?: number;
  name?: string;
  isLoading: boolean;
  error?: string;
}

// Use this
type UserState = 
  | { type: 'loading' }
  | { type: 'loaded'; user: { id: number; name: string } }
  | { type: 'error'; message: string };
```

### 2. Prefer Transformation Over Mutation

Use `map`, `andThen`, and other combinators to transform values rather than mutating state:

```typescript
const processUser = (input: string) =>
  parseJSON(input)
    .andThen(validateUser)
    .map(normalizeUser)
    .map(enrichUser);
```

### 3. Handle All Cases Explicitly

Always use exhaustive pattern matching to ensure you handle every possible case:

```typescript
function handleResult<T, E>(result: Result<T, E>): void {
  result.match({
    Ok: value => {/* handle success */},
    Err: error => {/* handle error */}
  });
}
```

## Conclusion

Algebraic Data Types represent a fundamental shift toward more reliable, expressive programming. By making invalid states unrepresentable and providing principled ways to handle uncertainty and failure, ADTs help create software that is easier to reason about, maintain, and extend.

The `true-myth` library brings these powerful concepts to TypeScript in an ergonomic way, allowing you to incrementally adopt ADT patterns in existing codebases. Start with `Maybe` for optional values and `Result` for error handling, then gradually expand to more sophisticated domain modeling as your team becomes comfortable with the patterns.

The investment in learning ADTs pays dividends in reduced bugs, clearer code, and more robust applications. As software systems continue to grow in complexity, the mathematical rigor and expressiveness of algebraic data types become increasingly valuable tools in the programmer's toolkit.