---
tags:
  - deckgl
gardening: 🌱
date: 2025-12-28
---
## The Challenge: Large-Scale Geospatial Visualization in the Browser

Consider a concrete problem: You need to render 1 million GPS points on a map in a web browser. Each point represents a taxi pickup in New York City. The user should be able to:

- Pan and zoom smoothly (60 fps)
- Hover over points to see details
- Update the data dynamically (e.g., filter by time of day)

### The Problem: CPU Bottleneck

Traditional web visualization libraries (D3.js, Leaflet markers, etc.) use the **DOM** or **Canvas 2D API**. Both approaches hit fundamental scaling limits.

![](../../../images/deckgl/traditional-rendering.png)

For each point:
- Read the position from a JS array
- Transform coordinates
- Issue draw command
- Rasterize shape

The bottleneck is **CPU-bound iteration**. The CPU must sequentially process each data point, and each draw call incurs overhead from the JavaScript-to-native boundary crossing.

### The Solution: GPU Parallelism

GPUs solve this through **massive parallelism**. A modern GPU has thousands of cores that can execute the same operation on different data simultaneously. This is the **SIMD** (Single Instruction, Multiple Data) paradigm.

![](../../../images/deckgl/gpu-rendering.png)

The key insight, **move computation from the CPU to the GPU**.

### WebGL: Browser's GPU Interface

WebGL is a JavaScript API that exposes OpenGL ES functionality to the browser. It allows JavaScript to:

1. **Upload data** to GPU memory (as "buffers")
2. **Upload programs** to run on the GPU (as "shaders")
3. **Issue draw calls** that execute those programs on that data

![](../../../images/deckgl/webgl-mental-model.png)

### The Problem with Raw WebGL

WebGL is powerful but **extremely low-level**. Rendering a single colored triangle requires approximately 50-100 lines of boilerplate:

1. Create shaders (vertex + fragment) as strings
2. Compile shaders, check for errors
3. Create program, link shaders, check for errors
4. Create buffer, bind buffer, upload vertex data
5. Get attribute locations, enable attributes, set attribute pointers
6. Set uniforms (transformation matrices, colors, etc.)
7. Configure viewport, clear color, depth testing
8. Issue draw call

For **data visualization specifically**, additional challenges arise:

| Challenge              | Description                                                        |
| ---------------------- | ------------------------------------------------------------------ |
| **Coordinate Systems** | Geographic coords (lat/lng) → Web Mercator → screen pixels         |
| **Dynamic Data**       | Data changes; must efficiently update GPU buffers                  |
| **Interactivity**      | Picking (which point did the user click?) requires reverse mapping |
| **Layering**           | Multiple visualization layers with proper depth ordering           |
| **Performance**        | Minimizing draw calls, efficient buffer updates, avoiding stalls   |

### Deck.gl's Value Proposition

Deck.gl provides a **high-level, declarative API** for GPU-accelerated visualization while handling all the complexity above. It was created at Uber (circa 2016) to power their internal visualization tools for analyzing ride data at scale.

The core abstraction is the **Layer**.

```typescript
import { ScatterplotLayer } from '@deck.gl/layers';

const layer = new ScatterplotLayer({
  id: 'taxi-pickups',
  data: taxiData,                    // 1 million points
  getPosition: d => d.coordinates,   // Accessor function
  getRadius: d => d.passengerCount,
  getFillColor: [255, 140, 0],
  radiusMinPixels: 1,
});
```

From this declarative specification, Deck.gl:

1. Extracts positions by calling `getPosition` on each datum
2. Packs the data into typed arrays with proper memory layout
3. Uploads to GPU buffers
4. Generates or selects appropriate shaders
5. Manages coordinate transformations
6. Handles picking, updates, and lifecycle

The developer thinks in terms of **data and visual mappings**; Deck.gl handles the GPU machinery.

## Deck.gl's Architecture

Deck.gl is organized into distinct subsystems with clear responsibilities.

![](../../../images/deckgl/deckgl-architecture.png)

### Deck Instance

The `Deck` class is the entry point and orchestrator. It owns:

1. **The canvas element** (or accepts an existing one)
2. **The WebGL context** (via luma.gl's `Device`)
3. **The animation loop** (requestAnimationFrame-based render cycle)
4. **References to all subsystems**

```typescript
import { Deck } from '@deck.gl/core';

const deck = new Deck({
  // Canvas configuration
  canvas: 'deck-canvas',  // ID or HTMLCanvasElement
  width: '100%',
  height: '100%',
  
  // Initial state
  initialViewState: {
    longitude: -122.4,
    latitude: 37.8,
    zoom: 12,
  },
  
  // What to render
  layers: [
    new ScatterplotLayer({ ... }),
  ],
  
  // How to project
  views: [
    new MapView({ id: 'main' }),
  ],
  
  // Event callbacks
  onHover: (info) => { ... },
  onClick: (info) => { ... },
  onViewStateChange: ({ viewState }) => { ... },
  
  // Lifecycle callbacks
  onLoad: () => { ... },
  onError: (error) => { ... },
});
```

The Deck instance is **stateful but declaratively configured**. You update it by calling `deck.setProps({ layers: [...] })`. Deck.gl diffs the new props against the old and updates only what changed.

### Layer System

Layers are the core abstraction. Each layer:

1. **Consumes data** (array of objects, typed arrays, Arrow tables, etc.)
2. **Maps data to visual properties** (via accessors)
3. **Manages GPU resources** (buffers, textures)
4. **Renders geometry** (via luma.gl Models)

![](../../../images/deckgl/layer-taxonomy.png)

**Primitive layers** are the workhorses. They own shader programs and issue draw calls. **Composite layers** are conveniences that construct and manage primitive layers.

The **LayerManager** orchestrates all layers:

- Tracks layer instances across renders
- Matches old layers to new layers by `id`
- Triggers lifecycle methods
- Manages resource cleanup

### View System

The View system handles **projection from world coordinates to screen pixels**. It has three components.

[Image]

### Controller System

Controllers translate **user input** (mouse, touch, keyboard) into **viewState changes**.

[Image]

Controllers are associated with Views.

```typescript
new Deck({
  views: [
    new MapView({
      id: 'map',
      controller: {
        type: MapController,
        dragPan: true,
        dragRotate: true,
        scrollZoom: true,
        doubleClickZoom: true,
        touchRotate: true,
        keyboard: true,
        inertia: 300,
      },
    }),
  ],
});
```

### Rendering Pipeline

When a frame renders, this sequence occurs:

1. Animation Frame
   └─▶ requestAnimationFrame callback fires

2. Update Phase
   ├─▶ Process pending setProps() calls
   ├─▶ Diff layers (match by id, detect adds/removes/updates)
   ├─▶ Run layer lifecycle:
   │ • initializeState() for new layers
   │ • updateState() for changed layers
   │ • finalizeState() for removed layers
   └─▶ Update GPU buffers (AttributeManager)

3. Viewport Computation
   ├─▶ For each View, compute Viewport from viewState
   └─▶ Generate view-projection matrices

4. Draw Phase
   ├─▶ Clear framebuffer
   ├─▶ For each Viewport:
   │ ├─▶ Set WebGL viewport/scissor
   │ ├─▶ For each Layer (sorted by render order):
   │ │ ├─▶ Bind layer's shader program
   │ │ ├─▶ Set uniforms (matrices, time, etc.)
   │ │ ├─▶ Bind attribute buffers
   │ │ └─▶ Issue draw call(s)
   │ └─▶ (end layer loop)
   └─▶ (end viewport loop)

5. Post-Process (if effects enabled)
   └─▶ Apply screen-space effects (lighting, bloom, etc.)

6. Picking (if interaction occurred)
   ├─▶ Re-render to offscreen buffer with color-coded IDs
   ├─▶ Read pixel at cursor position
   └─▶ Decode picked layer + object index