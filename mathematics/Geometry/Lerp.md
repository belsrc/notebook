---
tags:
  - math
  - geometry
gardening: 🌱
date: 2025-04-21
reference:
  - https://www.youtube.com/watch?v=LSNQuFEDOyQ
---
```
// Move position `a` `p` percent closer to position `b`
const lerp = (a, b, p) => (b - a) * p;

// Different vid
const lerp = (a, b, t) => a * (1 - t) + b * t;
```

```
function moveTowards(a: Position, b: Position, spd: number, dt: number) {
  const vec = b - a;
  const stepDis = spd * dt;

  return stepDist < Math.abs(vec) // prevent overshooting
    ? a + Math.sin(v) * stepDist
    : b;
}
// Slightly different when talking about vectors (positions).
// Talked about @ ~10min into the video
```

```
// Frame independent lerp
function fiLerp(a, b, r, dt) {
  return (a - b) * r**dt + b;
}
```
