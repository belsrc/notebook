---
tags:
  - gis
gardening: 🌳
reference:
  - https://mapbox.github.io/vector-tile-spec/
  - https://wiki.osgeo.org/wiki/Tile_Map_Service_Specification
  - https://www.ogc.org/standards/wcs/
  - https://www.ogc.org/standards/wfs/
  - https://www.ogc.org/standards/wmts/
  - https://www.ogc.org/standards/wms/
---
## Web Map Service (WMS)

WMS generates map images on-demand from geospatial data sources. The server renders styled maps based on client parameters and returns static images

**Standard**: OGC WMS 1.3.0 (ISO 19128)
**Purpose**: Server-rendered raster images

| Format      | MIME Type           | Description                           | Use Case                    |
|-------------|---------------------|---------------------------------------|-----------------------------|
| PNG         | image/png           | Lossless, supports transparency       | Overlay layers              |
| JPEG        | image/jpeg          | Lossy compression, no transparency    | Aerial imagery, basemaps    |
| GIF         | image/gif           | Limited colors, supports transparency | Legacy support              |
| TIFF        | image/tiff          | High quality, geospatial metadata     | Print, analysis             |
| GeoTIFF     | image/geotiff       | TIFF with embedded georeferencing     | GIS software integration    |
| WebP        | image/webp          | Modern compression, transparency      | Modern browsers             |
| SVG         | image/svg+xml       | Vector graphics (rare for WMS)        | Cartographic quality        |

**Characteristics:**
- Dynamic rendering on every request
- No caching at the protocol level
- Supports arbitrary bounding boxes and sizes
- High server computational cost
- Flexible styling via SLD (Styled Layer Descriptor)

**Use Cases:**
- Dynamic thematic mapping with frequently changing data
- Custom analysis visualizations requiring real-time rendering
- Print-quality map generation at specific scales
- Situations where caching is not beneficial (e.g., real-time sensor data)

## Web Map Tile Service (WMTS)

WMTS serves pre-rendered, cached map tiles at fixed scales and tile boundaries. This is the OGC's standardized approach to tile-based mapping.

**Standard**: OGC WMTS 1.0.0
**Purpose**: Pre-rendered, cached raster tiles

| Format | MIME Type      | Description                        | Use Case                     |
|--------|----------------|------------------------------------|------------------------------|
| PNG    | image/png      | Standard web tiles, transparency   | Most common for overlays     |
| JPEG   | image/jpeg     | Compressed, no transparency        | Satellite imagery, basemaps  |
| PNG8   | image/png      | 8-bit PNG, smaller file size       | Simple color palettes        |
| WebP   | image/webp     | Efficient compression              | Modern tile servers          |
| JPEG2000| image/jp2     | Advanced compression (rare)        | High-quality imagery         |

**Characteristics**:
- Fixed tile boundaries aligned to grid
- Pre-computed at specific zoom levels
- Highly cacheable (CDN-friendly)
- Fast delivery (static file serving)
- Limited flexibility in styling and scale

**Use Cases**:
- Basemaps with static or infrequently changing data
- High-traffic applications requiring fast tile delivery
- Mobile applications with limited bandwidth
- Any scenario where CDN caching provides value

## Tile Map Service (TMS)

TMS is a de facto standard for serving map tiles, predating WMTS. It uses a simpler URL structure with inverted Y-axis compared to WMTS.

**Standard**: OSGeo TMS Specification (community standard, not OGC)
**Purpose**: Pre-rendered raster tiles (community standard)

| Format | Extension | Description                    | Use Case                 |
|--------|-----------|--------------------------------|--------------------------|
| PNG    | .png      | Default for most tile servers  | General purpose          |
| JPEG   | .jpg      | Satellite and aerial imagery   | Large basemaps           |
| WebP   | .webp     | Modern browsers                | Bandwidth optimization   |
| MVT    | .pbf      | Vector tiles (if supported)    | Client-side rendering    |

**Characteristics**:
- Simpler URL structure than WMTS
- Widely supported by open-source tools
- Less formal specification
- Y-axis origin requires careful handling

**Use Cases**:
- Open-source mapping stacks (OpenStreetMap ecosystem)
- Custom tile servers (TileStache, TileServer GL)
- Legacy applications built before WMTS standardization

## Web Feature Service (WFS)

WFS provides transactional access to geographic features, returning vector geometry and attributes in formats like GML, GeoJSON, or Shapefile.

**Standard**: OGC WFS 2.0.0 (ISO 19142)
**Purpose**: Vector feature data with query capabilities

| Format         | MIME Type                    | Description                          | Use Case                     |
|----------------|------------------------------|--------------------------------------|------------------------------|
| GML 2.x        | text/xml; subtype=gml/2.1.2 | OGC Geography Markup Language v2     | Legacy OGC compatibility     |
| GML 3.x        | text/xml; subtype=gml/3.1.1 | OGC Geography Markup Language v3     | OGC standard workflows       |
| GML 3.2        | text/xml; subtype=gml/3.2.1 | Latest GML specification             | Modern OGC implementations   |
| GeoJSON        | application/json             | IETF standard JSON format            | Web applications (most common)|
| GeoJSON        | application/geo+json         | Official GeoJSON MIME type           | Strict standard compliance   |
| ESRI GeoJSON   | application/json             | Esri's GeoJSON variant               | ArcGIS integration           |
| Shapefile      | application/zip              | ESRI Shapefile (.shp + sidecar files)| Desktop GIS (QGIS, ArcGIS)   |
| CSV            | text/csv                     | Point data as comma-separated        | Spreadsheet integration      |
| KML            | application/vnd.google-earth.kml+xml | Google Earth format         | Google Earth, mapping apps   |
| JSON           | application/json             | Simple JSON (properties only)        | Attribute-only queries       |
| JSONP          | text/javascript              | JSON with padding (callback)         | Cross-origin requests (legacy)|
| GeoPackage     | application/geopackage+sqlite3| SQLite-based format                 | Mobile, offline applications |

**Characteristics**:
- Returns raw vector geometry and attributes
- Supports complex spatial and attribute queries
- Transactional capabilities (WFS-T)
- Protocol overhead for large datasets
- Client-side rendering required

**Use Cases**:
- Feature editing applications (WFS-T)
- Spatial analysis requiring geometry access
- Dynamic filtering and querying
- Integration with desktop GIS (QGIS, ArcGIS)
- Downstream processing pipelines

## Vector Tile Service (VTS)/Mapbox Vector Tile (MVT)

Vector tile services deliver pre-processed vector data in compact binary format (Protocol Buffers), tiled similarly to raster services. MVT (Mapbox Vector Tiles) is the predominant format.

**Standard**: Mapbox Vector Tile Specification 2.1 (community standard)
**Purpose**: Tiled vector data for client-side rendering

| Format                   | MIME Type                          | Extension | Description                        |
| ------------------------ | ---------------------------------- | --------- | ---------------------------------- |
| Mapbox Vector Tile (MVT) | application/x-protobuf             | .pbf      | Protocol Buffers binary (standard) |
| Mapbox Vector Tile       | application/vnd.mapbox-vector-tile | .mvt      | Alternative MIME type              |
| GeoJSON tiles*           | application/json                   | .geojson  | JSON tiles (not true VTS)          |
| TopoJSON tiles*          | application/json                   | .topojson | Topology-preserving JSON (rare)    |

**Characteristics**:
- Compact binary format (Protocol Buffers)
- Client-side styling and rendering
- Queryable features (click handlers, tooltips)
- Smooth zoom between discrete levels
- Efficient geometry simplification per zoom

**Use Cases**:
- Interactive basemaps with dynamic styling
- Data-driven visualizations requiring feature access
- Mobile applications (smaller payload than raster)
- Applications requiring feature interactivity
- Multi-resolution datasets

## Web Coverage Service (WCS)

WCS provides access to raw raster data (coverages) with original values intact, suitable for scientific analysis rather than visualization. Unlike WMS, which returns styled images, WCS returns the underlying data.

**Standard**: OGC WCS 2.1
**Purpose**: Raw raster data access for analysis

| Format      | MIME Type                | Description                              | Use Case                        |
|-------------|--------------------------|------------------------------------------|---------------------------------|
| GeoTIFF     | image/tiff               | TIFF with georeferencing                 | Most common, GIS analysis       |
| NetCDF      | application/netcdf       | Network Common Data Form                 | Climate/weather data            |
| HDF         | application/x-hdf        | Hierarchical Data Format                 | Scientific datasets             |
| GRIB2       | application/x-grib2      | Gridded Binary (meteorological)          | Weather forecasting             |
| JPEG2000    | image/jp2                | Wavelet compression                      | Large imagery archives          |
| PNG         | image/png                | Web-friendly (loses precision)           | Visualization only              |
| ASCII Grid  | text/plain               | Plain text grid values                   | Simple data exchange            |
| CoverageJSON| application/json         | JSON coverage format (emerging)          | Web applications                |
| Zarr        | application/zarr         | Cloud-optimized array storage            | Cloud-native workflows          |

**Characteristics**:
- Returns original data values, not styled images
- Supports multi-band rasters
- Subsetting and reprojection capabilities
- Scientific data formats (GeoTIFF, NetCDF, HDF)
- Preserves data precision

**Use Cases**:
- Elevation data access (DEM/DTM)
- Satellite imagery analysis
- Climate and weather data distribution
- Scientific computing workflows
- Machine learning training data

## Service Comparison Matrix

| Service | Data Type | Caching | Styling | Query/Edit | Tile-based | Client GPU |
|---------|-----------|---------|---------|------------|------------|------------|
| WMS     | Raster    | Low     | Server  | No         | No         | No         |
| WMTS    | Raster    | High    | Server  | No         | Yes        | No         |
| TMS     | Raster    | High    | Server  | No         | Yes        | No         |
| WFS     | Vector    | Low     | Client  | Yes        | No         | Optional   |
| VTS/MVT | Vector    | High    | Client  | Limited    | Yes        | Yes        |
| WCS     | Raster    | Low     | None    | No         | No         | No         |