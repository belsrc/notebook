---
tags:
  - deckgl
  - snippets
reference:
  - https://github.com/visgl/deck.gl/blob/c7f1a66e47062eee85ce30cfe1216d6fb8287aae/modules/core/src/lib/layer.ts#L389
---
```js
// Slightly changed for ease of exploring. The actual method accepts the index
// and a reference array.
// encodePickingColor(i, target: number[] = [])

function encodePickingColor(index) {
  const target = new Array(3);

  target[0] = (index + 1) & 255;
  target[1] = ((index + 1) >> 8) & 255;
  target[2] = (((index + 1) >> 8) >> 8) & 255;

  return target;
}

encodePickingColor(0)
//>> [1, 0, 0]

encodePickingColor(1)
//>> [2, 0, 0]

encodePickingColor(255)
//>> [0, 1, 0]

encodePickingColor(511)
//>> [0, 2, 0]

encodePickingColor(65535)
//>> [0, 0, 1]

function decodePickingColor(color) {
  const [r, g, b] = color;

  return r + g * 256 + b * 65536 - 1; 
}
```

So basically you count up the first index, when it hits `255 (+1)`, reset the first index to `0` and add one to the second index. So on and so forth.

And the reverse is just multiplication by the "break-points".

> [!WARNING]
> Break it down by bits. Rewrite decode via shifting?
