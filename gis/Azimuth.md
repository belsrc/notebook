---
tags:
  - gis
gardening: 🌱
date: 2025-08-11
reference:
---
Azimuth is defined as the horizontal angle measured clockwise from a north reference line to a point of interest. It is a fundamental concept in Geographic Information Systems (GIS), surveying, and navigation, used to specify direction with a single value.

## Understanding Azimuth

In GIS, **azimuth** is an essential measurement for determining the orientation of features or the direction of movement. It functions similarly to a compass, but with a specific, standardized starting point.

- **Reference Line:** The measurement of azimuth always begins at **North**, which is typically True North (aligned with the Earth's axis of rotation) or Magnetic North (the direction a compass needle points).
- **Direction of Measurement:** The angle is measured **clockwise** from the North reference line.
- **Units:** Azimuth is expressed in **degrees**, ranging from 0° to 360°.

A 0∘ azimuth points due North, 90∘ is East, 180∘ is South, and 270∘ is West. An azimuth of 45∘ would be Northeast. This system is unambiguous because every possible direction has a unique value between 0∘ and 360∘.

Mathematically, given two points $P_1(x_1, y_1)$ and $P_2(x_2, y_2)$ in a Cartesian coordinate system, the azimuth $a$ from $P_1$ to $P_2$ can be calculated using:

$$\alpha = \arctan2(\Delta x, \Delta y)$$

Where:
- $\Delta x = x_2 - x_1$ (eastward displacement)
- $\Delta y = y_2 - y_1$ (northward displacement)

However, this requires careful handling of quadrants and coordinate system conventions.

## Azimuth Reference Systems

Different azimuth reference systems are used in GIS depending on the application and coordinate system:

### 1. True Azimuth (Geodetic Azimuth)
References true north (geographic north pole). Used in geodetic calculations and when working with geographic coordinate systems.

### 2. Grid Azimuth
References grid north in projected coordinate systems (like UTM). The difference between grid north and true north is called **grid convergence** (γ).

### 3. Magnetic Azimuth
References magnetic north. Rarely used in modern GIS but important in navigation applications.

The relationship between these systems can be expressed as:

$$T_A = G_A + G_C$$

$$G_A = M_A + M_D - G_C$$

Where:
- $T_A$ is **True Azimuth**
- $G_A$ is **Grid Azimuth**
- $G_C$ is **Grid Convergence**
- $M_A$ is **Magnetic Azimuth**
- $M_D$ is **Magnetic Declination**

### Grid Convergence Calculation

For UTM coordinate systems, grid convergence can be approximated using:

$$\gamma = (\lambda - \lambda_0) \sin \phi$$

Where:
- $\gamma$ = grid convergence
- $\lambda$ = longitude of point
- $\lambda_0$ = central meridian of UTM zone
- $\phi$ = latitude of point

## Typescript Implementation

```typescript
// Geographic coordinate type definition
type GeographicCoordinate = {
  latitude: number;   // degrees, positive north
  longitude: number;  // degrees, positive east
};

// Cartesian coordinate type definition
type CartesianCoordinate = {
  x: number;  // easting
  y: number;  // northing
};

const calculateCartesianAzimuth = (
  from: CartesianCoordinate, 
  to: CartesianCoordinate
): number => {
  const deltaX = to.x - from.x;
  const deltaY = to.y - from.y;
  
  // Calculate angle in radians, then convert to degrees
  const azimuthRad = Math.atan2(deltaX, deltaY);
  
  // Convert to degrees and ensure positive value (0-360°)
  let azimuthDeg = azimuthRad * (180 / Math.PI);
  
  // Normalize to 0-360° range
  if (azimuthDeg < 0) {
    azimuthDeg += 360;
  }
  
  return azimuthDeg;
};

const calculateGeographicAzimuth = (
  from: GeographicCoordinate, 
  to: GeographicCoordinate
): number => {
  // Convert degrees to radians
  const lat1 = from.latitude * (Math.PI / 180);
  const lat2 = to.latitude * (Math.PI / 180);
  const deltaLon = (to.longitude - from.longitude) * (Math.PI / 180);
  
  // Calculate azimuth using spherical trigonometry
  const y = Math.sin(deltaLon) * Math.cos(lat2);
  const x = Math.cos(lat1) * Math.sin(lat2) - 
            Math.sin(lat1) * Math.cos(lat2) * Math.cos(deltaLon);
  
  const azimuthRad = Math.atan2(y, x);
  
  // Convert to degrees and normalize to 0-360°
  let azimuthDeg = azimuthRad * (180 / Math.PI);

  if (azimuthDeg < 0) {
    azimuthDeg += 360;
  }
  
  return azimuthDeg;
};

const calculateBackAzimuth = (forwardAzimuth: number): number => {
  let backAzimuth = forwardAzimuth + 180;

  if (backAzimuth >= 360) {
    backAzimuth -= 360;
  }

  return backAzimuth;
};

const azimuthToBearing = (azimuth: number): string => {
  if (azimuth >= 0 && azimuth <= 90) {
    return `N ${azimuth.toFixed(1)}° E`;
  } else if (azimuth > 90 && azimuth <= 180) {
    return `S ${(180 - azimuth).toFixed(1)}° E`;
  } else if (azimuth > 180 && azimuth <= 270) {
    return `S ${(azimuth - 180).toFixed(1)}° W`;
  } else {
    return `N ${(360 - azimuth).toFixed(1)}° W`;
  }
};

const pointA: CartesianCoordinate = { x: 0, y: 0 };
const pointB: CartesianCoordinate = { x: 100, y: 100 };

const cartesianAz = calculateCartesianAzimuth(pointA, pointB);
console.log(`Cartesian azimuth: ${cartesianAz.toFixed(2)}°`);
// Cartesian azimuth: 45.00°
console.log(`Bearing: ${azimuthToBearing(cartesianAz)}`);
// Bearing: N 45.0° E

const london: GeographicCoordinate = { latitude: 51.5074, longitude: -0.1278 };
const paris: GeographicCoordinate = { latitude: 48.8566, longitude: 2.3522 };

const geoAzimuth = calculateGeographicAzimuth(london, paris);
console.log(`London to Paris azimuth: ${geoAzimuth.toFixed(2)}°`);
// London to Paris azimuth: 148.12°
console.log(`Back azimuth: ${calculateBackAzimuth(geoAzimuth).toFixed(2)}°`);
// Back azimuth: 328.12°
```

## Azimuth vs. Bearing

Although both azimuth and bearing describe direction, they are distinct concepts. **Bearing** refers to the angle measured from either North or South, limited to a range of 0° to 90°. The direction is indicated by including a cardinal direction (e.g., Northeast, Southwest).

- **Azimuth:** A single numerical value, such as 225°.
- **Bearing:** A value accompanied by two cardinal directions, such as S45°W (South 45 degrees West).

For instance, an azimuth of 135° and a bearing of S45°E both denote the same direction: Southeast. Azimuth is often preferred in GIS applications because its single numerical value is easier for computers to process and store in databases.

## Applications in GIS

Azimuth is widely used in Geographic Information Systems (GIS) for various analytical and visualization tasks.

- **Line Direction:** Calculating the azimuth of a line feature helps determine its orientation. This is particularly useful for analyzing street networks or geological fault lines.
- **Movement and Paths:** In tracking and navigation systems, the azimuth of a moving object indicates its current direction of travel.
- **Viewshed Analysis:** Azimuth is a key parameter in determining which areas are visible from a specific viewpoint. The "Field of View" can be defined by a range of azimuths.
- **Spatial Analysis:** When analyzing relationships between points, the azimuth from one point to another provides the direction of that relationship.

### Precision and Accuracy

When implementing azimuth calculations in production GIS software, consider the following:

1. **Floating-point Precision:** Use double precision (64-bit) for geographic calculations.
2. **Coordinate System Transformations:** Always transform to the appropriate system before performing calculations.
3. **Earth Ellipsoid:** Account for the ellipsoidal shape of the Earth when high precision is required.
4. **Datum Considerations:** Ensure consistent use of datums.

## Error Sources and Mitigation

Common sources of azimuth calculation errors include:

- **Coordinate System Mismatch:** Always verify that input coordinates are in the expected system.
- **Quadrant Errors:** Properly handle results from the atan2() function.
- **Datum Differences:** Transform to a common datum before calculations.
- **Projection Distortion:** Consider the effects of distortion in highly distorted projections.