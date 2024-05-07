---
tags:
  - css
  - notes
---
Container Queries allow you to write styles that apply to the children of a container element when that container matches certain conditions.


## Defining a Container

To define a container, use the `container-type` property on the element. It can accept three options: `size`, `inline-size`, and `normal`.

- `size`: Establishes a query container for container size queries in both the inline and block dimensions.

- `inline-size`: Establishes a query container for dimensional queries on the inline axis of the container.

- `normal`: The element is not a query container for any container size queries, but remains a query container for container style queries.

```css
@container style(--variant: 1) { 
  button { } /* You can't style .container, but can select inside it */
  .other-things { }
} 

@container style(--variant: 2) {
  button { }
  .whatever { }
}

.container {
  --variant: 1;
  
  &.variant2 {
    --variant: 2;
  }
}
```


## Named Context

A containment context can be named using the `container-name` property. The `container-name` property is optional and is case-sensitive.

```css
.post {
  container-type: inline-size;
  container-name: sidebar;
}
```

or with the short-hand:

```css
.post {
  container: sidebar / inline-size;
}
```

You can then target that container by name using the `@container` at-rule:

```css
@container sidebar (min-width: 400px) {
  /* <stylesheet> */
}
```


## Container Logical Operators

- `and` combines two or more conditions.

- `or` combines two or more conditions.

- `not` negates the condition. Only one 'not' condition is allowed per container query and cannot be used with the `and` or `or` keywords.

```css
@container (width > 400px) and (height > 400px) {
  /* <stylesheet> */
}

@container (width > 400px) or (height > 400px) {
  /* <stylesheet> */
}

@container not (width < 400px) {
  /* <stylesheet> */
}
```


### References

- https://frontendmasters.com/blog/what-you-need-to-know-about-modern-css-spring-2024-edition/#toc-45
- https://developer.mozilla.org/en-US/docs/Web/CSS/@container
- https://developer.mozilla.org/en-US/docs/Web/CSS/container
- https://developer.mozilla.org/en-US/docs/Web/CSS/container-type