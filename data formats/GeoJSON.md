---
tags:
  - gis
  - data
gardening: 🌱
reference:
  - https://datatracker.ietf.org/doc/html/rfc7946
  - https://geojson.org/
  - https://en.wikipedia.org/wiki/GeoJSON
  - https://www.ogc.org/standard/features/
  - https://docs.mapbox.com/help/dive-deeper/geojson-coordinate-precision/
---
GeoJSON is a format for encoding geographic data structures using JavaScript Object Notation (JSON). Defined by RFC 7946, it provides a lightweight, text-based, and human-readable format for representing simple geographical features along with their non-spatial attributes. All GeoJSON coordinates use the WGS 84 geodetic datum (EPSG:4326) with coordinates in longitude-latitude order.

### Why GeoJSON?

GeoJSON has become the de facto standard for web-based geospatial applications due to several characteristics:

1. **Web-Native**: As valid JSON, GeoJSON integrates seamlessly with JavaScript environments and RESTful APIs without parsing overhead.
2. **Human Readable**: Text-based format enables inspection, debugging, and manual editing.
3. **Simplicity**: Minimal specification with only seven geometry types and two container types.
4. **Interoperability**: Supported by virtually every GIS tool, mapping library, and spatial database.
5. **Self-Describing**: Unlike WKB, GeoJSON documents are interpretable without external schema definitions.

## Basic Syntax and Grammar

GeoJSON follows JSON syntax with a hierarchical structure using type discriminators. The grammar can be expressed as:

```
<geojson>     ::= <geometry> | <feature> | <feature_collection>

<geometry>    ::= <point> | <multipoint> | <linestring> | <multilinestring> 
                | <polygon> | <multipolygon> | <geometrycollection>

<point>       ::= { "type": "Point", "coordinates": <position> }
<linestring>  ::= { "type": "LineString", "coordinates": [<position>, ...] }
<polygon>     ::= { "type": "Polygon", "coordinates": [[<position>, ...], ...] }

<position>    ::= [<longitude>, <latitude>] | [<longitude>, <latitude>, <altitude>]
<longitude>   ::= <number>  // range: -180 to 180
<latitude>    ::= <number>  // range: -90 to 90
<altitude>    ::= <number>  // meters above WGS 84 ellipsoid

<feature>     ::= { "type": "Feature", "geometry": <geometry> | null, 
                   "properties": <object> | null [, "id": <string> | <number>] }

<feature_collection> ::= { "type": "FeatureCollection", "features": [<feature>, ...] }
```

### Coordinate Order

GeoJSON mandates longitude-first ordering, which differs from many cartographic conventions:

```
Position Array: [longitude, latitude, altitude?]
                     ↑          ↑         ↑
                 x-axis      y-axis   optional z
                -180/180     -90/90    meters
```

This follows the mathematical convention of $(x, y, z)$ rather than the geographic convention of latitude-longitude.

## Core Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                       GeoJSON Object                            │
├─────────────────────────────────────────────────────────────────┤
│  "type": string (required)                                      │
│  "bbox": [west, south, east, north] (optional)                  │
├───────────────────┬───────────────────┬─────────────────────────┤
│     Geometry      │      Feature      │   FeatureCollection     │
├───────────────────┼───────────────────┼─────────────────────────┤
│ "coordinates": [] │ "geometry": {}    │ "features": []          │
│                   │ "properties": {}  │                         │
│                   │ "id": string|num  │                         │
└───────────────────┴───────────────────┴─────────────────────────┘
```

## Geometry Types

### Point

A Point represents a single position in coordinate space.

**Structure:**

```
┌──────────────────────────────────────────┐
│ Point                                    │
├──────────────────────────────────────────┤
│ type: "Point"                            │
│ coordinates: [longitude, latitude, z?]   │
└──────────────────────────────────────────┘
```

**Example:**

```json
{
  "type": "Point",
  "coordinates": [-86.2520, 41.6764]
}
```

### MultiPoint

A MultiPoint represents multiple discrete positions.

**Structure:**

```
┌──────────────────────────────────────────┐
│ MultiPoint                               │
├──────────────────────────────────────────┤
│ type: "MultiPoint"                       │
│ coordinates: [                           │
│   [lon₁, lat₁],                          │
│   [lon₂, lat₂],                          │
│   ...                                    │
│ ]                                        │
└──────────────────────────────────────────┘
```

**Example:**

```json
{
  "type": "MultiPoint",
  "coordinates": [
    [-86.2520, 41.6764],
    [-87.6298, 41.8781],
    [-84.5120, 39.1031]
  ]
}
```

### LineString

A LineString represents a sequence of two or more positions forming a continuous path.

**Structure:**

```
┌──────────────────────────────────────────┐
│ LineString                               │
├──────────────────────────────────────────┤
│ type: "LineString"                       │
│ coordinates: [                           │
│   [lon₁, lat₁],  ─────┐                  │
│   [lon₂, lat₂],  ◄────┘                  │
│   [lon₃, lat₃],  ◄─────connected path    │
│   ...                                    │
│ ]                                        │
│ Minimum: 2 positions                     │
└──────────────────────────────────────────┘
```

**Example:**

```json
{
  "type": "LineString",
  "coordinates": [
    [-86.2520, 41.6764],
    [-87.6298, 41.8781],
    [-88.0000, 42.0000]
  ]
}
```

### MultiLineString

A MultiLineString represents multiple LineString geometries.

**Example:**

```json
{
  "type": "MultiLineString",
  "coordinates": [
    [[-86.25, 41.67], [-87.62, 41.87]],
    [[-84.51, 39.10], [-83.00, 39.96]]
  ]
}
```

### Polygon

A Polygon represents a closed area defined by one exterior ring and zero or more interior rings (holes). Each ring is a closed LineString with identical first and last positions.

**Structure:**

```
┌──────────────────────────────────────────────────────────────┐
│ Polygon                                                      │
├──────────────────────────────────────────────────────────────┤
│ type: "Polygon"                                              │
│ coordinates: [                                               │
│   [ exterior_ring ],    ← counter-clockwise (CCW)            │
│   [ hole_1 ],           ← clockwise (CW)                     │
│   [ hole_2 ],           ← clockwise (CW)                     │
│   ...                                                        │
│ ]                                                            │
│ Each ring: minimum 4 positions, first == last                │
└──────────────────────────────────────────────────────────────┘
```

**Winding Order (Right-Hand Rule):**

```
  Exterior Ring (CCW)                    Hole/Interior Ring (CW)
  
  v₄ ←←←←←←←←←← v₃                       v₂ →→→→→→→→→→ v₃
   │             ↑                        │             ↑
   │   interior  │                        │    hole     │
   ↓             │                        ↓   (excluded)│
  v₁ →→→→→→→→→→ v₂                       v₁ ←←←←←←←←←← v₄
  
  Traversal: v₁→v₂→v₃→v₄→v₁              Traversal: v₁→v₂→v₃→v₄→v₁
  Direction: Counter-clockwise           Direction: Clockwise
  Interior to LEFT of path               Hole area to LEFT of path
```

When walking along the boundary, the area to your LEFT is the interior. For exterior rings this means CCW includes the polygon area; for holes this means CW excludes the hole area.

**Example with hole:**

```json
{
  "type": "Polygon",
  "coordinates": [
    [
      [-86.30, 41.60], [-86.30, 41.75], [-86.15, 41.75],
      [-86.15, 41.60], [-86.30, 41.60]
    ],
    [
      [-86.27, 41.63], [-86.20, 41.63], [-86.20, 41.70],
      [-86.27, 41.70], [-86.27, 41.63]
    ]
  ]
}
```

### MultiPolygon

A MultiPolygon represents multiple Polygon geometries, where each polygon follows the same exterior/interior ring rules.

**Example:**

```json
{
  "type": "MultiPolygon",
  "coordinates": [
    [
      [[-86.30, 41.60], [-86.30, 41.75], [-86.15, 41.75], [-86.15, 41.60], [-86.30, 41.60]]
    ],
    [
      [[-87.70, 41.80], [-87.70, 41.95], [-87.55, 41.95], [-87.55, 41.80], [-87.70, 41.80]]
    ]
  ]
}
```

### GeometryCollection

A GeometryCollection contains heterogeneous geometry objects.

**Example:**

```json
{
  "type": "GeometryCollection",
  "geometries": [
    { "type": "Point", "coordinates": [-86.2520, 41.6764] },
    { "type": "LineString", "coordinates": [[-86.25, 41.67], [-87.62, 41.87]] }
  ]
}
```

## Feature and FeatureCollection

### Feature

A Feature wraps a geometry with associated properties and an optional identifier.

**Structure:**

```
┌─────────────────────────────────────────────────────────────┐
│ Feature                                                     │
├─────────────────────────────────────────────────────────────┤
│ type: "Feature"                            (required)       │
│ geometry: Geometry | null                  (required)       │
│ properties: { key: value, ... } | null     (required)       │
│ id: string | number                        (optional)       │
│ bbox: [west, south, east, north]           (optional)       │
└─────────────────────────────────────────────────────────────┘
```

**Example:**
```json
{
  "type": "Feature",
  "id": "building-001",
  "geometry": {
    "type": "Point",
    "coordinates": [-86.2520, 41.6764]
  },
  "properties": {
    "name": "Main Building",
    "height": 45.5,
    "floors": 12,
    "built": 1985
  }
}
```

### FeatureCollection

A FeatureCollection aggregates multiple Feature objects.

**Example:**

```json
{
  "type": "FeatureCollection",
  "bbox": [-87.70, 39.10, -83.00, 42.00],
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [-86.25, 41.67] },
      "properties": { "name": "Location A" }
    },
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [-87.62, 41.87] },
      "properties": { "name": "Location B" }
    }
  ]
}
```

## Bounding Box

The optional `bbox` member defines the spatial extent of a GeoJSON object.

**2D Format:** `[west, south, east, north]` or `[minLon, minLat, maxLon, maxLat]`

**3D Format:** `[west, south, minAlt, east, north, maxAlt]`

```
        north (maxLat)
           ┌─────────────────┐
           │                 │
west       │    Features     │  east
(minLon)   │                 │  (maxLon)
           │                 │
           └─────────────────┘
        south (minLat)
```

## Validation Rules

GeoJSON documents must conform to these constraints:

| Rule | Description |
|------|-------------|
| **CRS** | WGS 84 (EPSG:4326) only; `crs` member is deprecated and must be ignored |
| **Coordinate Order** | `[longitude, latitude]` — longitude first |
| **Coordinate Range** | Longitude: $[-180, 180]$, Latitude: $[-90, 90]$ |
| **Precision** | 6 decimal places ≈ 10cm accuracy; more is rarely useful |
| **Ring Closure** | First and last positions of linear rings must be identical |
| **Ring Minimum** | Linear rings require at least 4 positions |
| **Winding Order** | Exterior rings: counter-clockwise; Holes: clockwise |
| **Antimeridian** | Geometries crossing ±180° should be split |
| **Foreign Members** | Parsers must ignore unknown members |

## Comparison with Alternative Formats

### Encoding Comparison

| Aspect | GeoJSON | WKT | WKB | GML |
|--------|---------|-----|-----|-----|
| Format | Text (JSON) | Text | Binary | Text (XML) |
| Human Readable | Yes | Yes | No | Verbose |
| Size Efficiency | Low | Medium | High | Very Low |
| Parse Speed | Medium | Medium | Fast | Slow |
| Web Native | Yes | No | No | No |
| Schema Required | No | No | No | Yes (XSD) |
| CRS Support | WGS84 only | Via EWKT | Via EWKB | Full |

### Size Comparison

For a polygon with 100 vertices:

| Format | Approximate Size |
|--------|------------------|
| GeoJSON | ~3,500 bytes |
| WKT | ~2,500 bytes |
| WKB | ~1,609 bytes |
| GML | ~5,000+ bytes |

### Feature Comparison

| Feature | GeoJSON | WKT/WKB | GML |
|---------|---------|---------|-----|
| Properties/Attributes | Native | External | Native |
| Collections | Yes | Limited | Yes |
| Topology | No | No | Yes |
| Temporal | No | No | Yes |
| 3D Support | Basic (altitude) | Full (Z/M) | Full |
| Styling | No | No | Via SLD |

## Advantages and Disadvantages

### Advantages

- **Universal Browser Support**: Native JSON parsing in all JavaScript environments
- **API Integration**: Direct serialization/deserialization with REST APIs
- **Readable**: Easy to inspect, debug, and manually edit
- **Simple Specification**: Complete spec fits in RFC 7946 (~30 pages)
- **Ecosystem**: Vast library support (Turf.js, MapLibre, Leaflet, D3, deck.gl)
- **Schema-less**: Flexible properties without predefined structure
- **Streaming**: Can be processed incrementally with JSON stream parsers

### Disadvantages

- **Verbosity**: Larger file sizes compared to binary formats
- **Limited CRS**: WGS84 only; other coordinate systems require transformation
- **No Topology**: Cannot represent shared boundaries or connectivity
- **Precision Loss**: JSON number encoding may lose precision beyond ~15 significant digits
- **No Styling**: Requires external specifications (e.g., MapLibre style spec)
- **Large Files**: Not suitable for datasets exceeding ~100MB without streaming

## Relationship to Other Standards

### OGC Simple Features

GeoJSON geometry types correspond to OGC Simple Features Access (ISO 19125):

```
OGC Simple Features          GeoJSON
─────────────────────────────────────
ST_Point                  →  Point
ST_LineString             →  LineString
ST_Polygon                →  Polygon
ST_MultiPoint             →  MultiPoint
ST_MultiLineString        →  MultiLineString
ST_MultiPolygon           →  MultiPolygon
ST_GeometryCollection     →  GeometryCollection
```

### TopoJSON

TopoJSON is an extension that encodes topology, eliminating redundancy for shared boundaries:

```
GeoJSON (redundant borders)      TopoJSON (shared arcs)
┌─────┬─────┐                    ┌─────┬─────┐
│  A  │  B  │                    │  A  │  B  │
│     │     │                    │     │     │
└─────┴─────┘                    └──┬──┴──┬──┘
    ↑   ↑                           │     │
 border border                    shared arc
 stored stored                    stored once
 twice  twice
```

### GeoJSON-LD

GeoJSON-LD adds JSON-LD context for semantic web compatibility, enabling RDF graph representation.

### GeoJSON-T

Proposed extension for temporal data, adding `when` properties for time-varying features.

## TypeScript Type Definitions

```typescript
type Position2D = [longitude: number, latitude: number];
type Position3D = [longitude: number, latitude: number, altitude: number];
type Position = Position2D | Position3D;

type BBox2D = [west: number, south: number, east: number, north: number];
type BBox3D = [west: number, south: number, minAlt: number, 
               east: number, north: number, maxAlt: number];
type BBox = BBox2D | BBox3D;

type Point = {
  readonly type: "Point";
  readonly coordinates: Position;
  readonly bbox?: BBox;
};

type MultiPoint = {
  readonly type: "MultiPoint";
  readonly coordinates: readonly Position[];
  readonly bbox?: BBox;
};

type LineString = {
  readonly type: "LineString";
  readonly coordinates: readonly Position[];
  readonly bbox?: BBox;
};

type MultiLineString = {
  readonly type: "MultiLineString";
  readonly coordinates: readonly (readonly Position[])[];
  readonly bbox?: BBox;
};

type Polygon = {
  readonly type: "Polygon";
  readonly coordinates: readonly (readonly Position[])[];
  readonly bbox?: BBox;
};

type MultiPolygon = {
  readonly type: "MultiPolygon";
  readonly coordinates: readonly (readonly (readonly Position[])[])[];
  readonly bbox?: BBox;
};

type Geometry =
  | Point
  | MultiPoint
  | LineString
  | MultiLineString
  | Polygon
  | MultiPolygon
  | GeometryCollection;

type GeometryCollection = {
  readonly type: "GeometryCollection";
  readonly geometries: readonly Geometry[];
  readonly bbox?: BBox;
};

type Feature<
  G extends Geometry | null = Geometry | null,
  P extends Record | null = Record | null
> = {
  readonly type: "Feature";
  readonly geometry: G;
  readonly properties: P;
  readonly id?: string | number;
  readonly bbox?: BBox;
};

type FeatureCollection<
  G extends Geometry | null = Geometry | null,
  P extends Record | null = Record | null
> = {
  readonly type: "FeatureCollection";
  readonly features: readonly Feature[];
  readonly bbox?: BBox;
};

type GeoJSON = Geometry | Feature | FeatureCollection;
```

## Common Use Cases

- **Web Mapping**: Leaflet, MapLibre GL, OpenLayers, deck.gl layer data
- **API Responses**: REST endpoints returning spatial query results
- **Data Exchange**: Transferring features between GIS applications
- **Spatial Databases**: PostGIS `ST_AsGeoJSON()`, MongoDB geospatial queries
- **Static Assets**: Map overlays, boundary files, point-of-interest datasets
- **Real-time Data**: Vehicle tracking, sensor networks, IoT location streams

## Implementation Notes

### Coordinate Precision

Precision beyond 6 decimal places rarely adds value:

| Decimal Places | Precision | Use Case |
|----------------|-----------|----------|
| 0 | ~111 km | Country/large region |
| 1 | ~11.1 km | Large city |
| 2 | ~1.11 km | Town/village |
| 3 | ~111 m | Neighborhood/street |
| 4 | ~11.1 m | Individual street/building |
| 5 | ~1.11 m | Individual tree/house |
| 6 | ~111 mm | Precise surveying |
| 7+ | ~11 mm | Specialized surveying |

### Antimeridian Handling

Geometries crossing the antimeridian (±180° longitude) should be split:

```
           ─180°                    +180°
              │                       │
    ┌─────────┼───────────────────────┼─────────┐
    │  Part B │        Invalid        │  Part A │
    └─────────┼───────────────────────┼─────────┘
              │                       │
    
    Split into two geometries that don't cross the line.
```

## Media Type

GeoJSON uses the registered media type:

```
Content-Type: application/geo+json
```

The legacy `application/vnd.geo+json` and `application/json` are also commonly accepted.
