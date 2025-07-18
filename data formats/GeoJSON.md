---
tags:
  - gis
  - data
gardening: 🌱
reference:
  - https://datatracker.ietf.org/doc/html/rfc7946
---
## Intro

A GeoJSON object may represent a region of space (a `Geometry`), a spatially bounded entity (a `Feature`), or a list of Features (a `FeatureCollection`).  GeoJSON supports the following geometry types: `Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`, and `GeometryCollection`.

## Object

Every Geometry object is a GeoJSON object no matter where it occurs in the GeoJSON text. The object has a property `type` that much be one of the seven geometry types. Objects, other than the `GeometryCollection` has a `coordinates` property, which is an array. Structure of this array is determined by the `type` of the object.

## Position

Is the most basic construct and is an array of numbers. With a minimum of two elements and a maximum of three. The first two elements are longitude and latitude, or easting and northing, precisely in that order and using decimal numbers.  Altitude or elevation MAY be included as an optional third element.  The `coordinates` property can be composed up:

- One `Position` in the case of `Point`.
	- `[x, y]`
- An array of `Positions` in the case of `LineString` and `MultiPoint`.
	- `[[x, y], [x, y]]`
- An array of `LineString` or linear ring in the case of a `Polygon` or `MultiLineString`.
	- `[[[x, y], [x, y]], [[x, y], [x, y]]`
- An array of `Polygon` in the case of `MultiPolygon`.
	- `[[[[x, y], [x, y]]], [[[x, y], [x, y]]]]`

```ts
type Position = [number, number] | [number, number, number];
type Point = Position;
type LineString = Position[];
type MultiPoint = Position[];
type Polygon = LineString[];
type MultiLineString = LineString[];
type MultiPolygon = Polygon[];
```
_(These are obviously not full as there is no object and no coordinate prop)_

