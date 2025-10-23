---
tags:
  - gis
gardening: 🌿
date: 2025-08-20
reference:
  - https://en.wikipedia.org/wiki/Universal_Transverse_Mercator_coordinate_system
  - https://en.wikipedia.org/wiki/Web_Mercator_projection
  - https://gis.utah.gov/blog/2021-06-23-choosing-right-transformation/
  - https://gis.utah.gov/blog/2015-12-21-nad83-and-webmercator-projections/
  - https://epsg.io/3857
  - https://en.wikipedia.org/wiki/World_Geodetic_System
  - https://geoinfospot.com/coordinate-systems-explained-wgs84-vs-utm/
  - https://www.fws.gov/r7/nwr/Realty/data/LandMappers/Public/Help/HTML/R7-Public-Land-Mapper-Help.html?Datumsprojectionsandcoordinatesy.html
  - https://docs.qgis.org/3.34/en/docs/gentle_gis_introduction/coordinate_reference_systems.html#coordinate-reference-system-crs-in-detail
---
Coordinate systems provide the mathematical foundation for all Geographic Information Systems (GIS). A proper understanding of how locations are defined and referenced on the Earth's surface is essential; without it, spatial data loses its meaning, measurements may become inaccurate, and it becomes difficult to integrate different datasets effectively.

Earth is not a perfect sphere; rather, it is an oblate spheroid, meaning it is flattened at the poles and bulges at the equator due to rotational forces. One of the main challenges for GIS practitioners is to accurately represent locations on this irregular, three-dimensional surface using mathematical models that can be processed by computers and displayed on flat screens or paper maps.

To address this challenge, geodesists and cartographers have developed a hierarchy of mathematical constructs:

1. [Ellipsoid](./Ellipsoid.md)
2. [Datum](./Datum.md)
3. Geographic Coordinate System
4. Projected Coordinate System

## Geographic Coordinate Systems (GCS)

A Geographic Coordinate System (GCS) defines locations on the Earth's surface using angular measurements of latitude and longitude, which are referenced to a three-dimensional ellipsoidal model. In contrast to projected coordinate systems that utilize linear units such as meters or feet, GCS coordinates are expressed in degrees, minutes, and seconds or in decimal degrees.

### Mathematical Components

A complete geographic coordinate system (GCS) specification includes four essential components:

1. **Ellipsoid (Spheroid):** A mathematical model that approximates the shape of the Earth.
2. **Datum:** The ellipsoid positioned in relation to the Earth's center of mass.
3. **Prime Meridian:** The reference line for longitude, typically located at Greenwich (0°).
4. **Angular Units:** Usually expressed in degrees for coordinate representation.

### Coordinate Representation

Geographic coordinates are expressed as:

- **Latitude ($\phi$)**: Angular distance north or south from the Equator (-90° to +90°)
- **Longitude ($\lambda$)**: Angular distance east or west from the Prime Meridian (-180° to +180°)

```
Latitude: 40.7589° N
Longitude: 73.9851° W
```

### Limitations of Geographic Coordinates

Geographic coordinates present significant challenges for distance and area calculations because the unit of length (degrees) is not held constant for longitude except along parallels—individual perfectly east-west lines. This variability occurs because:

- Meridians (longitude lines) converge at the poles
- One degree of longitude represents different distances at different latitudes
- The relationship follows: $\text{Distance} = \cos(\text{latitude}) \times \text{degree\_length}$

## World Geodetic System 1984 (WGS84)

WGS84 is a globally defined geodetic datum developed and maintained by the United States National Geospatial-Intelligence Agency (NGA). It is consistent with the International Terrestrial Reference Frame (ITRF) to within 1 centimeter. The system has evolved from earlier iterations:

- **WGS60 (1960):** The first global system that combined satellite, gravity, and survey data.
- **WGS66 (1966):** Improved integration of satellite data.
- **WGS72 (1972):** Enhanced with Doppler satellite data.
- **WGS84 (1984):** The current standard, which incorporates extensive measurements from GPS satellites.

### Technical Specifications

The parameters of the WGS84 ellipsoid are defined as follows:

- **Semi-major axis (a):** 6,378,137.0 meters
- **Flattening (f):** 1/298.257223563
- **Semi-minor axis (b):** 6,356,752.314245 meters

The error associated with WGS84 is believed to be less than 2 centimeters to the center of mass, which is more precise than regional datums such as NAD83

### EPSG Code and Technical Implementation

WGS84 is a geographic coordinate system identified by EPSG:4326. It uses an ellipsoidal 2D coordinate system with latitude and longitude axes oriented north and east, respectively, measured in degrees.

The well-known text (WKT) representation:

```
GEOGCS["WGS 84",
  DATUM["WGS_1984",
    SPHEROID["WGS 84",6378137,298.257223563,
      AUTHORITY["EPSG","7030"]],
    AUTHORITY["EPSG","6326"]],
  PRIMEM["Greenwich",0,
    AUTHORITY["EPSG","8901"]],
  UNIT["degree",0.0174532925199433,
    AUTHORITY["EPSG","9122"]],
  AUTHORITY["EPSG","4326"]]
```

### GPS Integration

The Global Positioning System (GPS) relies on WGS84 as its reference coordinate system. This system includes a reference ellipsoid, a standard coordinate system, altitude data, and a geoid. The integration of WGS84 with GPS provides several advantages, including: 
- Global positioning accuracy within centimeters
- Consistent coordinate references across GPS devices
- Seamless integration with satellite-based positioning systems

### Dynamic Nature

WGS84 is a dynamic datum, which means that when compared to fixed points on the ground, the coordinates of these points will change over time due to tectonic motion and variations in mass distribution within the Earth's interior.

## Universal Transverse Mercator (UTM)

### System Architecture and Design

The Universal Transverse Mercator (UTM) system divides the Earth into 60 zones, each 6° of longitude wide. Zone 1 covers longitudes from 180° to 174° W, and zone numbering increases eastward to zone 60, which spans longitudes from 174° E to 180°.

```
┌─────────────────────────────────────────────────────┐
│                    UTM Zone Layout                  │
├─────────────────────────────────────────────────────┤
│ Zone 1:  180°W to 174°W  │ Zone 31: 0° to 6°E       │
│ Zone 2:  174°W to 168°W  │ Zone 32: 6°E to 12°E     │
│ ...                      │ ...                      │
│ Zone 30: 6°W to 0°       │ Zone 60: 174°E to 180°E  │
└─────────────────────────────────────────────────────┘
```

### Mathematical Foundation

Each of the 60 zones utilizes a transverse Mercator projection, which is effective for mapping regions with significant north-south extent while minimizing distortion. By dividing the zones into narrow segments of 6° of longitude and reducing the scale factor along the central meridian to 0.9996, the distortion is kept below 1 part in 1,000 within each zone.

### Coordinate Representation

UTM coordinates are expressed in meters:

- **Easting (X)**: Distance east from the zone's central meridian
- **Northing (Y)**: Distance north from the Equator (Northern Hemisphere) or south from 10,000,000m (Southern Hemisphere)

Example UTM coordinate:

```
Zone: 17N
Easting: 630,084 m
Northing: 4,833,438 m
Full designation: "17N 630084 4833438"
```

### Advantages for Regional Analysis

UTM NAD83 is the ideal coordinate system for collecting and analyzing GIS data across the state, particularly for business needs that involve length, area, or geoprocessing. It effectively preserves the shapes of real-world objects modeled in your GIS. Key benefits include: 

- Use of linear units (meters), which simplifies distance calculations
- Minimal distortion within individual zones
- Applicability of the Pythagorean theorem for accurate distance measurements
- Suitability for engineering and surveying applications

### Datum Relationships

While UTM defines the projection method, the underlying datum varies by region:

- **UTM NAD83**: Uses GRS80 ellipsoid for North America
- **UTM WGS84**: Uses WGS84 ellipsoid for global applications
- The advantage of the NAD83 datum is more accuracy for modeling and analyzing locational data in North America

## Web Mercator

### Origins and Adoption

Web Mercator rose to prominence when Google Maps adopted it in 2005. It is used by virtually all major online map providers, including Google Maps, CARTO, Mapbox, Bing Maps, OpenStreetMap, Mapquest, Esri, and many others.

### Technical Implementation

Web Mercator (EPSG:3857) is a variant of the Mercator map projection that uses spherical development of ellipsoidal coordinates. While geographical coordinates are required to be in the WGS84 ellipsoidal datum, the projection uses spherical formulas.

Key parameters:

```
EPSG: 3857
Projection: Mercator_1SP
Central Meridian: 0°
Scale Factor: 1
False Easting: 0 m
False Northing: 0 m
Units: meters
```

### Mathematical Discrepancies

This discrepancy between ellipsoidal input and spherical projection causes the projection to be slightly non-conformal. Mistaking Web Mercator for the standard Mercator during coordinate conversion can lead to deviations as much as 40 km on the ground.

### Coverage Limitations

Because the Mercator projects the poles at infinity, services such as Google Maps cut off coverage at 85.051129° north and south. This latitude represents where the full projected map becomes a square.

The limiting latitude is calculated as:
$$
\phi_{max} = 2\arctan(e^\pi) - \frac{\pi}{2} = 85.051129°
$$

### Performance Optimization

Web Mercator was introduced specifically to support worldwide map applications on the web. It attempts to preserve the shape of features in the areas where people are most interested, while making the behind-the-scenes coordinate conversion math run faster, which is important for thin clients like browsers and mobile apps.

### Critical Limitations

Web Mercator's significant weakness is that measurements of distance and area in its native coordinates are completely unusable. For accurate distance and area calculations, web-mercator GIS data must be temporarily reprojected to a more suitable coordinate system.

Practical impact example:

- Distance measurement between two points: 138 meters in Web Mercator vs. 99 meters in UTM Zone 15N—a difference of approximately 39%
- Area measurement: Lake Maurepas measures about 240 sq. km in UTM but over 310 sq. km in Web Mercator

## Conversions

```typescript
type Coordinate = {
  x: number;
  y: number;
};

type GeographicCoordinate = {
  latitude: number;
  longitude: number;
};

const EARTH_RADIUS = 6378137; // WGS84 semi-major axis
const MAX_LATITUDE = 85.05112878; // Max latitude for square Web Mercator

const clampLat = (lat: number) =>
    Math.max(Math.min(lat, MAX_LATITUDE), -MAX_LATITUDE);

// Convert WGS84 decimal degrees to Web Mercator
const wgs84ToWebMercator = (coord: GeographicCoordinate): Coordinate => {
  const clampedLat = clampLat(coord.latitude); 
  const x = coord.longitude * (Math.PI / 180) * EARTH_RADIUS;
  const y =
    Math.log(Math.tan(((90 + clampedLat) * Math.PI) / 360)) * EARTH_RADIUS;

  return { x, y };
};

// Convert Web Mercator back to WGS84
const webMercatorToWGS84 = (coord: Coordinate): GeographicCoordinate => {
  let longitude = (coord.x / EARTH_RADIUS) * (180 / Math.PI);
  const latitude =
    (2 * Math.atan(Math.exp(coord.y / EARTH_RADIUS)) - Math.PI / 2) *
    (180 / Math.PI);

  // Wrap longitude to [-180, 180]
  while (longitude > 180) {
    longitude -= 360;
  }

  while (longitude < -180) {
    longitude += 360;
  }

  return { longitude, latitude };
};

// UTM Zone calculation
const calculateUTMZone = (longitude: number): number => {
  return Math.floor((longitude + 180) / 6) + 1;
};

// Determine hemisphere for UTM
const getUTMHemisphere = (latitude: number): 'N' | 'S' => {
  return latitude >= 0 ? 'N' : 'S';
};
```

## Distance

### Haversine Distance

The Haversine formula calculates the shortest distance between two points on the surface of a sphere (great circle distance). It's named after the haversine function, which was historically used in navigation before electronic calculators existed.

#### The Haversine Function

The haversine of an angle θ is defined as:

$$\text{haversine}(\theta) = \sin^2\left(\frac{\theta}{2}\right) = \frac{1 - \cos(\theta)}{2}$$

This function has useful properties for spherical trigonometry because it avoids some numerical issues that occur with other formulations when dealing with small distances.

#### The Complete Formula

Given two points with coordinates:

- Point 1: ($\phi_1$, $\lambda_1$) - latitude and longitude in radians
- Point 2: ($\phi_2$, $\lambda_2$) - latitude and longitude in radians

$$
\begin{gathered}
\Delta \phi = \phi_2 - \phi_1 \\
\Delta \lambda = \lambda_2 - \lambda_1 \\
a = \sin^2\left(\frac{\Delta \phi}{2}\right) + \cos(\phi_1) \cdot \cos(\phi_2) \cdot \sin^2\left(\frac{\Delta \lambda}{2}\right) \\
c = 2 \cdot \operatorname{atan2}\left(\sqrt{a}, \sqrt{1-a}\right) \\
d = R \cdot c
\end{gathered}
$$

```typescript
type GeographicCoordinate = {
  longitude: number; // x
  latitude: number;  // y
}

const toRadians = (degrees: number): number => {
  return degrees * (Math.PI / 180);
}

// Haversine formula for great circle distance (WGS84)
const haversineDistance = (
  coord1: GeographicCoordinate, 
  coord2: GeographicCoordinate
): number => {
  const R = 6371000; // Earth's radius in meters
  const y1 = toRadians(coord1.latitude);
  const y2 = toRadians(coord2.latitude);
  const dy = toRadians(coord2.latitude - coord1.latitude);
  const dx = toRadians(coord2.longitude - coord1.longitude);

  const a = Math.sin(dy / 2) * Math.sin(dy / 2) +
            Math.cos(y1) *
            Math.cos(y2) *
            Math.sin(dx / 2) * 
            Math.sin(dx / 2);
  
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c; // Distance in meters
};
```


### Euclidean Distance

$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

```typescript
type Coordinate = {
  x: number;
  y: number;
}

// Euclidean distance for projected coordinates (UTM, State Plane)
const euclideanDistance = (coord1: Coordinate, coord2: Coordinate): number => {
  const dx = coord2.x - coord1.x;
  const dy = coord2.y - coord1.y;
  return Math.sqrt(dx * dx + dy * dy);
};
```

## Coordinate System Selection Guidelines

### Global vs. Regional Applications

**Use WGS84 Geographic (EPSG:4326) for:**

- Global datasets spanning multiple continents
- GPS data collection and storage
- International data exchange
- Web services requiring global coverage

**Use UTM for:**

- Regional analysis requiring accurate measurements
- Engineering and surveying projects
- Local government mapping
- Scientific studies requiring precise area calculations

**Use Web Mercator (EPSG:3857) for:**

- Web mapping applications
- Visualization-focused applications
- Integration with commercial mapping services
- Applications prioritizing display speed over measurement accuracy

The choice of coordinate system directly impacts data accuracy, processing performance, and analytical results. Modern GIS software can handle coordinate transformations automatically, but understanding the underlying mathematics and limitations ensures proper data interpretation and prevents costly errors in spatial analysis.