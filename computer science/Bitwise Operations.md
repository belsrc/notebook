---
tags:
  - bitwise
gardening: 🌳
reference:
  - https://cp-algorithms.com/algebra/bit-manipulation.html
  - https://sudorudo.medium.com/bitwise-for-dummies-c1b996c21cbc
  - https://www.geeksforgeeks.org/introduction-to-bitwise-algorithms-data-structures-and-algorithms-tutorial/
  - https://www.alexhyett.com/bitwise-operators/
---
## Bitwise AND (`&`)

For each bit, if both bits are `1`, the resulting bit is `1`. If either bit or both are `0`, the resulting bit is `0`. In other words, it performs a logical **AND** operation on each individual bit.

Given the following two values:

![](../images/bitwise/setup-dark.png)

Performing the bitwise **AND** would visually look like:

![](../images/bitwise/and-dark.png)

```
22 & 93 === 20
```

The **AND** operator (`&`) can also be used to determine whether a number is even or odd. Since the only bit that indicates oddness is the rightmost bit (which is `1`), performing an **AND** operation with `1` will result in `1` for odd numbers and `0` for even numbers.

```ts
const isEven = (n: number) => (n & 1) === 0;
```

Another application of the **AND** operator is in checking bit masks. For example, if you have a user configuration with various states (yes or no), you can use this operator to evaluate the states effectively.

![](../images/bitwise/mask-dark.png)

```js
const userOptions = 0; // 0b0000
```

Then you can check options based on a bit mask.

```js
const SHOULD_SHOW_B = 1 << 2; // 0b0100

const hasFlag = (flags, mask) => !!(flags & mask);

const userConfig = 0b0101;

console.log(hasFlag(userConfig, SHOULD_SHOW_B));
// --> true
```

## Bitwise OR (`|`)

For each bit, if either one or both bits are `1`, the resulting bit will be `1`. If both bits are `0`, the resulting bit will be `0`. In other words, it performs a logical **OR** operation on each individual bit.

Given the following two values:

![](../images/bitwise/setup-dark.png)

Performing the bitwise **OR** would visually look like:

![](../images/bitwise/or-dark.png)

```
22 | 93 === 95
```

Using the example of user options mentioned earlier, the **OR** operator can be utilized to turn options ON.

```js
const SHOULD_SHOW_B = 1 << 2; // 0b0100

const hasFlag = (flags, mask) => !!(flags & mask);
const turnOnFlag = (flags, mask) => flags | mask;

let userConfig = 0b0001;

console.log(hasFlag(userConfig, SHOULD_SHOW_B));
// --> false

userConfig = turnOnFlag(userConfig, SHOULD_SHOW_B);

console.log(hasFlag(userConfig, SHOULD_SHOW_B));
// --> true
```

## Using AND (`&`) and OR (`|`) together

By combining both **AND** and **OR** operators in the configuration example mentioned earlier, we can toggle the user settings.

```js
const SHOULD_SHOW_A = 1 << 3; // 0b1000

const hasFlag = (flags, mask) => !!(flags & mask);
const turnOnFlag = (flags, mask) => flags | mask;
const turnOffFlag = (flags, mask) => flags & ~mask;
// bitwise NOT (~) on mask to inverse

const toggleOption = (flags, mask) =>
  hasFlag(flags, mask) ? turnOffFlag(flags, mask) : turnOnFlag(flags, mask);

let userConfig = 0b0101;

console.log(hasFlag(userConfig, SHOULD_SHOW_A));
// --> false

userConfig = toggleOption(userConfig, SHOULD_SHOW_A);

console.log(hasFlag(userConfig, SHOULD_SHOW_A));
// --> true

userConfig = toggleOption(userConfig, SHOULD_SHOW_A);

console.log(hasFlag(userConfig, SHOULD_SHOW_A));
// --> false
```

Or if we want to perform a comprehensive check to determine if **any** of the options are set. This way, we can skip processing if none are necessary. We can combine both **AND** and **OR** operators in this check.

```js
const SHOULD_SHOW_A = 1 << 3; // 0b1000
const SHOULD_SHOW_B = 1 << 2; // 0b0100
const SHOULD_SHOW_C = 1 << 1; // 0b0010
const SHOULD_SHOW_D = 1 << 0; // 0b0001

const hasAnyFlag = (flags) => !!(flags & (SHOULD_SHOW_A | SHOULD_SHOW_B | SHOULD_SHOW_C | SHOULD_SHOW_D));
// SHOULD_SHOW_A | SHOULD_SHOW_B | SHOULD_SHOW_C | SHOULD_SHOW_D === 0b1111

const userAConfig = 0b0101;
const userBConfig = 0b0000;

console.log(hasAnyFlag(userAConfig));
// --> true

console.log(hasAnyFlag(userBConfig));
// --> false
```

## Bitwise XOR (`^`) : Exclusive OR

The **exclusive OR** (XOR) differs from the standard **OR** in that it only produces a result of `1` when exactly one of the two bits is `1`. In all other cases, the result is `0`.

Given the following two values:

![](../images/bitwise/setup-dark.png)

Performing the bitwise XOR would visually look like:

![](../images/bitwise/xor-dark.png)

```
22 ^ 93 === 75
```

While the toggling example mentioned above does work, it's more complex than necessary for simply changing a bit from `1` to `0` or vice versa. Fortunately, the **XOR** operation can handle this for us more effectively.

```js
const SHOULD_SHOW_A = 1 << 3; // 0b1000

const hasFlag = (flags, mask) => !!(flags & mask);

const toggleOption = (flags, mask) => flags ^ mask;

let userConfig = 0b0101;

console.log(hasFlag(userConfig, SHOULD_SHOW_A));
// --> false

userConfig = toggleOption(userConfig, SHOULD_SHOW_A);

console.log(hasFlag(userConfig, SHOULD_SHOW_A));
// --> true

userConfig = toggleOption(userConfig, SHOULD_SHOW_A);

console.log(hasFlag(userConfig, SHOULD_SHOW_A));
// --> false
```

## Bitwise Not (`~`)

The bitwise NOT operator is a "two's compliment" operation. See below. The quick and dirty way to think about is that if the operand is positive: 

> Add one and then flip the sign

`~0 -> -(0 + 1) -> -1`

```js
~0
// >> -1
```

`~342 -> -(342 + 1) -> -343`

```js
~342
// >> -343
```

And if the operand is negative:

> Flip the sign and then subtract one

`~(-1) -> -(-1) - 1 -> 0`

```js
~(-1)
// >> 0
```

```js
~(-743)
// >> 742
```

A fairly common use case is using it to determine if an array index was found.

```js
const idx = [].findIndex(x => <condition>)

if(~idx) {
  // ...
}
```

Using the "quick and dirty" approach mentioned above, only `-1` results in `0` (¹), making it falsey. So the condition will execute only when an index is found, unlike the more verbose condition `if(idx > -1)`.

**⁽¹⁾** Actually, that's not entirely accurate. Due to how the number system operates, `~Number.MAX_SAFE_INTEGER` will evaluate to `0`. While I believe the likelihood of an array containing `9007199254740991` elements is low, it's still important to point this out.

## Twos Complement

Two's complement is a method for representing both positive and negative integers in binary form. For an n-bit number:

- **Positive numbers** are represented in standard binary format, including leading zeros if necessary.
- **Negative numbers** are represented by the following steps:
    1. **Inverting** all the bits of the corresponding positive number.
    2. **Adding one** to the inverted number.

#### Example

Lets convert +13 to -13.

Take the original binary format of the number.

```
+13 = 0b00001101
```

Invert all of the bits.

```
00001101
↓↓↓↓↓↓↓↓
11110010
```

And add one.

```
11110010 + 1 = 11110011
```

#### findeIndex Example

To understand why `~([].findIndex())` behaves the way it does, we need to look at the two's complement of `-1`.

First, we start with the binary representation of `1`, which is `0b00000001`. Next, we invert all the bits, resulting in `0b11111110`. Finally, we add `1`, giving us `0b11111111`. This is the two's complement representation of `-1`.

Now, when we apply the bitwise **NOT** operator (`~`), we flip all the bits again. This gives us `0b00000000`, which is simply `0`. Which will evaluate to `false`.

## Bitwise Truth Table

|  X  |  Y  | X & Y | X &#124; Y | X ^ Y | ~(X) |
| :-: | :-: | :---: | :--------: | :---: | :--: |
|  0  |  0  |   0   |     0      |   0   |  1   |
|  0  |  1  |   0   |     1      |   1   |  1   |
|  1  |  0  |   0   |     1      |   1   |  0   |
|  1  |  1  |   1   |     1      |   0   |  0   |

## Logical Shifting (`<<`, `>>`)

When shifting, the term refers to moving each bit either to the left or the right by _n_ positions. `0`s are added to the left or right end, depending on the direction of the shift. As a result, the left-most or right-most bits are lost.

![](../images/bitwise/shift-double-dark.png)

This operation essentially multiplies the number by 2 with each left shift. But what happens when we lose the most significant digit?

![](../images/bitwise/shift-drop-dark.png)

Shifting to the right works the same but in the opposite direction.

![](../images/bitwise/shift-half-dark.png)

Since it only operates on whole numbers, the result of a shift right on an odd number is "rounded" down.

```js
13 >> 1
// 6
```

## Using Logical Shift to (Un)Pack RGBA Codes

An RGBA code can be represented as four 8-bit integers that are "packed" together.

![](../images/bitwise/rgb-dark.png)

```
83, 155, 245, 255
```
```js
const rgbaCode = 0b01010011100110111111010111111111;
```

To "unpack" these values, we can use a combination of right shifting and an **AND** bit mask. The mask will extract the first 8 bits.

![](../images/bitwise/alpha-mask-dark.png)

```js
const VALUE_MASK = 0b00000000000000000000000011111111;
```

To obtain the Alpha value, we simply **AND** the original value with the mask.

```js
const VALUE_MASK = 0b00000000000000000000000011111111;
const rgbaCode = 0b01010011100110111111010111111111;

const alpha = rgbaCode & VALUE_MASK;
// 0b00000000000000000000000011111111
```

For the remaining values (Red, Green, and Blue), we need to right shift the original code by 8, 16, and 24 bits, respectively, and then apply the **AND** operation with the mask.

![](../images/bitwise/rgb-shift-dark.png)

```js
const VALUE_MASK = 0b00000000000000000000000011111111;
const rgbaCode = 0b01010011100110111111010111111111; // 1402729983

const alpha = rgbaCode & VALUE_MASK;
// 0b00000000000000000000000011111111 -> 255

const blue = (rgbaCode >> 8) & VALUE_MASK;
// 0b00000000000000000000000011110101 -> 245

const green = (rgbaCode >> 16) & VALUE_MASK;
// 0b00000000000000000000000010011011 -> 155

const red = rgbaCode >> 24;
// Since were shifting all the way over, we dont need to mask
// 0b00000000000000000000000001010011 -> 83

console.log([red, green, blue, alpha].join(', '));
// 83, 155, 245, 255
```

And to pack them back up we simply need to shift each one the appropriate number to the left.

```js
const red = 83;
const green = 155;
const blue = 245;
const alpha = 255;

const packedColor = (red << 24) + (green << 16) + (blue << 8) + alpha;

console.log(packedColor.toString(2));
// 0b01010011100110111111010111111111
// Technically its "1010011100110111111010111111111", JS trims the zeros from the left
```
