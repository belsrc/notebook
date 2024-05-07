---
tags:
  - css
  - notes
---
Cascade layers enable more explicit control of your CSS files to prevent style-specificity conflicts. This is particularly useful for large codebases, design systems, and when managing third party styles in applications.

Layering your CSS in a clear way prevents unexpected style overrides and promotes better CSS architecture.


## Defining Layers
 
 You can define a layer using the `@layer` rule and a name to create a block for the layer styles.

```css
@layer base {
  body { ... }
}
```


Optionally, you can also define and order layers using the a `@layer` statement.

```css
@layer reset, default, themes, patterns, layouts, components, utilities;
```

If you don’t explicitly order them, they order in source order.


You can put third-party styles on a layer, this helps with possibly having to fight with third-party library in terms of selector strength.

```css
@import url("https://cdn.com/bootstrap.css") layer;
```


## Nesting

You can also nest `@layer` rules.

```css
@layer framework {
  @layer reset, base, components;
}
```

you reference the parent and nested rules together using dot notation.

```css
@layer framework.reset { ... }
```

You can also define and order them using a `@layer` statement and dot notation, as well.

```css
@layer framework.reset, framework.base, framework.components;
```

If you re-use a name within a nested layer, it will not reference the outer layer but rather start a new layer with that name.

```css
@layer base;

@layer framework {
  @layer base { ... }
}
```


## Specificity

Styles in higher layers automatically beat styles in lower layers, **regardless of selector strength.**
That is to say, layers defined later will have their rules take priority over layers defined earlier.

A more simple selector in a later layer can override what historically would have been a stronger selector in an earlier layer.

This enables the creation of simpler CSS selectors because you do not have to ensure that a selector will have high enough specificity to override competing rules; all you need to ensure is that it appears in a later layer.

Styles _not_ within layers, “unlayered styles”, are the most powerful. Any styles not in a layer are gathered together and placed into a single anonymous layer that comes after all the declared layers.

```html
<body id="home">
```

```css
@layer base {
  body#home {
    margin: 0;
    background: #eee;
  }
}

body { background: white; }
```

Unlayered are stronger so background will be `white`

If you write styles inside a layer and then add nested layers, the rules outside of nested layers act like unlayered styles in terms of the cascade specificity.

```css
@layer typography {
  p {
    color: green;
  }

  @layer content;
}

@layer typography.content {
  p {
    color: blue;
  }
}
```

In this example, the paragraph will remain `green`.

Marking a property definition as `!important` raises its priority. In cascade layers, adding in `!important` reverses the sorting order such that competing `!important` styles in a layer defined earlier win over `!important` styles in later layers. Additionally, the use of `!important` within a layer will also win over an unlayered style.


### References

- https://frontendmasters.com/blog/what-you-need-to-know-about-modern-css-spring-2024-edition/#toc-45
- https://www.smashingmagazine.com/2022/01/introduction-css-cascade-layers/
- https://developer.mozilla.org/en-US/docs/Web/CSS/@layer