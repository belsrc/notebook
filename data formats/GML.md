---
tags:
  - data
  - gis
gardening: 🌳
date: 2025-10-29
reference:
  - https://www.ogc.org/standards/gml/
  - https://www.w3.org/TR/xlink/
  - http://schemas.opengis.net/gml/3.2.1/
  - https://www.isotc211.org/
---
## Geography Markup Language (GML) Overview

Geography Markup Language (GML) is an XML grammar for expressing geographical features, serving as both a modeling language for geographic systems and an open interchange format for geographic transactions on the Internet. GML is defined by the Open Geospatial Consortium (OGC) and adopted as ISO 19136:2007.

**Official Standards:**
- ISO 19136:2007 - Geography Markup Language (GML) 3.2
- ISO 19136-1:2020 - GML Part 1: Fundamentals
- ISO 19136-2:2015 - GML Part 2: Extended schemas and encoding rules
- OGC 07-036 (GML 3.2)

## Key Concepts

### Core Model

GML models the world using geographic features, which are abstractions of real-world phenomena associated with locations relative to Earth. Features are essentially lists of properties and geometries, where properties have name, type, and value descriptions, and geometries are composed of basic building blocks such as points, lines, curves, surfaces, and polygons.

### Two-Part Grammar

GML consists of two parts: the schema that describes the document structure and the instance document that contains the actual data.

```
┌─────────────────────────────────────┐
│      GML Application Schema         │
│   (defines feature types in XSD)    │
└─────────────────┬───────────────────┘
                  │ imports
                  ↓
┌─────────────────────────────────────┐
│         GML Core Schema             │
│  (defines primitives & base types)  │
└─────────────────┬───────────────────┘
                  │ referenced by
                  ↓
┌─────────────────────────────────────┐
│       GML Instance Document         │
│     (contains actual feature data)  │
└─────────────────────────────────────┘
```

## Version History

| Version | Year | Key Features |
|---------|------|-------------|
| GML 1.0 | 2000 | Initial release, based on RDF concepts |
| GML 2.0 | 2001 | Transition to XML Schema (XSD); gained widespread adoption in early WFS |
| GML 2.1 | 2002 | Enhanced feature schema |
| GML 3.0 | 2003 | Major expansion: added topology, coverages, and complex geometries |
| GML 3.1.1 | 2004 | OGC Standard. Stabilized the 3.0 model; widely used in WFS 1.1 |
| GML 3.2.1 | 2007 | First ISO Version (ISO 19136:2007). Unified OGC and ISO abstract models. Incompatible with 3.1.1 schemas. |
| GML 3.3 | 2012+ | Extends GML 3.2.1 with additional features (e.g., TINs, Linear Referencing) but does not replace the core 3.2 schema |

GML 3.2.1 (ISO 19136:2007) remains the widely deployed schema set, while ISO 19136-1:2020 and ISO 19136-2:2015 update and structure the standard text without replacing the 3.2.1 schemas.

## Core Components

### Geometric Primitives

GML defines geometric property elements to associate geometries with features, providing descriptive names encoded as common English terms.

**Basic Geometry Types:**
- `Point` - 0-dimensional position
- `LineString` - 1-dimensional curve composed of line segments
- `LinearRing` - Closed LineString
- `Polygon` - 2-dimensional surface with exterior and optional interior boundaries
- `Curve` - General 1-dimensional geometric primitive
- `Surface` - General 2-dimensional geometric primitive

**Aggregate Types (GML 3.2):**
- `MultiPoint`
- `MultiCurve` (multi-line concept)
- `MultiSurface` (multi-polygon concept)
- `MultiSolid`
- `MultiGeometry`

These correspond to the familiar multi-point, multi-line, and multi-polygon concepts, but use the GML names `MultiCurve` and `MultiSurface` rather than `MultiLineString` and `MultiPolygon`.

### Example: Point Feature

```xml
<?xml version="1.0" encoding="UTF-8"?>
<gml:FeatureCollection 
    xmlns:gml="http://www.opengis.net/gml/3.2"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.opengis.net/gml/3.2 
                        http://schemas.opengis.net/gml/3.2.1/gml.xsd"
    gml:id="fc1">
  
  <gml:boundedBy>
    <gml:Envelope srsName="urn:ogc:def:crs:EPSG::4326">
      <gml:lowerCorner>-180 -90</gml:lowerCorner>
      <gml:upperCorner>180 90</gml:upperCorner>
    </gml:Envelope>
  </gml:boundedBy>
  
  <gml:featureMember>
    <LocationFeature gml:id="loc1">
      <gml:name>Sample Point</gml:name>
      <gml:description>Example location feature</gml:description>
      <geometry>
        <gml:Point gml:id="p1" srsName="urn:ogc:def:crs:EPSG::4326">
          <gml:pos>41.6764 -86.2520</gml:pos>
        </gml:Point>
      </geometry>
    </LocationFeature>
  </gml:featureMember>
  
</gml:FeatureCollection>
```

### Example: Polygon Feature

```xml
<gml:Polygon gml:id="poly1" srsName="urn:ogc:def:crs:EPSG::4326">
  <gml:exterior>
    <gml:LinearRing>
      <gml:posList>
        0.0 0.0 
        10.0 0.0 
        10.0 10.0 
        0.0 10.0 
        0.0 0.0
      </gml:posList>
    </gml:LinearRing>
  </gml:exterior>
  <gml:interior>
    <gml:LinearRing>
      <gml:posList>
        2.0 2.0 
        8.0 2.0 
        8.0 8.0 
        2.0 8.0 
        2.0 2.0
      </gml:posList>
    </gml:LinearRing>
  </gml:interior>
</gml:Polygon>
```

## Schema Structure

The GML 3.2 schema is organized into multiple XSD files:

```
gml/
├── gml.xsd                    # Root schema document
├── gmlBase.xsd                # Base types and patterns
├── feature.xsd                # Feature definitions
├── geometryBasic0d1d.xsd      # Point, line geometries
├── geometryBasic2d.xsd        # Surface geometries
├── geometryPrimitives.xsd     # Primitive geometry types
├── geometryAggregates.xsd     # Multi-geometry types
├── geometryComplexes.xsd      # Complex geometry types
├── topology.xsd               # Topological relationships
├── coverage.xsd               # Coverage types
├── observation.xsd            # Observation & measurement
├── temporal.xsd               # Temporal primitives
├── temporalTopology.xsd       # Temporal relationships
├── temporalReferenceSystems.xsd
├── coordinateReferenceSystems.xsd
├── coordinateSystems.xsd
├── coordinateOperations.xsd
├── datums.xsd
├── referenceSystems.xsd
├── measures.xsd               # Units of measure
├── units.xsd
├── valueObjects.xsd           # Value types
├── dictionary.xsd             # Dictionary definitions
├── direction.xsd
├── grids.xsd                  # Grid geometries
├── dynamicFeature.xsd         # Time-varying features
└── basicTypes.xsd
```

## Application Schemas

GML supports community-specific application schemas that extend the base specification, allowing users to define domain-specific features like roads and bridges instead of generic points and lines.

Application schemas are designed using ISO 19103-conformant UML and then mapped to GML following rules in Annex E of ISO 19136.

### Notable Application Schemas:

- **AIXM** - Aeronautical Information eXchange Model
- **CityGML** - 3D city and regional models
- **IndoorGML** - Indoor navigation data
- **InfraGML** - Infrastructure information
- **CAAML** - Canadian Avalanche Association Markup Language
- **Coverages** - Spatio-temporally varying phenomena

## Object-Property-Value Rule

GML is founded on the Object-Property-Value rule, where geographic objects are defined by their properties, and properties have typed values.

```
Object (Feature)
  └── Property
        ├── name: string
        ├── type: datatype
        └── value: actual_value | geometry | remote_reference
```

## Coordinate Reference Systems

GML supports comprehensive CRS definitions using the `srsName` attribute:

```xml
<!-- Using EPSG URN (recommended for GML 3.2+) -->
<gml:Point srsName="urn:ogc:def:crs:EPSG::4326">
  <gml:pos>41.6764 -86.2520</gml:pos>
</gml:Point>

<gml:Point srsName="EPSG:4326">
  <gml:pos>-86.2520 41.6764</gml:pos>
</gml:Point>
```

**Important:** In GML 3.2, coordinate tuples are expected to follow the axis order defined by the referenced CRS. For EPSG:4326 this definition is latitude, longitude (y, x), even though many legacy systems and services have historically used longitude, latitude. Implementations must check and consistently apply the axis order associated with the `srsName` they use.

## Advanced Features

### Coverages

Coverages are features with a coverage function having a spatial domain and a value set range of homogeneous multi-dimensional tuples, modeling spatial distributions of phenomena.

### Observations

Observations model the act of observing with instruments or sensors, including the time of observation and the measured value.

### Topology

GML supports topological relationships between features, including:
- Node relationships
- Edge connectivity
- Face boundaries
- Solid containment

### Temporal Support

Temporal primitives include:
- Time instants
- Time periods
- Temporal positions
- Temporal reference systems

## Feature Collections

```xml
<gml:FeatureCollection gml:id="fc1">
  <gml:boundedBy> <!-- Recommended / often required by profiles -->
    <gml:Envelope>...</gml:Envelope>
  </gml:boundedBy>
  
  <gml:featureMember>
    <Feature1 gml:id="f1">...</Feature1>
  </gml:featureMember>
  
  <gml:featureMember xlink:href="http://example.com/feature2"/>
  
  <gml:featureMember>
    <Feature3 gml:id="f3">...</Feature3>
  </gml:featureMember>
</gml:FeatureCollection>
```

Feature members can either contain features inline or reference remote features using XLink attributes.

`gml:boundedBy` should provide the spatial extent of the collection where known and is required by some profiles, but the core GML 3.2 schema allows omission when no meaningful bounds can be supplied.

## XLink Integration

GML uses W3C XLink to create sophisticated links between resources, enabling remote property references and distributed feature storage.

```xml
<gml:featureMember xlink:href="http://example.com/features/road123" 
                   xlink:type="simple"/>
```

## Conformance Classes

GML 3.2 defines multiple conformance classes and profiles for application schemas, including:

- GML application schema fundamentals
- GML Simple Features Profile (Levels 0, 1, and 2)
- Feature collections
- Geometry primitives and aggregates
- Coordinate reference systems
- Temporal objects
- Coverage objects
- Observation and measurement

The exact grouping and numbering of conformance classes is specified in ISO 19136 and related OGC profile documents (for example, the Simple Features Profile and coverage application schema).

## Media Type

GML documents use the registered media type `application/gml+xml` with optional version parameter:

```
Content-Type: application/gml+xml; version=3.2
```

## Relationship to Other Standards

### KML

KML is an OGC encoding standard used to encode and transport geographic content for display in earth browsers and virtual globes such as Google Earth and other 2D/3D clients. Transforming rich GML data into KML typically involves loss of structural and semantic detail because KML is primarily focused on visualization and portrayal rather than full-featured data exchange.

### GeoJSON

GeoJSON is the lightweight alternative to GML for web development.

- **GML**: Used for enterprise data exchange, complex geometries (curves, surfaces), and strict schema validation (WFS).
- **GeoJSON**: Used for web APIs, mobile apps, and simple feature visualization. It does not support CRS (always WGS84) or advanced topologies.

### WFS (Web Feature Service)

WFS implementations read and write GML data, making GML the standard encoding for feature services.

### ISO 19100 Series

GML implements concepts from the ISO 19100 series of geographic information standards, particularly ISO 19107 (spatial schema).

## Key Design Principles

1. **Content-Presentation Separation** - GML separates geographic content from visualization
2. **Vendor Neutrality** - Open framework not tied to specific implementations
3. **Extensibility** - Application schemas extend core types
4. **Interoperability** - Based on international standards
5. **Text-Based** - Human-readable XML encoding
6. **Schema Validation** - Formal validation against XSD schemas

## Common Use Cases

- Geospatial data exchange between GIS systems
- Web Feature Service (WFS) responses
- Storing geographic features in XML databases
- Integrating spatial and non-spatial data
- City and infrastructure modeling
- Sensor observation data
- Coverage and raster data encoding

## Implementation Notes

### Validation

GML documents must validate against:
1. The referenced application schema (XSD)
2. The core GML schema
3. Additional Schematron constraints (optional)

### Namespaces

```xml
xmlns:gml="http://www.opengis.net/gml/3.2"
xmlns:xlink="http://www.w3.org/1999/xlink"
```

### Feature Identifiers

Most GML feature types are derived from `gml:AbstractGMLType`, which carries a required `gml:id` attribute of type `xs:ID`, and it is best practice for each feature instance to have a unique `gml:id` within the document. Application schemas or profiles can further constrain identifier requirements, but the core model distinguishes between `gml:id`, `gml:identifier`, and `gml:name`.

```xml
<Feature gml:id="unique_id_123">
```

## Tools and Software Support

GML is supported by major GIS vendors, database providers, and earth browser applications. Common tools include:

- GDAL/OGR - Reading and writing GML
- PostGIS - Spatial database with GML support
- GeoServer - WFS server producing GML
- ArcGIS - GML support via Data Interoperability extension
- FME - GML transformation and validation
