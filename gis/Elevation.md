---
tags:
  - gis
gardening: 🌱
date: 2025-08-13
reference:
  - https://eos-gnss.com/knowledge-base/articles/elevation-for-beginners
  - https://support.esri.com/en-us/knowledge-base/how-to-calculate-mean-sea-level-values-from-altitudes-u-000016396
  - https://support.virtual-surveyor.com/support/solutions/articles/1000261349-the-difference-between-ellipsoidal-geoid-and-orthometric-elevations-
  - https://vdatum.noaa.gov/docs/datums.html
  - https://earth-info.nga.mil/index.php?dir=wgs84&action=wgs84
  - https://rashms.com/gis/geoid-vertical-datum-elevation-navd88/
  - https://ahrs.readthedocs.io/en/stable/wgs84.html
  - https://www.mathworks.com/help/map/ref/wgs84ellipsoid.html
  - https://svenruppert.com/2023/12/18/what-is-wgs84-an-overview/
---
Calculating altitude in Geographic Information Systems (GIS) involves various reference frameworks, each tailored for specific applications. The main altitude measurement systems are Above Ground Level (AGL), Mean Sea Level (MSL), and Height Above Ellipsoid (HAE). Understanding these systems is essential for accurate geospatial analysis, especially in fields like aviation, surveying, and environmental modeling.

Altitude determination in GIS requires careful consideration of the reference surface or datum being used for measurement. Unlike horizontal positioning, which primarily focuses on latitude and longitude on a defined ellipsoid, vertical positioning entails complex gravitational and geometric factors that influence accuracy and interoperability among different systems.

## Primary Altitude Reference Systems

### 1. Above Ground Level (AGL)

AGL represents the height measured with respect to the underlying ground surface at a specific location.

**Mathematical Representation**:

$$AGL = H_{total} - H_{ground}$$

Where:
- $H_{total}$ is the total height above reference datum
- $H_{ground}$ is the ground elevation above same reference datum

**Applications**:

- Aviation obstacle clearance
- Telecommunications tower regulations
- Environmental monitoring (atmospheric measurements)
- Drone flight planning

**Key Characteristics**:

- Highly location-dependent
- Varies with terrain elevation
- Critical for safety applications
- Requires accurate Digital Elevation Models (DEMs)

### 2. Mean Sea Level (MSL)

MSL represents an arithmetic average height of the sea surface relative to the Earth's surface, typically calculated over a 19-year tidal cycle to account for lunar and solar gravitational effects.

**Limitations**:

- Only directly applicable near coastlines
- Varies geographically due to ocean currents, temperature, and salinity
- Not a uniform global surface

**Relationship to Orthometric Height**: MSL serves as the foundation for orthometric height systems, though modern geodetic practice uses the geoid as a more precise reference.

### 3. Height Above Ellipsoid (HAE)

**Definition**: HAE represents the perpendicular distance from a point to a mathematical reference ellipsoid, measured along the normal to the ellipsoid surface. This is the primary altitude measurement provided by Global Navigation Satellite Systems (GNSS) including GPS.

#### 3.1 Mathematical Foundation

**Ellipsoid as Earth Model**: An ellipsoid is a mathematically defined surface that approximates the geoid, accounting for the Earth's flattening caused by centrifugal force from rotation. The Earth's actual shape is more accurately represented by an ellipsoid than a perfect sphere, better approximating the Earth's accurate dimensions and accounting for flattening at the poles and bulging at the equator.

**WGS84 Ellipsoid Parameters**: The WGS84 reference ellipsoid is defined by four primary parameters:

- **Semi-major axis (a)**: 6,378,137.0 meters (equatorial radius)
- **Semi-minor axis (b)**: 6,356,752.314245179 meters (polar radius)
- **Flattening (f)**: 1/298.257223563
- **Inverse flattening (1/f)**: 298.257223563
- **First eccentricity squared (e²)**: 6.69437999014×10⁻³
- **Eccentricity (e)**: 0.0818191908426215

**Derived Relationships**:

$b = a \times (1 - f) = a \times (1 - \frac{1}{298.257223563})$

$e^2 = \frac{a^2 - b^2}{a^2} = 2f - f^2$

$\text{Eccentricity} = \sqrt{1 - \frac{b^2}{a^2}}$

#### 3.2 Geodetic Coordinate System

**Coordinate Triad**: Geodetic coordinates include geodetic latitude (north/south) $\phi$ , longitude (east/west) $\lambda$, and ellipsoidal height $h$, forming a curvilinear orthogonal coordinate system.

**Geodetic vs. Geocentric Latitude**: Geodetic latitude ($\phi$) is the angle between the ellipsoid normal and the equatorial plane, while geocentric latitude is the angle from Earth's center. The difference can be up to 11.5 arcminutes due to Earth's oblateness.

**Normal Section**: The ellipsoid normal at any point is the line perpendicular to the ellipsoid surface at that location. HAE is measured along this normal, not along a vertical line to Earth's center.

#### 3.3 Technical Implementation

```typescript
type EllipsoidParameters = {
  semiMajorAxis: number;       // a - meters
  semiMinorAxis: number;       // b - meters  
  flattening: number;          // f - dimensionless
  inverseFlattening: number;   // 1/f
  firstEccentricitySq: number; // e² - dimensionless
  eccentricity: number;        // e - dimensionless
};

const WGS84_ELLIPSOID: EllipsoidParameters = {
  semiMajorAxis: 6378137.0,
  semiMinorAxis: 6356752.314245179,
  flattening: 1 / 298.257223563,
  inverseFlattening: 298.257223563,
  firstEccentricitySq: 6.69437999014e-3,
  eccentricity: 0.0818191908426215
};

type GeodeticCoordinate = {
  latitude: number;          // φ - radians
  longitude: number;         // λ - radians  
  ellipsoidalHeight: number; // h - meters above ellipsoid
};

type CartesianCoordinate = {
  x: number; // meters - toward intersection of equator and prime meridian
  y: number; // meters - toward intersection of equator and 90°E
  z: number; // meters - toward North Pole
};

/**
 * Calculate prime vertical radius of curvature (N)
 * N = a / sqrt(1 - e² * sin²(φ))
 */
const calculatePrimeVerticalRadius = (
  latitude: number, 
  ellipsoid: EllipsoidParameters
): number => {
  const sinLat = Math.sin(latitude);
  const denominator = Math.sqrt(
    1 - ellipsoid.firstEccentricitySq * sinLat * sinLat
  );

  return ellipsoid.semiMajorAxis / denominator;
};

/**
 * Calculate meridional radius of curvature (M)  
 * M = a(1 - e²) / (1 - e² * sin²(φ))^(3/2)
 */
const calculateMeridionalRadius = (
  latitude: number,
  ellipsoid: EllipsoidParameters  
): number => {
  const sinLat = Math.sin(latitude);
  const temp = 1 - ellipsoid.firstEccentricitySq * sinLat * sinLat;
  const numerator = ellipsoid.semiMajorAxis * (1 - ellipsoid.firstEccentricitySq);
  return numerator / Math.pow(temp, 1.5);
};

/**
 * Convert geodetic coordinates to Earth-Centered Earth-Fixed (ECEF) Cartesian
 * This is fundamental to GPS positioning calculations
 */
const geodeticToECEF = (
  coord: GeodeticCoordinate,
  ellipsoid: EllipsoidParameters = WGS84_ELLIPSOID
): CartesianCoordinate => {
  const N = calculatePrimeVerticalRadius(coord.latitude, ellipsoid);
  const cosLat = Math.cos(coord.latitude);
  const sinLat = Math.sin(coord.latitude);
  const cosLon = Math.cos(coord.longitude);
  const sinLon = Math.sin(coord.longitude);
  
  return {
    x: (N + coord.ellipsoidalHeight) * cosLat * cosLon,
    y: (N + coord.ellipsoidalHeight) * cosLat * sinLon,
    z: (N * (1 - ellipsoid.firstEccentricitySq) + coord.ellipsoidalHeight) * sinLat
  };
};

/**
 * Convert ECEF to geodetic coordinates using iterative method
 * Critical for processing raw GNSS measurements
 */
const ecefToGeodetic = (
  cartesian: CartesianCoordinate,
  ellipsoid: EllipsoidParameters = WGS84_ELLIPSOID,
  tolerance: number = 1e-12
): GeodeticCoordinate => {
  const { x, y, z } = cartesian;
  const { semiMajorAxis: a, firstEccentricitySq: e2 } = ellipsoid;
  
  // Calculate longitude (straightforward)
  const longitude = Math.atan2(y, x);
  
  // Calculate latitude and height iteratively
  const p = Math.sqrt(x * x + y * y);
  let latitude = Math.atan2(z, p * (1 - e2));
  let height = 0;
  let prevLatitude: number;
  
  do {
    prevLatitude = latitude;
    const N = calculatePrimeVerticalRadius(latitude, ellipsoid);
    height = p / Math.cos(latitude) - N;
    latitude = Math.atan2(z, p * (1 - e2 * N / (N + height)));
  } while (Math.abs(latitude - prevLatitude) > tolerance);
  
  return {
    latitude,
    longitude, 
    ellipsoidalHeight: height
  };
};

/**
 * Calculate local coordinate system transformation matrix
 * Used for converting between global and local coordinate systems
 */
const calculateLocalTransformMatrix = (
  reference: GeodeticCoordinate
): number[][] => {
  const cosLat = Math.cos(reference.latitude);
  const sinLat = Math.sin(reference.latitude);
  const cosLon = Math.cos(reference.longitude);
  const sinLon = Math.sin(reference.longitude);
  
  // East-North-Up (ENU) transformation matrix
  return [
    [-sinLon, cosLon, 0],
    [-sinLat * cosLon, -sinLat * sinLon, cosLat],
    [cosLat * cosLon, cosLat * sinLon, sinLat]
  ];
};
```

#### 3.4 GNSS Implementation and Processing

GPS uses WGS84 as its reference coordinate system, with positioning accuracy better than 2 centimeters to Earth's center of mass. GNSS receivers solve for position using trilateration from multiple satellites.

#### 3.5 Accuracy and Error Sources

**Precision Characteristics**:
- **GPS Standard Positioning Service**: ±3-5 meters horizontal, ±5-10 meters vertical (95% confidence)
- **Differential GPS (DGPS)**: ±1-3 meters horizontal, ±2-5 meters vertical  
- **Real-Time Kinematic (RTK)**: ±2-5 centimeters horizontal, ±5-10 centimeters vertical
- **Precise Point Positioning (PPP)**: ±10-30 centimeters after convergence

**Error Sources**:
1. **Atmospheric delays**: Ionospheric (1-10m) and tropospheric (0.1-2m) signal delays
2. **Satellite clock errors**: Nanosecond timing errors translate to meter-level position errors
3. **Orbital ephemeris errors**: Satellite position uncertainties (±1-5m)
4. **Multipath**: Signal reflections from nearby surfaces (±0.1-5m)
5. **Receiver noise**: Thermal and quantization noise (±0.1-1m)

#### 3.6 Datum Transformations and Interoperability

WGS84 differs very slightly from GRS80 ellipsoid, with a maximum departure of 0.1 millimeter in ellipsoidal height at Earth's pole, but transformations to other datums require careful attention to datum shifts.

#### 3.7 Practical Considerations

**When to Use HAE**:
- Satellite positioning and tracking
- Global datasets requiring consistency
- Preliminary positioning before datum conversion
- Scientific applications requiring geometric reference

**Limitations**:
- Not intuitive for practical applications (doesn't represent "sea level")
- Requires geoid model for conversion to meaningful elevations
- Varies significantly from local vertical datums
- WGS84 accuracy depends on availability of reference stations, with cm-level accuracy limited outside of areas with dense control networks

### 4. Orthometric Height

The distance between a point on the Earth's surface and the geoid, measured along the curved path of gravity (the plumb line).

**Mathematical Relationship**:

$$h = H + N$$

Where:
- $h$ is the ellipsoidal height (HAE)
- $H$ is the orthometric height
- $N$ is the geoid height (undulation)

## Geoid Models and Vertical Datums

### Earth Gravitational Models (EGM)

**EGM2008**: The current standard global geoid model, complete to spherical harmonic degree and order 2159, providing geoid heights with accuracy better than 1 meter globally.

**EGM96**: Legacy model complete to degree and order 360, still used in some applications but superseded by EGM2008.

### National Vertical Datums

**NAVD88 (North American Vertical Datum 1988)**:

- Primary vertical datum for the United States
- Based on the Helmert orthometric height system
- Uses GEOID18 model for continental US (as of 2021)
- Realizes the geoid through a network of benchmarks

**Future: NAPGD2022**:

- Planned replacement for NAVD88
- Based on GEOID2022 and GRAV-D project data
- Will rely primarily on GNSS positioning

### Accuracy Considerations

**Error Sources**:

1. Geoid model uncertainty: ±5-50 cm depending on location and model
2. Ellipsoid-geoid conversion: ±1-10 cm for well-surveyed areas
3. Ground elevation model errors: ±0.5-10 m depending on DEM quality
4. GNSS measurement errors: ±0.5-5 m depending on equipment and conditions

**Best Practices**:

- Use the most recent geoid model available for your area of interest
- Validate conversions against known benchmarks
- Account for antenna height in GNSS measurements
- Consider local vertical datum transformations when required

## Standards and Specifications

**International Standards**:

- ISO 19111:2019 - Geographic Information - Spatial Referencing by Coordinates
- ISO 19161:2020 - Geographic Information - Geodetic References

**US Standards**:

- NGS Guidelines for GNSS Data Processing
- FGDC-STD-018-2012 - United States Thoroughfare, Landmark, and Postal Address Data Standard

**Data Sources**:

- NOAA VDatum: Vertical datum transformation software
- NGA EGM2008: Global geoid model
- USGS 3DEP: High-resolution elevation data