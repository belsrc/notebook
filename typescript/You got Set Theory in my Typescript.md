---
tags:
  - notes
  - typescript
  - set-theory
---
## Element

${\in}$ equals an `element of`.

> $3~{\in}~A$

$3$ is an element of set $A$

```ts
namespace Element {
  type A = number;

  // 3 ∈ A
  const e: A = 3;
}
```

## Not an Element

${\notin}$ equals `not an element of`.

> $4~{\notin}~B$

$4$ is not an element of set $B$

```ts
namespace NotElement {
  type B = string;

  // 4 ∉ B
  const e: B = 3; // TS Error
}
```

## Subset

$\subseteq$ equals a `subset`

> $A \subseteq B$

$A$ is a subset of $B$ if every element of $A$ is contained in $B$ and $n(A) ≤ n(B)$

> $A = \lbrace1,2,3\rbrace$

> $B = \lbrace1,2,3\rbrace$

> $A \subseteq B$

> $X = \lbrace1,2\rbrace$

> $Y = \lbrace1,2,3\rbrace$

> $X \subseteq Y$

```ts
namespace Subset {
  type A = 1 | 2 | 3;
  type B = 1 | 2 | 3;

  const a1: A = 1;
  const a2: A = 2;
  const a3: A = 3;
  const b1: B = 1;
  const b2: B = 2;
  const b3: B = 3;
}
```

![image](https://user-images.githubusercontent.com/1832432/193629476-85d4d5c6-3591-44b1-922e-cc0cef639357.png)


## Proper Subset

$\subset$ equals a proper subset

$A$ is a proper subset of $B$ if every element of $A$ is contained in $B$ and $n(A) < n(B)$

> $A = \lbrace1,2\rbrace$

> $B = \lbrace1,2,3\rbrace$

> $A \subset B$

```ts
namespace ProperSubset {
  type A = 1 | 2;
  type B = 1 | 2 | 3;

  const a1: A = 1;
  const a2: A = 2;
  const b1: B = 1;
  const b2: B = 2;

  // @ts-expect-error Type '3' is not assignable to type 'A'
  const a3: A = 3;
}
```

![image](https://user-images.githubusercontent.com/1832432/193629546-58358ad5-df9c-41c8-8082-e53d663e1305.png)


## Equality

If $A \subseteq B$ and $B \subseteq A$ then $A = B$

![image](https://user-images.githubusercontent.com/1832432/193629495-9abec5ed-ac29-483c-b2b6-8ad6f829de5a.png)


## Not a Subset

$\not\subset$ equals `not a subset`

$A \not\subset B$ means $A$ is not a subset of $B$

## Union

$\cup$ equals `union`

$A \cup B$ is the union of set $A$ and set $B$

Unions of two sets is the set containing those 2 sets, so $A \cup B$ is the type containing all values of type $A$ and all values of type $B$

```ts
namespace Unions {
  type FirstThree = 'a' | 'b' | 'c';
  type ThreeToSix = 'c' | 'd' | 'e' | 'f';

  type FirstSixLetters = FirstThree | ThreeToSix;
  // "a" | "b" | "c" | "d" | "e" | "f"
}
```
Any duplicate elements in the set are removed.

![image](https://user-images.githubusercontent.com/1832432/193633760-eded8445-8621-4e89-8743-5ebca967a421.png)


## Intersection

$\cap$ equals the intersection

$A \cap B$ is the intersection of set $A$ and set $B$

Intersecting a set $A$ with a set $B$ means extracting the part of $A$ that also belongs to $B$. In other words, elements in common to both

```ts
namespace Intersections {
  type Evens = 2 | 4 | 6 | 8 | 10 | 12;
  // Evens = {2, 4, 6, 8, 10, 12}

  type Threes = 3 | 6 | 9 | 12;
  // Threes = {3, 6, 9, 12}

  // Intersecting a set A with a set B means extracting the part of A that also belongs to B.
  // In other words, elements in common to both

  type EvensThreeOverlap = Evens & Threes; // 6 | 12
  // EvensThreeOverlap = Evens ∩ Threes
  // {2, 4, 6, 8, 10, 12} ∩ {3, 6, 9, 12} = {6, 12}

  const eto: EvensThreeOverlap = 6;
}
```

![image](https://user-images.githubusercontent.com/1832432/193633331-82c395fb-6f33-4ec1-88bd-1d9fecc9a009.png)


The result of intersecting types that do not overlap is the empty set. A set that does not contain anything. The empty set is called `never` in TS.

```ts
type EmptySet = string & number;
// EmptySet = {} or EmptySet = Ø
```

## Empty Set

$\emptyset$ equals the empty set, also expressed as {}

There are no elements in an empty set, so there can be no elements in the empty set that aren't contained in the complete set. Therefore, the empty set is a subset of every set. In TS $\emptyset$  is expressed as `never`

> $\emptyset \subset A$

![image](https://user-images.githubusercontent.com/1832432/193636519-b312cb63-79a7-4663-946a-3d09f44e8dc1.png)


The union of any type $A$ with $\emptyset$ is equal to $A$.

```ts
type A = 1 | 2 | 3 | 4 | 5;

type B = A | never;
// = A
```

> $B = A \cup \emptyset$

> $A = B$

If you intersect a type $A$ with $\emptyset$ however, you will always get back $\emptyset$.

```ts
type A = 1 | 2 | 3 | 4 | 5;

type B = A & never;
// = never
```

## Universal Set

$\textit{U}$ equals the universal set

A Universal Set is the set of all elements under consideration. All other sets are subsets of the universal set. The Universal Set contains each and every type you will ever use in TypeScript. And is expressed as `unknown`.

> $A \subset \textit{U}$

> $B \subset \textit{U}$

> $C \subset \textit{U}$

![image](https://user-images.githubusercontent.com/1832432/193635426-51fec143-eca9-42ba-8ddc-05460b5c47a0.png)

The union of any type $A$ with $\textit{U}$ is equal to $\textit{U}$.

```ts
type A = 1 | 2 | 3 | 4 | 5;

type B = A | unknown;
// = unknown
```

If you intersect a type $A$ with $\textit{U}$ however, you will always get back $A$.

```ts
type A = 1 | 2 | 3 | 4 | 5;

type B = A & unknown;
// = A
```

# Back to your regularly scheduled Typescript

## Common Types

Before getting started, some ground work is needed. In Typescript there are some common categories of types. Some, like literals `type a = 42`, can be subsets of others, like `number`. Unions and Intersections also fall under this heading.

```ts
type Primitives =
  | number
  | string
  | boolean
  | symbol
  | bigint
  | undefined
  | null;
// A = {all numbers}
// B = {all strings}
// C = {true, false}
// ...

// Literals
type Literals =
  | 42
  | 'Typescript'
  | true;

type DataStructures =
  | { key1: boolean; key2: number } // objects
  | { [key: string]: number } // records
  | [boolean, number] // tuples
  | number[]; // arrays

// Object types describe objects with a finite set of keys, and these keys
// contain values of potentially different types.

// Record types are similar to object types, except they describe objects with
// an unknown number of keys, and all values
//     in a record share the same type.
//     For example, in { [key: string]: number }, all values are numbers.

// Tuple types describe arrays with a fixed length. They can have a different type for each index.

// Array types describe arrays with an unknown length.
// Just like with records, all values share the same type.
```

## Object Types [Data Structure]

Type objects, like the JS object they represent, are structured in the same way. And, also like the JS object, can have as many properties as is needed. Indexed by unique keys.

```ts
type Motorcycle = {
  make: string;
  model: string;
  year: number;
}

const Tuono: Motorcycle = {
  make: 'Aprilia',
  model: 'Tuono 660',
  year: 2021,
};
```

Since we are defining these objects inline TS doesnt let us add extra types as we wouldnt be able to reference those extra props afterwards due to the type.

```ts
const Rs: Motorcycle = {
  make: 'Aprilia',
  model: 'RSV4',
  year: 2021,
  // @ts-expect-error Object literal may only specify known properties
  engine: 'V4',
};
```

But if, for instance, we were getting those from an API response we would be able to assign them to Motorcycle (though we still wouldnt be able to use the extra). So an object type is the set of objects with _**at least**_ all properties it defines.

![image](https://user-images.githubusercontent.com/1832432/193663454-f8886b6d-c138-4254-8591-ff392fa9d7e0.png)


### Property Types

Prop types can be accessed similar to accessing prop values using bracket ([]) syntax. Trying to use dot notation will throw however.

```ts
type Make = Motorcycle['make']; // string
```

And since the "value" in the bracket is just a string literal, unions also apply.

```ts
type ModelOrYear = Motorcycle['model' | 'year']; // string | number
```

This is the same as accessing them each seperately.

```ts
type MakeOrYear = Motorcycle['make'] | Motorcycle['year']; // string | number
```

### keyof

`keyof` functions like `Object.keys` for the object types.

```ts
type Keys = keyof Motorcycle; // 'make' | 'model' | 'year'
```

And, like above, since they are string literals, you can use them to access the types.

```ts
type MotorcycleValues = Motorcycle[keyof Motorcycle]; // string | number
```

This pattern can be refactored to a generic for easy reuse

```ts
type ValueOf<T> = T[keyof T];

type MotoValues = ValueOf<Motorcycle>; // string | number
```

### Merging Object Types

Merging objects using the intersection symbol, unlike the normal behavior of

> extracting the part of A that also belongs to B

when used with objects, it acts more like `{ ...A, ...B}`.

```ts
type Name = { name: string };
type Age = { age: number };

type NameAndAge = Name & Age; // { name: string } & { age: number };

const user: NameAndAge = { name: 'Fred', age: 32 };

// @ts-expect-error Property 'age' is missing in type '{ name: string; }'
const userName: NameAndAge = { name: 'Fred' };

// @ts-expect-error Property 'name' is missing in type '{ age: number; }'
const userAge: NameAndAge = { age: 32 };
```

But why though? With objects, you are not intersecting their keys, but their subtyping set. Since you can assign object types with additional keys (`{ a: string, b: number, c: Date }`) to object types with fewer keys (`{ a: string, b: number }`) -- caveat, as long as it isnt done inline -- so there could be objects `{ a: string, b: number }` that contain a `c: Date`. So, intersecting objects returns the set of values that belong to both sets.

```ts
type KeyOfName = keyof Name; // 'name'
type KeyOfAge = keyof Age; // 'age'
type KeyOfNameAge = keyof NameAndAge; // 'age' | 'name'
```

![image](https://user-images.githubusercontent.com/1832432/193670225-8bd155ee-6b02-451b-acb2-942cf5406be1.png)


Caveat for duplicates. In the event that two objects contain the same prop of differing types (non-subsets), the resulting property type will be `never`.

```ts
type A = { a: string, b: number };
type B = { b: string, c: number };

type C = A & B;
// { a: string, b: never, c: number }

// @ts-expect-error "Type 'number' is not assignable to type 'never'." For prop 'b'
const tm: CMerge = { a: '', b: 0, c: 0 };
```

And the union of two objects is the opposite

```ts
type X = { name: string, height: number };
type Y = { name: string, age: number };

type KeyOfX = keyof X; // 'name' | 'height'
type KeyOfY = keyof Y; // 'name' | 'age'

type Z = X | Y;

type KeyOfZ = keyof Z; // 'name'
```

![image](https://user-images.githubusercontent.com/1832432/193676792-deea6bb4-138a-421a-b132-34bd841e550b.png)


If that doesn't make sense, the TL:DR is the intersection of two objects is the union of their keys and the union of two objects is the intersecction of their keys. Additionally, Unions and Intersections of objects aren't as performant as Interfaces. Though Interfaces can only be defined statically.

## Record Types [Data Structure]

Records are like Objects with the exception that all of the values, of all of the props, must be of the same type.

```ts
type NamesRecord = { [key: string]: string };
```

There is also a built in generic for this if prefered.

```ts
type AgeRecord = Record<string, number>;
```

The key for Records can also be a union.

```ts
type AddressRecord = { [key in 'street' | 'city' | 'state']: string };
```

which equates to

```ts
type AddressObj = {
  street: string;
  city: string;
  state: string;
}
```

And because of this, we can get the types of the props just like normal objects using `[]`. Though if the generic records are used, all of the props have the same value type and it can be simplified.

```ts
type TypeOfNamesRecord = NamesRecord[string]; // string

type TypeOfAgeRecord = AgeRecord[string]; // number
```

This simultaneously reads all keys assignable to the type `string`. Since all of them are the same type, you get that type back.

## Object Type Helpers

### `Partial<T>`

Partial takes an object type and returns a similar object type with all of the property types as optional props `{ a: string } → { a?: string }`.

```ts
type A = { name: string, age: number };

type PartialA = Partial<A>;
// { name?: string | undefined, age?: number | undefined }
```

### `Required<T>`

Required, like Partial, takes an object type but, instead, returns a similar object type with all of the property types as required props `{ a?: string } → { a: string }`.

```ts
type B = { name?: string, age?: number };

type RequiredB = Required<B>;
// { name: string, age: number }
```

### `ReadOnly<T>`

ReadOnly, like the above, also takes an object type and returns a similar object type. And as you have probably guess, the difference is that all of the props are readonly.

```ts
type A = { name: string, age: number };

type ReadOnlyA = Readonly<A>;
// { readonly name: string, readonly age: number }
```

### `Pick<T>`

Pick, those familiar with lodash will probably recognize this. It takes an object type and a key literal (or union of literals) and returns an object type with just those props.

```ts
type C = { name: string, age: number, height: number, weight: number };

type PickName = Pick<C, 'name'>;
// { name: string }

type PickHeightWeight = Pick<C, 'height' | 'weight'>;
// { height: number, weight: number }
```

### `Omit<T>`

Omit is similar to Pick except it does the opposite. It returns an object type with the given keys removed.

```ts
type C = { name: string, age: number, height: number, weight: number };

type OmitName = Omit<C, 'name'>;
// { age: number, height: number, weight: number }

type OmitHeightWeight = Omit<C, 'height' | 'weight'>;
// { name: string, age: number }
```