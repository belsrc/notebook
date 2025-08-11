---
tags:
  - gis
gardening: 🌱
date: 2025-08-11
reference:
  - https://desktop.arcgis.com/en/arcmap/latest/extensions/network-analyst/bearing-and-bearingtol-what-are.htm#:~:text=The%20Bearing%20and%20BearingTol%20fields%20are%20described%20below.,clockwise%20fashion%20from%20true%20north.
  - https://support.esri.com/en-us/gis-dictionary/bearing#:~:text=%5Bmeasurement%2C%20navigation%5D%20The%20horizontal,direction%20clockwise%20through%20360%20degrees.
  - https://atlas.co/gis-use-cases/bearings/#:~:text=This%20involves%20importing%20data%20that,property%20lines%20and%20physical%20features.
---
**Bearing** is a measurement used in surveying and navigation to describe the direction of a line from a specific point. It is an angular measurement, typically expressed in degrees, indicating the direction of one point relative to another. In the context of Geographic Information Systems (GIS), bearing is essential for defining spatial relationships between features, analyzing movement, and performing geoprocessing tasks.

## True vs. Magnetic vs. Grid Bearing

When working with GIS data, you will encounter several types of bearing, each with its own reference point:

- **True Bearing:** This is the angle measured clockwise from true north (geographic north). True north points towards the geographic North Pole and serves as a constant and unchanging reference point.
- **Magnetic Bearing:** This is the angle measured clockwise from magnetic north. Magnetic north indicates the direction that a compass needle points, influenced by the Earth's magnetic field. Because the magnetic field shifts over time and varies by location, magnetic bearing is less stable than true bearing.
- **Grid Bearing:** This is the angle measured clockwise from grid north. Grid north corresponds to the direction of the vertical grid lines on a map projection, such as the Universal Transverse Mercator (UTM). Grid north provides a consistent reference for a specific map grid, but it may differ slightly from true north.

## Quadrantal and Azimuthal Systems

Bearing can be expressed using two primary systems:

- **Azimuthal Bearing:** This is the most common system used in GIS and surveying. It measures the angle relative to a reference direction, usually north, and is expressed clockwise from 0° to 360°. For example, a bearing of 90° indicates a direction due east.
- **Quadrantal Bearing:** This system divides the compass into four quadrants: North-East (NE), South-East (SE), South-West (SW), and North-West (NW). The bearing is expressed as the angle measured from either north or south, toward the east or west. For example, N45°E represents the same direction as an azimuthal bearing of 45°, while S45°W corresponds to an azimuthal bearing of 225°.

## Mathematical Representation

### Forward Azimuth Calculation

Given two points with coordinates $(x_1, y_1)$ and $(x_2, y_2)$, the bearing from point 1 to point 2 is calculated as:

$$\theta = \arctan2(\Delta x, \Delta y)$$

Where:
- $\Delta x = x_2 - x_1$
- $\Delta y = y_2 - y_1$

The result is typically normalized to the range $[0, 2\pi)$ radians or $[0°, 360°)$.

### Geographic Coordinate Bearing

For geographic coordinates (latitude/longitude), the bearing calculation requires spherical trigonometry:

$$\theta = \arctan2(\sin(\Delta\lambda) \cdot \cos(\phi_2), \cos(\phi_1) \cdot \sin(\phi_2) - \sin(\phi_1) \cdot \cos(\phi_2) \cdot \cos(\Delta\lambda))$$

Where:
- $\phi_1, \phi_2$ = latitude of points 1 and 2 (in radians)
- $\Delta\lambda$ = difference in longitude (in radians)

## TS Implementation

```typescript
type Point2D = {
  x: number;
  y: number;
};

type GeographicPoint = {
  latitude: number;
  longitude: number;
};

type BearingResult = {
  degrees: number;
  radians: number;
  quadrant: string;
};

const degreesToRadians = (degrees: number): number =>
  degrees * Math.PI / 180;

const radiansToDegrees = (radians: number): number =>
  radians * 180 / Math.PI;

const normalizeAngle = (angle: number): number => {
  return ((angle % (2 * Math.PI)) + (2 * Math.PI)) % (2 * Math.PI);
};

const calculateCartesianBearing = (
  from: Point2D,
  to: Point2D
): BearingResult => {
  const deltaX = to.x - from.x;
  const deltaY = to.y - from.y;
  
  // Calculate bearing in radians
  // (mathematical convention: 0 = east, counterclockwise)
  const mathAngle = Math.atan2(deltaY, deltaX);
  
  // Convert to geographic bearing (0 = north, clockwise)
  const geoBearing = normalizeAngle((Math.PI / 2) - mathAngle);
  
  const degrees = radiansToDegrees(geoBearing);
  
  return {
    degrees,
    radians: geoBearing,
    quadrant: getQuadrant(degrees)
  };
};

const calculateGeographicBearing = (
  from: GeographicPoint, 
  to: GeographicPoint
): BearingResult => {
  const phi1 = degreesToRadians(from.latitude);
  const phi2 = degreesToRadians(to.latitude);
  const deltaLambda = degreesToRadians(to.longitude - from.longitude);
  
  const y = Math.sin(deltaLambda) * Math.cos(phi2);
  const x = Math.cos(phi1) * Math.sin(phi2) - 
            Math.sin(phi1) * Math.cos(phi2) * Math.cos(deltaLambda);
  
  const bearing = normalizeAngle(Math.atan2(y, x));
  const degrees = radiansToDegrees(bearing);
  
  return {
    degrees,
    radians: bearing,
    quadrant: getQuadrant(degrees)
  };
};

const getQuadrant = (degrees: number): string => {
  if (degrees >= 0 && degrees < 90) return "NE";
  if (degrees >= 90 && degrees < 180) return "SE";
  if (degrees >= 180 && degrees < 270) return "SW";
  return "NW";
};

// Forward and back bearing relationship
const calculateBackBearing = (forwardBearing: number): number => {
  return normalizeAngle(forwardBearing + Math.PI);
};

// Example usage
const point1: Point2D = { x: 100, y: 200 };
const point2: Point2D = { x: 300, y: 400 };

const cartesianBearing = calculateCartesianBearing(point1, point2);
console.log(`Cartesian bearing: ${cartesianBearing.degrees.toFixed(2)}°`);
// Cartesian bearing: 45.00°

const geoPoint1: GeographicPoint = { latitude: 40.7128, longitude: -74.0060 };
// NYC
const geoPoint2: GeographicPoint = { latitude: 34.0522, longitude: -118.2437 }; 
// LA

const geoBearing = calculateGeographicBearing(geoPoint1, geoPoint2);
console.log(`Geographic bearing: ${geoBearing.degrees.toFixed(2)}°`);
// Geographic bearing: 273.69°
```

## Bearing in GIS Applications

In Geographic Information Systems (GIS), bearing is a crucial concept employed in various applications:

- **Linear Direction:** Bearing defines the direction of line features, such as roads, pipelines, or property boundaries. This information is typically stored as an attribute of the line segment.
- **Navigation:** In route planning and navigation systems, bearing is used to provide turn-by-turn directions. The system calculates the bearing to the next waypoint to guide users effectively.
- **Movement Analysis:** Bearing plays a vital role in analyzing the movement of objects, such as tracking wildlife or monitoring vehicles. By calculating the bearing between consecutive points, one can determine the direction of travel.
- **Geoprocessing:** Many GIS tools utilize bearing as an input or output parameter. For example, a tool may generate a buffer around a line based on a specific bearing or calculate the bearing between two points to determine their relative direction.