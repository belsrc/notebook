---
tags:
  - data
  - gis
gardening: 🌳
date: 2025-10-29
reference:
  - http://schemas.opengis.net/filter/1.0.0/
  - http://schemas.opengis.net/filter/1.1.0/
  - http://schemas.opengis.net/filter/2.0/
  - https://www.ogc.org/standards/filter/
  - https://www.iso.org/standard/42137.html
---
The OGC Filter Encoding specification defines an XML encoding for expressing query filter expressions. These filters constrain property values of objects to identify specific subsets for operations. The specification evolved from version 1.0.0 through 1.1.0 to 2.0, with each iteration adding capabilities while maintaining backward compatibility concepts.

## Core Concepts

### Filter Expression Structure

A filter expression is a construct used to constrain property values of objects. The root element `<Filter>` contains predicate expressions that evaluate to boolean values (true/false). If a property satisfies all predicates, the object is included in the result set.

**Basic structure:**
```xml
<Filter>
  <!-- Predicate operators: comparison, spatial, temporal, logical -->
</Filter>
```

### Property References

Property names reference facets or attributes of objects:

**Version 1.0/1.1:** `<PropertyName>`  
**Version 2.0:** `<ValueReference>` (renamed for broader applicability)

All versions support XPath expressions for referencing complex properties in XML-encoded objects.

## Version 1.0.0 (OGC 02-059)

**Date:** Original specification, later revised  
**Key characteristics:**
- Initial XML encoding of Common Catalog Query Language (CQL) predicates
- Foundation for WFS and other OGC services
- GML 2.x support

### Operator Categories

#### Comparison Operators

- `PropertyIsEqualTo`, `PropertyIsNotEqualTo`
- `PropertyIsLessThan`, `PropertyIsGreaterThan`
- `PropertyIsLessThanOrEqualTo`, `PropertyIsGreaterThanOrEqualTo`
- `PropertyIsLike` (pattern matching with wildCard, singleChar, escapeChar)
- `PropertyIsNull` (tests for null values)
- `PropertyIsBetween` (range checks with inclusive boundaries)

#### Spatial Operators

Based on OGC Simple Features Specification for SQL:
- `Equals`, `Disjoint`, `Touches`, `Within`, `Overlaps`
- `Crosses`, `Intersects`, `Contains`
- `DWithin`, `Beyond` (distance-based operators)
- `BBOX` (bounding box, equivalent to NOT Disjoint)

Geometries encoded using GML 2.x.

#### Logical Operators

- `And` (all conditions must be true)
- `Or` (any condition must be true)
- `Not` (negation)

#### Object Identifiers

- `<FeatureId>` with `fid` attribute for GML 2 features

#### Expressions

- `<Literal>` for constant values
- `<Function>` for extensible operations
- Arithmetic operators: `Add`, `Sub`, `Mul`, `Div`

### Filter Capabilities

Services declare supported functionality via `Filter_Capabilities` XML:
- `Spatial_Capabilities`: Lists supported spatial operators
- `Scalar_Capabilities`: Declares logical operators, comparison operators, arithmetic operators, and functions
- `Id_Capabilities`: Indicates support for feature identifiers

## Version 1.1.0 (OGC 04-095)

**Date:** 2005-05-03  
**Editor:** Panagiotis A. Vretanos (CubeWerx Inc.)

### Major Changes from 1.0.0

#### Enhanced Property Addressing

- Full XPath support for complex attribute navigation
- Path expressions can traverse nested XML structures
- Support for attribute references using `@attribute` syntax

**XPath subset rules:**
1. Abbreviated child and attribute axis specifiers required
2. Support for predicates (indices, value equality, child constraints)
3. Final step must be a property or sub-component
4. Logical combination of predicates using `and`/`or`

**Example XPath expressions:**

```
Person/lastName
Person/mailAddress/Address/city
Person/phone[1]
Person/@SIN
mailAddress/Address[street="Oxfordstrasse"]/number
```

#### Object Identifier Abstraction

- Introduced abstract `<_Id>` element as substitution group head
- `<GmlObjectId>` for GML 3.x with `gml:id` attribute
- `<FeatureId>` deprecated but retained for GML 2 backward compatibility

#### Sorting Support

- Added `SORTBY` encoding for result set ordering
- Capabilities document extended to describe sorting support

#### GML 3.x Support

- Updated for GML 3.1.1 compatibility
- Geometry operands extended (Arc, Circle, Bezier, Clothoid, CubicSpline, Geodesic, OffsetCurve, Triangle, PolyhedralSurface, TriangulatedSurface, Tin, Solid)

#### Enhanced Filter Capabilities

```xml
<Filter_Capabilities>
  <Spatial_Capabilities>
    <GeometryOperands>...</GeometryOperands>
    <SpatialOperators>...</SpatialOperators>
  </Spatial_Capabilities>
  <Scalar_Capabilities>
    <LogicalOperators/>
    <ComparisonOperators>...</ComparisonOperators>
    <ArithmeticOperators>...</ArithmeticOperators>
  </Scalar_Capabilities>
  <Id_Capabilities>
    <EID/>
    <FID/>
  </Id_Capabilities>
</Filter_Capabilities>
```

### Comparison Operator Enhancements

- `matchCase` attribute (Boolean, default=true) for case-sensitive string comparisons

### Spatial Reference System Handling

- Defined rules for SRS handling in literal geometries
- Default SRS from queried feature type applies when unspecified
- Exception raised if SRS doesn't match supported values

## Version 2.0 (OGC 09-026r2 / ISO 19143:2010)

**Date:** 2010-11-22  
**Editor:** Peter Vretanos  
**Status:** OGC Standard and ISO International Standard (ISO 19143:2010)

### Architectural Changes

#### Query Expression Framework

New abstract query model:
- `AbstractQueryExpression` (base for all queries)
- `AbstractAdhocQueryExpression` (for ad hoc queries)
  - `typeNames`: Resource types to query
  - `aliases`: Alternate names for resource types
  - Projection clause (`AbstractProjectionClause`)
  - Selection clause (`AbstractSelectionClause`)
  - Sorting clause (`AbstractSortingClause`)

#### KVP (Keyword-Value Pair) Encoding

Added URL parameter encoding alongside XML:

| Parameter | Description |
|-----------|-------------|
| `TYPENAMES` | Comma-separated list of resource types |
| `ALIASES` | Comma-separated list of aliases |
| `PROPERTYNAME` | Projection clause properties |
| `FILTER` | XML-encoded filter expression |
| `FILTER_LANGUAGE` | Predicate language URI |
| `RESOURCEID` | Comma-separated resource identifiers |
| `BBOX` | Bounding rectangle |
| `SORTBY` | Sorting specification |

#### Terminology Updates

- "Property" → "Value Reference" (broader scope beyond properties)
- "Feature" → "Resource" (generalized to any identifiable object)
- `<PropertyName>` → `<ValueReference>`

### Temporal Operators (New in 2.0)

Based on ISO 19108 temporal relationships:

- `After`, `Before`, `Begins`, `BegunBy`
- `TContains`, `During`, `TEquals`, `TOverlaps`
- `Meets`, `MetBy`, `OverlappedBy`, `EndedBy`
- `Ends`, `AnyInteracts`

**Encoding:**
```xml
<After>
  <ValueReference>dateProperty</ValueReference>
  <gml:TimeInstant>...</gml:TimeInstant>
</After>
```

Time zone handling follows XML Schema Part 2 datatypes specification.

### Enhanced Comparison Operators

#### New Operators

- `PropertyIsNil`: Tests if property value is nil (with optional `nilReason`)

#### matchAction Attribute

Controls evaluation for multi-valued properties:
- `Any`: At least one value satisfies predicate (default)
- `All`: All values must satisfy predicate
- `One`: Exactly one value satisfies predicate

**Example:**
```xml
<PropertyIsEqualTo matchAction="Any">
  <ValueReference>gml:name</ValueReference>
  <Literal>SearchValue</Literal>
</PropertyIsEqualTo>
```

### Resource Identification

`<ResourceId>` replaces feature-specific identifiers with generic resource identification:

**Attributes:**
- `rid`: Resource identifier (required)
- `previousRid`: Previous identifier for versioning
- `version`: Version number, date, or token (FIRST, LAST, PREVIOUS, NEXT, ALL)
- `startDate`, `endDate`: Time-based version range

**Example:**
```xml
<ResourceId rid="resource.123" version="LAST"/>
```

### Conformance Classes

FES 2.0 defines 15 conformance classes:

| Class | Description |
|-------|-------------|
| Query | Materializes `AbstractQueryElement` |
| Ad hoc Query | Implements ad hoc query pattern |
| Functions | Adds custom functions |
| Resource Identification | Implements `ResourceId` operator |
| Minimum Standard Filter | Basic comparison + logical operators |
| Standard Filter | All comparison + logical operators + functions |
| Minimum Spatial Filter | BBOX only |
| Spatial Filter | BBOX + additional spatial operators |
| Minimum Temporal Filter | During only |
| Temporal Filter | During + additional temporal operators |
| Version navigation | ResourceId version parameters |
| Sorting | Result set ordering |
| Extended Operators | Custom operators beyond standard |
| Minimum XPath | Required XPath subset |
| Schema Element Function | `schema-element()` XPath function |

Standards referencing FES 2.0 must declare minimum conformance requirements.

### Extension Mechanisms

#### Function Extension
```xml
<Function name="CustomFunction">
  <Literal>arg1</Literal>
  <ValueReference>property</ValueReference>
</Function>
```

Functions declared in capabilities document with metadata.

#### Operator Extension

New operators added via substitution groups:
- `comparisonOps`
- `spatialOps`
- `temporalOps`
- `extensionOps`

Custom operators must use non-FES namespaces.

### Enhanced Capabilities Document

```xml
<Filter_Capabilities>
  <Conformance>
    <Constraint name="ImplementsQuery">
      <NoValues/>
      <DefaultValue>TRUE</DefaultValue>
    </Constraint>
    <!-- Additional conformance classes -->
  </Conformance>
  <Id_Capabilities>
    <ResourceIdentifier name="fes:ResourceId"/>
  </Id_Capabilities>
  <Scalar_Capabilities>
    <LogicalOperators/>
    <ComparisonOperators>...</ComparisonOperators>
  </Scalar_Capabilities>
  <Spatial_Capabilities>
    <GeometryOperands>...</GeometryOperands>
    <SpatialOperators>...</SpatialOperators>
  </Spatial_Capabilities>
  <Temporal_Capabilities>
    <TemporalOperands>...</TemporalOperands>
    <TemporalOperators>...</TemporalOperators>
  </Temporal_Capabilities>
  <Functions>...</Functions>
  <Extended_Capabilities>...</Extended_Capabilities>
</Filter_Capabilities>
```

### Join Queries

Support for multi-resource type queries:

```xml
<typeNames>ns1:Type1 ns2:Type2</typeNames>
<aliases>a b</aliases>
<Filter>
  <PropertyIsEqualTo>
    <ValueReference>/a/property</ValueReference>
    <ValueReference>/b/foreignKey</ValueReference>
  </PropertyIsEqualTo>
</Filter>
```

Implements inner join semantics by default.

### Units of Measure

Distance operators use `MeasureType` with `uom` attribute:

```xml
<Distance uom="m">10.0</Distance>
```

UOM identifier can be symbol or URI to definition.

## Comparison Matrix

| Feature | 1.0.0 | 1.1.0 | 2.0 |
|---------|-------|-------|-----|
| **Property References** |
| Basic property names | ✓ | ✓ | ✓ |
| XPath expressions | Limited | Full subset | Full subset |
| Element name | `PropertyName` | `PropertyName` | `ValueReference` |
| **Operators** |
| Comparison operators | 8 | 8 | 9 (+PropertyIsNil) |
| Spatial operators | 11 | 11 | 11 |
| Temporal operators | - | - | 14 |
| Logical operators | 3 | 3 | 3 |
| matchCase attribute | - | ✓ | ✓ |
| matchAction attribute | - | - | ✓ |
| **Identifiers** |
| FeatureId (GML 2) | ✓ | ✓ (deprecated) | - |
| GmlObjectId (GML 3) | - | ✓ | - |
| ResourceId | - | - | ✓ |
| Version navigation | - | - | ✓ |
| **Query Model** |
| Filter only | ✓ | ✓ | ✓ |
| Abstract query expressions | - | - | ✓ |
| Ad hoc queries | - | - | ✓ |
| Projection clause | - | - | ✓ |
| Selection clause | Implicit | Implicit | Explicit |
| Sorting clause | - | ✓ | ✓ |
| Join queries | - | - | ✓ |
| **Encoding** |
| XML | ✓ | ✓ | ✓ |
| KVP (URL parameters) | - | - | ✓ |
| **GML Support** |
| GML 2.x | ✓ | ✓ | ✓ |
| GML 3.x | - | ✓ | ✓ (primary) |
| **Conformance** |
| Capabilities document | ✓ | ✓ | ✓ (enhanced) |
| Conformance classes | Implicit | Implicit | 15 explicit |
| **Standards Status** |
| OGC specification | ✓ | ✓ | ✓ |
| ISO standard | - | - | ✓ (19143:2010) |

---

## Migration Guidance

### 1.0.0 → 1.1.0

- **Required changes:**
  - Update GML namespace references for 3.x support
  - Replace `<FeatureId>` with `<GmlObjectId>` for GML 3 data
- **Optional enhancements:**
  - Adopt XPath expressions for complex property navigation
  - Implement SORTBY for ordered results

### 1.1.0 → 2.0

- **Required changes:**
  - Rename `<PropertyName>` to `<ValueReference>`
  - Update namespace to `http://www.opengis.net/fes/2.0`
  - Replace `<GmlObjectId>` with `<ResourceId>`
- **Optional enhancements:**
  - Implement temporal operators for time-based filtering
  - Add KVP encoding for URL-based queries
  - Adopt conformance classes for capabilities declaration
  - Implement version navigation if supporting versioned resources
  - Use `matchAction` for multi-valued property filtering

### Backward Compatibility Notes

- Version 2.0 is not directly backward compatible with 1.x XML syntax
- Conceptual compatibility maintained (same logical operations)
- Translation between versions feasible with XSLT or similar tools
- Services may support multiple versions simultaneously

## Implementation Considerations

### XPath Subset Requirements

All versions supporting complex properties require XPath support. Minimum requirements (v1.1/2.0):

1. Abbreviated child (`/`) and attribute (`@`) axis specifiers
2. Context node is resource element (or parent for joins)
3. Predicates supporting:
   - Positive integer indices: `property[2]`
   - Equality tests: `property[child="value"]`
   - Logical combinations: `property[a="x" and b="y"]`
4. Final step must be property or sub-component

**EBNF grammar** defined in FES 2.0 Annex D.

### Geometry Handling

**Coordinate Reference Systems:**
- GML geometries include optional `srsName` attribute
- Services must define behavior when SRS differs or is unspecified
- Transformations may be required for spatial operator evaluation

**Supported geometry types vary by version:**
- Basic: Point, LineString, Polygon, Envelope
- Extended (1.1+): Arc, Circle, Bezier, Clothoid, CubicSpline, Geodesic, OffsetCurve
- Solids (1.1+): Triangle, PolyhedralSurface, TriangulatedSurface, Tin, Solid

## Example Filters

### Simple Comparison (all versions)

```xml
<Filter>
  <PropertyIsEqualTo>
    <PropertyName>DEPTH</PropertyName>  <!-- 1.x -->
    <!-- or <ValueReference>DEPTH</ValueReference> for 2.0 -->
    <Literal>30</Literal>
  </PropertyIsEqualTo>
</Filter>
```

### Spatial Filter (all versions)

```xml
<Filter>
  <Intersects>
    <PropertyName>Geometry</PropertyName>
    <gml:Polygon srsName="EPSG:4326">
      <gml:exterior>
        <gml:LinearRing>
          <gml:posList>0 0 10 0 10 10 0 10 0 0</gml:posList>
        </gml:LinearRing>
      </gml:exterior>
    </gml:Polygon>
  </Intersects>
</Filter>
```

### Temporal Filter (2.0 only)

```xml
<Filter>
  <During>
    <ValueReference>observationDate</ValueReference>
    <gml:TimePeriod>
      <gml:beginPosition>2024-01-01T00:00:00Z</gml:beginPosition>
      <gml:endPosition>2024-12-31T23:59:59Z</gml:endPosition>
    </gml:TimePeriod>
  </During>
</Filter>
```

### Complex Logical Expression (all versions)

```xml
<Filter>
  <And>
    <PropertyIsBetween>
      <PropertyName>temperature</PropertyName>
      <LowerBoundary><Literal>20</Literal></LowerBoundary>
      <UpperBoundary><Literal>30</Literal></UpperBoundary>
    </PropertyIsBetween>
    <Or>
      <PropertyIsEqualTo>
        <PropertyName>status</PropertyName>
        <Literal>active</Literal>
      </PropertyIsEqualTo>
      <PropertyIsEqualTo>
        <PropertyName>status</PropertyName>
        <Literal>pending</Literal>
      </PropertyIsEqualTo>
    </Or>
  </And>
</Filter>
```

### XPath Complex Property (1.1+)

```xml
<Filter>
  <PropertyIsEqualTo>
    <PropertyName>Person/mailAddress/Address/city</PropertyName>
    <Literal>Toronto</Literal>
  </PropertyIsEqualTo>
</Filter>
```

### Pattern Matching (all versions)

```xml
<Filter>
  <PropertyIsLike wildCard="*" singleChar="?" escapeChar="\">
    <PropertyName>name</PropertyName>
    <Literal>John*</Literal>
  </PropertyIsLike>
</Filter>
```

### Resource Identification (2.0)

```xml
<Filter>
  <ResourceId rid="building.123" version="3"/>
  <ResourceId rid="building.456" version="LAST"/>
</Filter>
```

### matchAction Example (2.0)

```xml
<Filter>
  <PropertyIsEqualTo matchAction="Any">
    <ValueReference>gml:name</ValueReference>
    <Literal>Flatiron</Literal>
  </PropertyIsEqualTo>
</Filter>
```
