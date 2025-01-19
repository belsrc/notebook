---
tags:
  - deckgl
reference:
  - https://deck.gl/docs/developer-guide/custom-layers/picking
  - https://github.com/visgl/deck.gl/blob/57119698d781cc70bd7611dad023ab61d6497c0a/modules/core/src/lib/picking/pick-info.ts#L25
  - https://github.com/visgl/deck.gl/blob/57119698d781cc70bd7611dad023ab61d6497c0a/modules/core/src/lib/layer.ts#L404
gardening: 🌿
---
## Color Picking

Deck.gl implements picking on the GPU using a technique we refer to as "color picking". When deck.gl needs to determine what is under the mouse, all pickable layers are rendered into an off-screen buffer, but in a special mode activated by a GLSL uniform. In this mode, the shaders of the core layers render picking colors instead of their normal visual colors.

Each object in each layer gets its own picking color assigned. The picking color is determined using `layer.encodePickingColor()` that converts the index of an object of a given layer into a 3-byte color array. The color buffer allows us to distinguish between 16M unique colors per layer, and between 256 different layers.

After the picking buffer is rendered, deck.gl looks at the color of the pixel under the pointer, and decodes it back to the index number using `layer.decodePickingColor()`.

### encodePickingColor

Converts a "sub-feature index" number to a color.

```ts
encodePickingColor(index: number): [r: number, g: number, b: number];
```

It encodes the given index using a combination of [bitwise AND (`&`)](../../../computer%20science/Bitwise%20Explanation.md#bitwise-and-) and [bitwise right shifts](../../../computer%20science/Bitwise%20Explanation.md##logical-shifting--).

```js
function encode(index) {
  const target = new Array(3);

  target[0] = (index + 1) & 255;
  target[1] = ((index + 1) >> 8) & 255;
  target[2] = (((index + 1) >> 8) >> 8) & 255;

  return target;
}

encode(0)
//>> [1, 0, 0]

encode(1)
//>> [2, 0, 0]

encode(255)
//>> [0, 1, 0]

encode(510)
//>> [0, 1, 0]

encode(65535)
//>> [0, 0, 1]
```

### decodePickingColor

Converts a color to a "sub-feature index" number.

```ts
decodePickingColor(Uint8Array(color: [r: number, g: number, b: number])): number;
```

Decoding is just the reverse operation from above.

```js
function decode(color) {
  const [r, g, b] = color;

  return r + g * 256 + b * 65536 - 1; 
}
```

## Pointer Events

Once an object is picked, the `layer.getPickingInfo()` method is called first on the layer that directly rendered the picked object, to modify or add additional fields to the info.

The info object is then passed to the `getPickingInfo()` of its parent layer, and then its grandparent, and so on. This is so that composite layers can further augment the `info` object after it is processed by the picked sublayer.

```ts
type PickingInfo<T> = {
  // I believe this is the internal picking color
  color: Uint8Array | null;
  coordinate?: [number, number];
  devicePixel?: [number, number];
  index: number;
  index: number;
  layer: Layer | null;
  object: T;
  picked: boolean;
  pixel?: [number, number];
  pixelRatio: number;
  sourceLayer?: Layer | null;
  viewport?: Viewport;
  // Same as `pixel`
  x: number;
  y: number,
};

type getPickingInfoParam<T> = {
  info: PickingInfo;
  mode: 'hover' | 'query'; // query being picked; technical "string"
  sourceLayer: Layer | null;
};

function getPickingInfo(param: getPickingInfoParam): PickingInfo;

```

When the processing chain is over, the event is invoked on the top-level layer. This means that only the top-level layer's `on<Event>` callbacks are invoked, and the final picking info's `layer` field will point to the top-level layer. The idea is that the user should only interface with the layer that they created, and not having to know the underlying implementation.

When an event fires, the callback functions are executed in the following order:

- `layer.on<Event>` (default implementation invokes `this.props.on<Event>`)
- `layer.props.on<Event>` (only if the layer method is not defined)
- `deck.props.on<Event>`

If any of the callback functions return `true`, the event is marked handled and the rest of the callbacks will be skipped.

Whenever the pointer moves over the canvas, deck.gl performs a new picking pass, yielding a picking info object describing the result. This object is used for multiple purposes:

- The `onHover` callbacks are called with it
- To update the picked layer if autoHighlight is enabled
- Saved for later use

When other gestures (click, drag, etc.) are detected, deck.gl does not repeat picking. Instead, their callbacks are called with the last picked info from `hover`.