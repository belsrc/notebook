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
  // r + (g << 8) + (b << 16) - 1;

  // multiplication:
  // 70M ops/s ± 0.93%  
  // _1.92 % slower_

  // shifting:
  // 72M ops/s ± 0.95%  
  // **Fastest**
}

// 0...254 -> 0...254
// 255 -> 0
// >> 8 is basically subtracting 255
// and then doing the same as above
// >> 8 >> 8 is basically subtracting 255^2
// and then doing the same as the first

// encode(0...254) -> [n, 0, 0]
// encode(255...65535) -> [r, n, 0]
// encode(65535...) -> [r^2, r, n]

// 1 = 0b00000001 &= 1
// 2 = 0b00000010 &= 2
// 255 = 0b11111111 &= 255
// ...
// 256 = 0b0000000100000000 >>= 00000001 &= 1
// 65535 = 0b1111111111111111 >>= 0b11111111 &= 255
// ...
// 65536 = 0b000000010000000000000000 >>= 0b0b0000000100000000 >>= 0b0b00000001 &= 1
```

So basically you count up the first index, when it hits `255 (+1)`, reset the first index to `0` and add one to the second index. So on and so forth.

And the reverse is just multiplication by the "break-points". Though you can also just use bit shifting in the other direction to accomplish the same thing.
