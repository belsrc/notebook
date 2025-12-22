---
tags:
  - design-pattern
  - comp-sci
gardening: 🌿
date: 2025-12-21
reference:
  - https://softwaredesignpatterns.azurewebsites.net/eBooks/Design%20Patterns%20Elements%20of%20Reusable%20Object-Oriented%20Software.pdf
  - https://refactoring.guru/design-patterns/singleton
---
## What & Why

You need exactly one instance of a class throughout your application's lifetime, with global access to that instance.

- Coordinate actions across the system (e.g., logging, configuration, database connections)
- Control access to shared resources (e.g., file systems, hardware interfaces)
- Ensure consistency (e.g., single cache manager, single event bus)

## Structure Diagram

```
┌─────────────────────────────────┐
│      Singleton                  │
├─────────────────────────────────┤
│ - instance: Singleton (static)  │
│ - constructor() (private)       │
├─────────────────────────────────┤
│ + getInstance(): Singleton      │
│ + operation(): void             │
└─────────────────────────────────┘
```

## Traditional Implementation

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection | null = null;
  private connectionId: string;
  private connectionTime: Date;

  // Private constructor prevents direct instantiation
  private constructor() {
    this.connectionId = Math.random().toString(36).substring(7);
    this.connectionTime = new Date();
    console.log(`Database connection ${this.connectionId} established`);
  }

  // Global access point - creates instance on first call
  public static getInstance(): DatabaseConnection {
    if (DatabaseConnection.instance === null) {
      DatabaseConnection.instance = new DatabaseConnection();
    }

    return DatabaseConnection.instance;
  }

  public query(sql: string): string {
    return `Executing on ${this.connectionId}: ${sql}`;
  }

  public getConnectionInfo(): string {
    return `Connection ${this.connectionId} established at ` +
           `${this.connectionTime.toISOString()}`;
  }
}

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();

console.log(db1 === db2); // true - same instance
console.log(db1.getConnectionInfo());
```

**Key Characteristics**:
- Private constructor
- Static instance variable
- Static `getInstance()` method
- Lazy initialization (created on first access)

## Modern Alternative

We achieve singleton behavior through:
1. **Module-level closure** - JavaScript modules are singletons by default
2. **Lazy initialization via closures**
3. **Immutability and pure functions**

```typescript
// Module-level state (private to this module)
type ConnectionState = {
  readonly connectionId: string;
  readonly connectionTime: Date;
  readonly queryCount: number;
};

let instance: ConnectionState | null = null;

// Lazy initialization function
// Function that returns the same reference
const initialize = (): ConnectionState => {
  if (instance === null) {
    instance = {
      connectionId: Math.random().toString(36).substring(7),
      connectionTime: new Date(),
      queryCount: 0
    };

    console.log(`Database connection ${instance.connectionId} established`);
  }

  return instance;
};

// Public API - all functions operate on the singleton state
export const databaseConnection = {
  getInstance: (): ConnectionState => initialize(),
  
  query: (sql: string): string => {
    const conn = initialize();

    // In real implementation, would update queryCount immutably
    return `Executing on ${conn.connectionId}: ${sql}`;
  },
  
  getConnectionInfo: (): string => {
    const conn = initialize();
    return `Connection ${conn.connectionId} established at ` +
           `${conn.connectionTime.toISOString()}`;
  }
};

// Usage
const conn1 = databaseConnection.getInstance();
const conn2 = databaseConnection.getInstance();
console.log(conn1 === conn2); // true - same reference
```

### Comparison: Traditional vs Modern

| Aspect        | Class-based         | Closure/Module       |
| ------------- | ------------------- | -------------------- |
| Encapsulation | Private constructor | Module/closure scope |
| Global Access | Static method       | Exported function    |
| Testability   | Difficult (global)  | Easier (dependency)  |
| Memory        | Single class inst.  | Single closure state |
| Immutability  | Mutable by default  | Can be immutable     |
| Thread Safety | Requires locking    | Requires locking     |
| Composability | Limited             | High (HOF)           |

### Problems with Traditional Singleton

1. **Global State** - Hidden dependencies, hard to test
2. **Violates Single Responsibility** - Manages both instance creation AND business logic
3. **Tight Coupling** - Classes directly depend on Singleton
4. **Testing Difficulty** - Can't easily mock or reset state

### Stackblitz Link

[Singleton Pattern](https://stackblitz.com/edit/vitejs-vite-wxuvptb2?file=src%2Fmain.ts)