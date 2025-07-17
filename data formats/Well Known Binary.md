---
tags:
  - data
gardening: 🌱
date: 2025-07-17
reference:
  - https://www.ogc.org/standards/sfa/
  - https://postgis.net/docs/manual-3.0/
  - https://libgeos.org/specifications/wkb/
---
The Well Known Binary (WKB) format represents a fundamental building block in modern geospatial data processing. Unlike text-based formats such as Well Known Text (WKT) or GeoJSON, WKB provides a compact, efficient binary representation of geometric objects that optimizes both storage space and processing performance.

WKB's design philosophy centers on creating a platform-independent binary format that can represent complex geometric structures while maintaining compatibility across different hardware architectures and software systems.

## Core Structure

WKB data is structured as a sequence of bytes that encode the geometry type, coordinate values, and metadata such as byte order and SRID. The format adheres to a strict specification to ensure interoperability across systems.

```
┌─────────────────┬─────────────────┬─────────────────┐
│  Byte Order     │  Geometry Type  │  Geometry Data  │
│  (1 byte)       │  (4 bytes)      │  (variable)     │
└─────────────────┴─────────────────┴─────────────────┘
```

## Byte Order Semantics

WKB uses a single byte to indicate the byte order ([endianness](../computer%20science/Endianness.md)) of the data:

| **Value** | **Meaning**                                  |
| --------- | -------------------------------------------- |
| `0x00`    | Big endian (most significant byte first)     |
| `0x01`    | Little endian (least significant byte first) |

This design ensures cross-platform compatibility by explicitly declaring the byte ordering convention used throughout the geometry data.

## Geometry Type Taxonomy

The geometry type is encoded as a 32-bit unsigned integer immediately following the byte order. The value indicates the type of geometry and whether it includes Z or M coordinates. The OGC specification defines standard geometry type codes, with additional ranges for extended types.

### Basic Geometry Types

The WKB specification defines seven fundamental geometry types:

```typescript
type BasicGeometryType = 
  | 1 // Point
  | 2 // LineString  
  | 3 // Polygon
  | 4 // MultiPoint
  | 5 // MultiLineString
  | 6 // MultiPolygon
  | 7 // GeometryCollection
```

### Extended Geometry Types

Modern WKB implementations support additional geometry types for specialized applications. For geometries with Z or M coordinates, the type code is offset:

```
Add 1000 for Z
Add 2000 for M
Add 3000 for both Z and M
```

```typescript
type ExtendedGeometryType = 
  | 1001 // PointZ (3D Point)
  | 1002 // LineStringZ
  | 1003 // PolygonZ
  | 1004 // MultiPointZ
  | 1005 // MultiLineStringZ
  | 1006 // MultiPolygonZ
  | 1007 // GeometryCollectionZ
  | 2001 // PointM (Measured Point)
  | 2002 // LineStringM
  | 2003 // PolygonM
  | 3001 // PointZM (3D + Measured)
  | 3002 // LineStringZM
  | 3003 // PolygonZM
```

## Detailed Binary Structure Analysis

Coordinates are encoded as 64-bit double-precision floating-point numbers (IEEE 754). For a 2D point $(x, y)$, the WKB format includes two doubles. For 3D points (Z) or points with measures (M), additional doubles are included.

### Point Geometry

A 2D Point represents the simplest geometric object:

```
┌─────────┬─────────┬─────────┬─────────┐
│ Order   │ Type    │ X       │ Y       │
│ (1)     │ (4)     │ (8)     │ (8)     │
└─────────┴─────────┴─────────┴─────────┘
```

**Total Size**: 21 bytes

**Binary Layout**:

- Offset 0: Byte order indicator
- Offset 1-4: Geometry type (1 for Point)
- Offset 5-12: X coordinate (IEEE 754 double)
- Offset 13-20: Y coordinate (IEEE 754 double)

### LineString Geometry

A LineString contains an ordered sequence of points:

```
┌─────────┬─────────┬─────────┬─────────────────┐
│ Order   │ Type    │ Count   │ Points          │
│ (1)     │ (4)     │ (4)     │ (16 * count)    │
└─────────┴─────────┴─────────┴─────────────────┘
```

**Size Calculation**: $21 + 16 \times \text{point\_count}$ bytes

**Constraints**:

- Minimum point count: 2
- Points must form a continuous path
- First and last points may be identical (closed LineString)

### Polygon Geometry

A Polygon consists of one exterior ring and zero or more interior rings:

```
┌─────────┬─────────┬─────────┬─────────────────┐
│ Order   │ Type    │ Ring    │ Linear Rings    │
│ (1)     │ (4)     │ Count   │ (variable)      │
│         │         │ (4)     │                 │
└─────────┴─────────┴─────────┴─────────────────┘
```

**Ring Structure**: Each ring follows the LineString format with additional constraints:

- First and last points must be identical (closed ring)
- Minimum of 4 points per ring (3 unique + 1 closing)
- Exterior ring must be counter-clockwise
- Interior rings must be clockwise

### Multi-Geometry Types

Multi-geometry types contain collections of homogeneous geometries:

```
┌─────────┬─────────┬─────────┬─────────────────┐
│ Order   │ Type    │ Geom    │ Geometries      │
│ (1)     │ (4)     │ Count   │ (variable)      │
│         │         │ (4)     │                 │
└─────────┴─────────┴─────────┴─────────────────┘
```

Each contained geometry is a complete WKB geometry including its own byte order and type indicators.

## Performance Characteristics

### Space Efficiency

WKB provides significant space savings compared to text-based formats:

| Geometry Type        | WKB Size     | WKT Size      | Compression Ratio |
| -------------------- | ------------ | ------------- | ----------------- |
| Point                | 21 bytes     | ~25 chars     | ~1.2:1            |
| LineString (100 pts) | 1,609 bytes  | ~2,500 chars  | ~1.6:1            |
| Polygon (1000 pts)   | 16,009 bytes | ~25,000 chars | ~1.6:1            |

### Processing Performance

Binary format advantages:

- **Parsing Speed**: Direct binary reading vs. string parsing
- **Memory Layout**: Efficient for memory-mapped files
- **Network Transfer**: Reduced bandwidth requirements
- **Database Storage**: Native binary storage in spatial databases

## Extended WKB Features

### 3D Geometry Support

Extended WKB supports Z-coordinate for 3D geometries:

```
┌─────────┬─────────┬─────────┬─────────┬─────────┐
│ Order   │ Type    │ X       │ Y       │ Z       │
│ (1)     │ (4)     │ (8)     │ (8)     │ (8)     │
└─────────┴─────────┴─────────┴─────────┴─────────┘
```

### Measured Geometries

M-coordinate support for linear referencing:

```typescript
type PointM = { x: number; y: number; m: number };

// Applications:
// - Mile markers on highways
// - Time-based positioning
// - Engineering measurements
```

### Coordinate Reference Systems

If a spatial reference system identifier (SRID) is included (e.g., for EPSG codes like 4326 for WGS84), it is encoded as a 32-bit unsigned integer before the geometry data.

```
┌─────────┬─────────┬─────────┬─────────────────┐
│ Order   │ Type    │ SRID    │ Geometry Data   │
│ (1)     │ (4)     │ (4)     │ (variable)      │
└─────────┴─────────┴─────────┴─────────────────┘
```

## Comparison with Alternative Formats

### WKT (Well Known Text)

|Aspect|WKB|WKT|
|---|---|---|
|Size|Compact|Verbose|
|Human Readable|No|Yes|
|Parsing Speed|Fast|Slow|
|Network Transfer|Efficient|Inefficient|
|Debugging|Difficult|Easy|

### GeoJSON

|Aspect|WKB|GeoJSON|
|---|---|---|
|Size|Compact|Very Verbose|
|Web Compatibility|Requires conversion|Native|
|Precision|Full double precision|JSON number limits|
|Metadata|Limited|Rich properties|

### Protocol Buffers

|Aspect|WKB|Protocol Buffers|
|---|---|---|
|Standardization|OGC standard|Google standard|
|Schema Evolution|Fixed|Flexible|
|Geometry Focus|Specialized|General purpose|
|Tooling|GIS-specific|General development|
