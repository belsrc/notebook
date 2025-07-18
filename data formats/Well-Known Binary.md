---
tags:
  - data
  - gis
gardening: 🌳
date: 2025-07-17
reference:
  - https://www.ogc.org/standards/sfa/
  - https://postgis.net/docs/manual-3.0/
  - https://libgeos.org/specifications/wkb/
  - https://medium.com/@hanxuyang0826/understanding-and-implementing-wkb-in-c-with-postgis-0c5d5e594805
  - https://loc.gov/preservation/digital/formats/fdd/fdd000549.shtml
  - https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry#Well-known_binary
---
The Well-Known Binary (WKB) format is a crucial component in modern geospatial data processing. Unlike text-based formats such as Well-Known Text (WKT) or GeoJSON, WKB offers a compact and efficient binary representation of geometric objects, optimizing both storage space and processing performance.

The design philosophy behind WKB emphasizes the creation of a platform-independent binary format capable of representing complex geometric structures while ensuring compatibility across various hardware architectures and software systems.

### Why WKB?

Let's consider a simple point geometry: `POINT (10.0 20.0)`. In Well-Known Text (WKT), this is represented as a string of characters. However, using a binary representation can significantly reduce its size. The main reasons for using Well-Known Binary (WKB) format include:

1. **Compactness**: WKB offers much smaller file sizes and network payloads compared to text-based formats like WKT.
2. **Performance**: It allows for faster parsing and serialization through direct byte manipulation, eliminating the overhead of character encoding and decoding.
3. **Database Integration**: WKB is ideal for direct storage in spatial database columns, which minimizes conversion overhead.

## Core Structure

WKB data is organized as a sequence of bytes that represent the geometry type, coordinate values, and metadata such as byte order and SRID. This format follows a strict specification to ensure interoperability among systems.

```
┌─────────────────┬─────────────────┬─────────────────┐
│  Byte Order     │  Geometry Type  │  Geometry Data  │
│  (1 byte)       │  (4 bytes)      │  (variable)     │
└─────────────────┴─────────────────┴─────────────────┘
```

## Byte Order Semantics

WKB uses a single byte to indicate the byte order ([endianness](../computer%20science/Endianness.md)) of the data:

```c
#define WKB_BIG_ENDIAN    0x00  // most significant byte first
#define WKB_LITTLE_ENDIAN 0x01  // least significant byte first
```

This design ensures cross-platform compatibility by explicitly declaring the byte ordering convention used throughout the geometry data.

## Geometry Type Taxonomy

The geometry type is represented as a 32-bit unsigned integer that follows the byte order. This value indicates the type of geometry and specifies whether it includes Z (elevation) or M (measure) coordinates. The Open Geospatial Consortium (OGC) specification defines standard geometry type codes, along with additional ranges for extended types.

### Basic Geometry Types

The WKB specification defines seven fundamental geometry types:

```c
#define WKB_POINT              1
#define WKB_LINESTRING         2
#define WKB_POLYGON            3
#define WKB_MULTIPOINT         4
#define WKB_MULTILINESTRING    5
#define WKB_MULTIPOLYGON       6
#define WKB_GEOMETRYCOLLECTION 7
```

### Extended Geometry Types

Modern WKB implementations support additional geometry types for specialized applications. For geometries with Z or M coordinates, the type code is offset:

```
Add 1000 for Z
Add 2000 for M
Add 3000 for both Z and M
```

```c
#define WKB_POINTZ              1001 // 3D Point
#define WKB_LINESTRINGZ         1002 // 3D LineString
#define WKB_POLYGONZ            1003 // 3D Polygon
#define WKB_MULTIPOINTZ         1004 // 3D MultiPoint
#define WKB_MULTILINESTRINGZ    1005 // 3D MultiLineString
#define WKB_MULTIPOLYGONZ       1006 // 3D MultiPolygon
#define WKB_GEOMETRYCOLLECTIONZ 1007 // 3D GeometryCollection

#define WKB_POINTM            2001  // Measured Point
#define WKB_LINESTRINGM       2002  // Measured LineString
#define WKB_POLYGONM          2003  // Measured Polygon

#define WKB_POINTZM           3001  // 3D + Measured Point
#define WKB_LINESTRINGZM      3002  // 3D + Measured LineString
#define WKB_POLYGONZM         3003  // 3D + Measured Polygon
```

## Detailed Binary Structure Analysis

Coordinates are represented as 64-bit double-precision floating-point numbers (IEEE 754). For a 2D point $(x, y)$, the WKB format includes two doubles. For 3D points (Z) or points with measures (M), additional doubles are added.

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

```c
const uint8_t point_example[] = {
    0x01,                                           // Little endian
    0x01, 0x00, 0x00, 0x00,                         // Point type (1)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF8, 0x3F, // X = 1.5
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x04, 0x40  // Y = 2.5
};
```

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

```c
const uint8_t linestring_example[] = {
    0x01,                                           // Little endian
    0x02, 0x00, 0x00, 0x00,                         // LineString type (2)
    0x02, 0x00, 0x00, 0x00,                         // Point count (2)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Point 1: X = 0.0
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Point 1: Y = 0.0
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F, // Point 2: X = 1.0
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F  // Point 2: Y = 1.0
};
```

### Polygon Geometry

A Polygon consists of one exterior ring and zero or more interior rings:

```
┌─────────┬─────────┬────────────┬─────────────────┐
│ Order   │ Type    │ Ring Count │ Linear Rings    │
│ (1)     │ (4)     │ (4)        │ (variable)      │
└─────────┴─────────┴────────────┴─────────────────┘
```

**Ring Structure**: Each ring follows the LineString format with additional constraints:

- First and last points must be identical (closed ring)
- Minimum of 4 points per ring (3 unique + 1 closing)
- Exterior ring must be counter-clockwise
- Interior rings must be clockwise

```c
const uint8_t polygon_example[] = {
    0x01,                                           // Little endian
    0x03, 0x00, 0x00, 0x00,                         // Polygon type (3)
    0x01, 0x00, 0x00, 0x00,                         // Ring count (1)
    0x05, 0x00, 0x00, 0x00,                         // Point count in ring (5)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Point 1: (0,0)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F, // Point 2: (1,0)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F, // Point 3: (1,1)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Point 4: (0,1)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Point 5: (0,0) - closing
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};
```

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

```c
const uint8_t multipoint_example[] = {
    0x01,                                           // Little endian
    0x04, 0x00, 0x00, 0x00,                         // MultiPoint type (4)
    0x02, 0x00, 0x00, 0x00,                         // Geometry count (2)
    // First Point geometry
    0x01,                                           // Little endian
    0x01, 0x00, 0x00, 0x00,                         // Point type (1)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // X = 0.0
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // Y = 0.0
    // Second Point geometry
    0x01,                                           // Little endian
    0x01, 0x00, 0x00, 0x00,                         // Point type (1)
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F, // X = 1.0
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xF0, 0x3F  // Y = 1.0
};
```

## Binary Data Examples

**Point (10.5, 20.25) in Little Endian:**

```
01 01 00 00 00 00 00 00 00 00 00 25 40 00 00 00 00 00 40 34 40
│  │           │                                │
│  │           └─ X coordinate (10.5)           └─ Y coordinate (20.25)
│  └─ Geometry type (Point = 1)
└─ Byte order (Little Endian = 0x01)
```

**LineString with 3 points:**

```
01 02 00 00 00 03 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 F0 3F 00 00 00 00 00 00 00 40
00 00 00 00 00 00 00 40 00 00 00 00 00 00 08 40
│  │           │           │
│  │           │           └─ Points: (0,0), (1,2), (2,3)
│  │           └─ Point count (3)
│  └─ Geometry type (LineString = 2)
└─ Byte order (Little Endian)
```

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

```
┌─────────┬─────────┬─────────┬─────────┬─────────┐
│ Order   │ Type    │ X       │ Y       │ M       │
│ (1)     │ (4)     │ (8)     │ (8)     │ (8)     │
└─────────┴─────────┴─────────┴─────────┴─────────┘
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
