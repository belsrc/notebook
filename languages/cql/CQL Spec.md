---
tags:
  - cql
  - gis
reference:
  - https://portal.ogc.org/files/96288
  - https://portal.ogc.org/files/96288#cql-bnf
gardening: 🌱
---
A literal value is any part of an CQL filter expression that is used exactly as it is specified in the expression.

Properties in an object being evaluated in the CQL filter expression can be referenced by their name

```
http://www.someserver.com/ogcapi/search?
  collections=collection1,collection3&
  filter-lang=cql-text&
  filter=prop1=10 AND prop2>45
```

#### Comparisons

A binary comparison predicate evaluates two scalar expressions to determine if the expressions satisfy the specified comparison operator. Both scalar expressions SHALL evaluate to the same type of literal.

```
city='Toronto'

avg(windSpeed) < 4
```

#### Like

The _like predicate_ tests whether a string value matches the specified pattern.

The character specified using the `WILDCARD` modifier SHALL match zero of more characters in the test value. The wildchar character SHALL not match the NULL value. If the `WILDCHAR` modifier is not specified, the default wildcard character SHALL be `%`.

The character specified using the `SINGLECHAR` modifier SHALL match exactly one characters in the test value. If the `SINGLECHAR` modifier is not specified, the default singlechar character SHALL be `_`.

The character specified using the `ESCAPECHAR` modifier SHALL be used to escape the `WILDCHAR` and/or `SINGLECAHR` characters in the pattern string. If the `ESCAPECHAR` modifier is not specified, the default `escapechar` character SHALL be `\`.

If `NOCASE` is `TRUE`, the comparison SHALL be case insensitive, if it is `FALSE` it SHALL be case sensitive. The default is `TRUE`.

```
name LIKE 'Smith.' SINGLECHAR '.' NOCASE true
```

#### Between

The _between predicate_  tests whether a numeric value lies within the specified range. The between operator is inclusive.

```
depth BETWEEN 100.0 and 150.0
```

#### In

The _in-list predicate_  tests, for equality, the value of a scalar expression against a list of values of the same type.

The items in the list of an in-list predicate (i.e., the items on the right hand side of the predicate) SHALL be of the same literal type as the value being tested by the predicate.

```
cityName IN ('Toronto','Franfurt','Tokyo','New York') NOCASE false

category NOT IN (1,2,3,4)
```

#### Null

The _null predicate_ tests whether the value of a scalar expression is null.

```
geometry IS NOT NULL
```

#### Spatial

A _spatial predicate_ evaluates two geometry-valued expressions to determine if the expressions satisfy the requirements of the specified spatial operator.

> [!note]
> The spatial predicates currently use a different pattern than the comparison and temporal predicates and are written like function calls. This difference is inherited from previous versions of CQL.

CQL uses Well-Known-Text (WKT) to encode geometry literals. Since WKT does not provide a capability to specify the CRS (Coordinate Reference System) of a geometry literal, the server has to determine the CRS of the geometry literals in a filter expression through another mechanism. In this standard, the `filter-crs` query parameter is used to pass the CRS information to the server.

```
INTERSECTS(geometry,POLYGON((36.319836 32.288087,36.320041 32.288032,36.320210 32.288402,36.320008 32.288458,36.319836 32.288087)))
```

#### Temporal

A temporal predicate evaluates two time-valued expressions to determine if the expressions satisfy the requirements of the specified temporal operator.

```
event_date ANYINTERACTS 1969-07-16T05:32:00Z/1969-07-24T16:50:35Z
```

#### Encodings

This OGC API - Features - Part 3 standard defines a **text** encoding and a **JSON** encoding of CQL that covers Simple CQL and all enhanced capabilities specified in the next clause.

#### Enhanced Spatial

- ∩ - intersection, operation on two or more sets
- ∧ - and, logical intersection
- ∅ - empty set, the set having no members
- ≠ - not equal
- ⟺ - if and only if, logical equivalence between statements
- ⊆ - is a subset of
- dim(x) - returns the maximum dimension (-1, 0, 1, or 2) of the geometric object x

| Spatial operator | Definition                                                                             |
| ---------------- | -------------------------------------------------------------------------------------- |
| EQUALS           | EQUALS(a,b) ⟺ a ⊆ b ∧ b ⊆ a                                                            |
| DISJOINT         | DISJOINT(a,b) ⟺ a ∩ b = ∅                                                              |
| TOUCHES          | TOUCHES(a,b) ⟺ (I(a) ∩ I(b) = ∅) ∧ (a ∩ b) ≠ ∅                                         |
| WITHIN           | WITHIN(a,b) ⟺ (a ∩ b = a) ∧ (I(a) ∩ E(b) ≠ ∅)                                          |
| OVERLAPS         | OVERLAPS(a,b) ⟺ (dim(I(a)) = dim(I(b)) = dim(I(a) ∩ I(b))) ∧ (a ∩ b ≠ a) ∧ (a ∩ b ≠ b) |
| CROSSES          | CROSSES(a,b) ⟺ [I(a) ∩ I(b) = ∅) ∧ (a ∩ b ≠ a) ∧ (a ∩ b ≠ b)]                          |
| INTERSECTS       | INTERSECTS(a,b) ⟺ ! a DISJOINT b                                                       |
| CONTAINS         | CONTAINS(a,b) ⟺ b CONTAINS a                                                           |
![](../../images/cql/cql-touches.png)
![](../../images/cql/cql-within.png)
> [!note]
> If geometry **_a_** `CONTAINS` geometry **_b_**, then geometry **_b_** is `WITHIN` geometry **_a_**.

![](../../images/cql/cql-overlaps.png)

![](../../images/cql/cql-crosses.png)

#### Enhanced Temporal

CQL supports date and timestamps as time instants, but even the smallest "instant" has a duration and can also be evaluated as an interval. For the purposes of determining the temporal relationship between two temporal expressions, an instant is treated as the interval from the beginning to the end of the instant.

- `AFTER`
- `BEFORE`
- `BEGINS`
- `BEGUNBY`
- `TCONTAINS`
- `DURING`
- `ENDEDBY`
- `ENDS`
- `TEQUALS`
- `MEETS`
- `METBY`
- `TOVERLAPS`
- `OVERLAPPEDBY`

image

```
1969-07-20T20:17:40Z DURING 1969-07-16T13:32:00Z/1969-07-24T16:50:35Z

touchdown DURING 1969-07-16T13:32:00Z/1969-07-24T16:50:35Z
```

![](../../images/cql/cql-time-intervals.png)

#### Arithmetic Expressions

This clause specifies requirements for supporting arithmetic expressions in CQL. An arithmetic expression is an expression composed of an arithmetic operand (a property name, a number or a function that returns a number), an arithmetic operator (i.e. one of `+`,`-`,`*`,`/`) and another arithmetic operand.

```
vehicle_height > (bridge_height-1)
```

#### Arrays

Both array expressions SHALL be evaluated as sets. No inherent order SHALL be implied in a array of values.

- `AEQUALS` evaluates to the Boolean value `TRUE`, if both sets are identical; otherwise the predicate SHALL evaluate to the Boolean value `FALSE`.
- `ACONTAINS` evaluates to the Boolean value `TRUE`, if the first set is a superset of the second set; otherwise the predicate SHALL evaluate to the Boolean value `FALSE`.
- `CONTAINED BY` evaluates to the Boolean value `TRUE`, if the first set is a subset of the second set; otherwise the predicate SHALL evaluate to the Boolean value `FALSE`.
- `AOVERLAPS` evaluates to the Boolean value `TRUE`, if both sets share at least one common element; otherwise the predicate SHALL evaluate to the Boolean value `FALSE`.

```
layer:ids ACONTAINS ["layers-ca","layers-us"]
```

