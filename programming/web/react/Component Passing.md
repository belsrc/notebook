---
tags:
  - react
  - web
gardening: 🌳
date: 2025-12-24
---
## Two Patterns for Passing Components

### 1. Passing Component Types (References)

Pass the component function itself, allowing the receiving component to instantiate it:

```typescript
type ButtonProps = {
  icon: React.ComponentType<{ size: number }>;
  label: string;
};

const Button = ({ icon: Icon, label }: ButtonProps) => {
  return (
    <button>
      <Icon size={16} />
      <span>{label}</span>
    </button>
  );
};

// Usage
<Button icon={ChevronIcon} label="Next" />
```

**Characteristics:**
- Receives uninstantiated component (function reference)
- Parent doesn't control props, receiver does
- Defers element creation to the receiving component
- More flexible for dynamic prop injection

### 2. Passing Component Elements (Instances)

Pass pre-rendered JSX elements:

```typescript
type CardProps = {
  header: React.ReactNode;
  footer: React.ReactNode;
  children: React.ReactNode;
};

const Card = ({ header, footer, children }: CardProps) => {
  return (
    <div className="card">
      <div className="card-header">{header}</div>
      <div className="card-body">{children}</div>
      <div className="card-footer">{footer}</div>
    </div>
  );
};

// Usage
<Card 
  header={<Title text="Overview" />}
  footer={<Button label="Save" />}
>
  <Content />
</Card>
```

**Characteristics:**
- Receives already instantiated elements
- Parent controls all props
- Elements are created in parent's render cycle
- Less flexible but more explicit

## Under the Hood

When you write `<Icon size={16} />`, React transforms it to:

```typescript
React.createElement(Icon, { size: 16 }, null)
```

This returns a plain object:

```typescript
{
  type: Icon,        // Reference to component function
  props: { size: 16 },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element')
}
```

This element object is just data. React doesn't execute the component function until reconciliation.

## Performance Implications

### Component Type Pattern

```typescript
// Icon instantiated during Card's render
<Card icon={UserIcon} />

// Inside Card:
const Card = ({ icon: Icon }) => {
  return (
    <div className="card">
      <Icon />
    </div>
  );
};
```

* Component instantiated during Card's render cycle
* Can be memoized separately from parent
* Better for conditional rendering (Icon not created if unused)
* Card controls when/if Icon renders

### Element Pattern (Explicit Prop)

```typescript
// Icon instantiated during parent's render
<Card icon={<UserIcon />} />

// Inside Card:
const Card = ({ icon }) => {
  return (
    <div className="card">
      {icon}
    </div>
  );
};
```

* Icon element created during parent's render cycle
* Icon re-renders whenever parent re-renders
* Even if Card is memoized, icon isn't independently memoizable
* Element is created every render (new object reference breaks memoization)

### Children Pattern (Standard Composition)

```typescript
// Icon instantiated during parent's render
<Card>
  <UserIcon />
</Card>

// Inside Card:
const Card = ({ children }) => {
  return (
    <div className="card">
      {children}
    </div>
  );
};
```

* **Identical performance to Element Pattern**
* Icon element created during parent's render cycle
* Icon re-renders whenever parent re-renders
* New element object each render breaks React.memo
* Most idiomatic React pattern for composition

## Type-Safe Generic Pattern

For maximum flexibility with type safety:

```typescript
type ContainerProps<P = {}> = {
  component: React.ComponentType<P>;
  componentProps: P;
  wrapper?: React.ComponentType<{ children: React.ReactNode }>;
};

const Container = <P,>({ 
  component: Component, 
  componentProps,
  wrapper: Wrapper 
}: ContainerProps<P>) => {
  const content = <Component {...componentProps} />;
  return Wrapper ? <Wrapper>{content}</Wrapper> : content;
};

// Usage with full type inference
<Container 
  component={UserProfile}
  componentProps={{ userId: "123", showBadge: true }}
  wrapper={ErrorBoundary}
/>
```

## Render Props Pattern

A variation passing functions that return elements:

```typescript
type DataFetcherProps<T> = {
  url: string;
  children: (data: T | null, loading: boolean) => React.ReactNode;
};

const DataFetcher = <T,>({ url, children }: DataFetcherProps<T>) => {
  const [data, setData] = React.useState<T | null>(null);
  const [loading, setLoading] = React.useState(true);
  
  React.useEffect(() => {
    fetch(url)
      .then(r => r.json())
      .then(d => { setData(d); setLoading(false); });
  }, [url]);
  
  return <>{children(data, loading)}</>;
};

// Usage
<DataFetcher<User> url="/api/user">
  {(user, loading) => loading ? <Spinner /> : <UserCard user={user} />}
</DataFetcher>
```
