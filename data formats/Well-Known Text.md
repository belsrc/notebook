---
tags:
  - data
  - gis
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry#Well-known_binary
  - https://www.loc.gov/preservation/digital/formats/fdd/fdd000548.shtml?loclr=blogsig
  - https://www.here.com/learn/blog/wkt-well-known-text-format-with-hereapis
  - https://mapscaping.com/a-guide-to-wkt-in-gis/
---
Well-Known Text (WKT) is a markup language used to represent vector geometry objects on a map, along with the spatial reference systems of these objects and the transformations between different spatial reference systems. Originally developed by the Open Geospatial Consortium (OGC), WKT provides a standardized method for expressing geometric data in a format that is easily readable by humans.

### Why is WKT Important?

- **Interoperability**: Well-Known Text (WKT) offers a standardized format for exchanging spatial data between different systems. A geometry created in one application can be readily shared and understood by another, provided both applications comply with the WKT standard.
- **Human Readability**: In contrast to binary formats, WKT strings are easily readable and understandable by developers and data analysts. This feature simplifies debugging and data validation processes.
- **Database Storage**: Numerous spatial databases, such as PostGIS, MySQL Spatial, and SQL Server Spatial, utilize or support WKT for the storage and querying of geometry data.

## Basic Syntax and Grammar

WKT follows a consistent hierarchical structure using keywords, parentheses, and coordinate lists. The basic grammar can be expressed as:

```
<geometry> ::= <POINT> | <LINESTRING> | <POLYGON> | <MULTIPOINT> | 
               <MULTILINESTRING> | <MULTIPOLYGON> | <GEOMETRYCOLLECTION>

<point> ::= "POINT" [<dimension>] "(" <x> <y> [<z>] [<m>] ")" | 
            "POINT" [<dimension>] "EMPTY"

<coordinate> ::= <x> <y> [<z>] [<m>]
<x> ::= <number>
<y> ::= <number>
<z> ::= <number>
<m> ::= <number>
```

- **Keywords**: Geometry type keywords (e.g., `POINT`, `LINESTRING`, `POLYGON`) are case-insensitive according to the specification, although conventional usage strongly favors uppercase letters.
- **Whitespace**: Whitespace (including spaces, tabs, and newlines) is used to separate elements and is generally flexible; however, one space is usually sufficient.
- **Numbers**: All coordinate values are represented as standard double-precision floating-point numbers.
- **Parentheses**: Parentheses are used to group coordinates for individual parts of a geometry and to group geometries within collections.
- **Commas**: Commas are employed to separate distinct parts or coordinates within a list.
- **Empty Geometries**: A geometry that contains no points can be represented by adding the term `EMPTY` after the geometry type, for example: `POINT EMPTY`, `LINESTRING EMPTY`.

### Coordinate System

WKT uses a right-handed coordinate system where:

- **X-axis**: Generally represents longitude or easting
- **Y-axis**: Generally represents latitude or northing
- **Z-axis**: Represents elevation or height (optional)
- **M-dimension**: Represents measure values (optional)

## Geometry Types

### Point

A point represents a single location in coordinate space.

**Syntax**: `POINT (x y [z] [m])`

**Examples**:

```
POINT (30 10)
POINT (30 10 15)
POINT Z (30 10 15)
POINT M (30 10 40)
POINT ZM (30 10 15 40)
POINT EMPTY
```

### LineString

A linestring represents a sequence of connected line segments.

**Syntax**: `LINESTRING (x1 y1 [z1] [m1], x2 y2 [z2] [m2], ...)`

**Examples**:

```
LINESTRING (30 10, 10 30, 40 40)
LINESTRING Z (30 10 15, 10 30 20, 40 40 25)
LINESTRING EMPTY
```

### Polygon

A polygon represents a closed area with an exterior boundary and optional interior holes.

**Syntax**: `POLYGON ((exterior_ring), (interior_ring1), (interior_ring2), ...)`

**Examples**:

```
POLYGON ((30 10, 40 40, 20 40, 10 20, 30 10))

POLYGON ((35 10, 45 45, 15 40, 10 20, 35 10),
         (20 30, 35 35, 30 20, 20 30))
```

### MultiPoint

A collection of points.

**Syntax**: `MULTIPOINT ((x1 y1), (x2 y2), ...)`

**Examples**:

```
MULTIPOINT ((10 40), (40 30), (20 20), (30 10))
MULTIPOINT (10 40, 40 30, 20 20, 30 10)
```

### MultiLineString

A collection of linestrings.

**Syntax**: `MULTILINESTRING ((linestring1), (linestring2), ...)`

**Examples**:

```
MULTILINESTRING ((10 10, 20 20, 10 40),
                 (40 40, 30 30, 40 20, 30 10))
```

### MultiPolygon

A collection of polygons.

**Syntax**: `MULTIPOLYGON (((polygon1)), ((polygon2)), ...)`

**Examples**:

```
MULTIPOLYGON (((30 20, 45 40, 10 40, 30 20)),
              ((15 5, 40 10, 10 20, 5 10, 15 5)))

MULTIPOLYGON (((40 40, 20 45, 45 30, 40 40)),
              ((20 35, 10 30, 10 10, 30 5, 45 20, 20 35),
               (30 20, 20 15, 20 25, 30 20)))
```

### GeometryCollection

A heterogeneous collection of geometries.

**Syntax**: `GEOMETRYCOLLECTION (geometry1, geometry2, ...)`

**Examples**:

```
GEOMETRYCOLLECTION (POINT (40 10),
                    LINESTRING (10 10, 20 20, 10 40),
                    POLYGON ((40 40, 20 45, 45 30, 40 40)))
```

## Extended Well Known Text (EWKT)

PostGIS introduced Extended Well Known Text (EWKT), which includes information about the spatial reference system. The SRID is an integer value that uniquely identifies the coordinate system for the geometry, such as EPSG:4326 for WGS84 latitude and longitude:

```
SRID=4326;POINT(-122.4194 37.7749)
SRID=3857;POLYGON((20037508.34 -20037508.34, 20037508.34 20037508.34, 
                  -20037508.34 20037508.34, -20037508.34 -20037508.34, 
                  20037508.34 -20037508.34))
```

## Coordinate Reference Systems

While WKT geometry doesn't inherently include coordinate reference system (CRS) information, it's often combined with CRS definitions. The OGC also defines WKT for CRS:

```
GEOGCS["WGS 84",
    DATUM["WGS_1984",
        SPHEROID["WGS 84",6378137,298.257223563,
            AUTHORITY["EPSG","7030"]],
        AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0,
        AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.01745329251994328,
        AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4326"]]
```

## Validation Rules

Valid WKT must conform to several rules:

1. **Closed Rings**: Linear rings in polygons must be closed, meaning the first and last points must be identical.
2. **Minimum Points**: LineStrings require at least 2 points, while linear rings require a minimum of 4 points.
3. **Orientation**: Exterior rings should be oriented counter-clockwise, and holes should be oriented clockwise.
4. **Consistency in Dimensions**: All coordinates within a geometry must maintain the same dimensionality.

## Advantages and Disadvantages of WKT

### Advantages:

- **Human-readable and editable**: This format is easy to inspect, debug, and manually create simple geometries.
- **Widely supported**: It has become a de facto standard for the exchange and storage of spatial data.
- **Simple grammar**: The structure is relatively straightforward to parse, especially for basic geometries.
- **Text-based**: Being text-based allows for easy storage in text files, transmission over networks, and use in environments where binary data might pose issues.

### Disadvantages:

- **Verbosity**: For complex geometries with numerous vertices (such as detailed coastlines or large polygons with multiple holes), Well-Known Text (WKT) strings can become excessively long and take up significant storage space compared to binary formats.
- **Parsing Overhead**: Text parsing is generally slower than binary parsing, leading to performance issues.
- **Floating-Point Precision**: As with any floating-point representation, there are limitations in precision that can result in subtle geometric errors or discrepancies if not managed carefully.