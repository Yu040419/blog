---
title: CSS Animation
date: 2024-02-13 20:48:18
tags:
  - CSS
---

## User Story

The user sees the roulette game, and only after clicking the start button, the API call is made to retrieve prize information. Once the prize data is obtained and the roulette is spun to the designated position, the winning information will be displayed.

## Keyframes

CSS animations are created using keyframes. Keyframes define the styles at various points in the animation timeline, allowing for smooth transitions between them.

```css=
@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
```

1. `from` / `to` = `0%` / `100%`
2. Automatic calculation if no start or end is specified
3. Use percentage with numbers

## Animation Properties

To apply keyframes to an element, you'll use the animation property. This property specifies the animation name, duration, timing function, iteration count and fill mode.

### Animation Name

The name of the keyframe animation you want to apply.

- case sensitive
- with or without quote
- reserved word `initial`, `None` is illegal for naming, but could be `'None'`

```css
.element {
  animation-name: rotate;
}

.element {
  animation-name: "initial";
}
```

### Animation Duration

> The total time taken for the animation to complete one cycle (specified in seconds or milliseconds).

- default set to 0

```css
.element {
  animation-duration: 5s;
}
```

### [Animation Timing Function](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timing-function)

> Specifies the speed curve of the animation. Common values include `ease`, `linear`, `ease-in`, `ease-out`, and `ease-in-out`.

- default set to `ease`
- [`cubic-bezier()`](https://cubic-bezier.com/#.17,.67,.83,.67)

```css
.element {
  animation-timing-function: ease-out;
}
```

### Animation Delay

> The time to wait before starting the animation (specified in seconds or milliseconds).

- default set to 0

```css
.element {
  animation-delay: 500ms;
}
```

### Animation Iteration Count

Specifies the number of times the animation should run. Use a finite number or `infinite` for continuous animation.

- default set to 1

```css
.element {
  animation-iteration-count: infinite; /* Runs the animation infinite times */
}
```

### [Animation Fill Mode](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode)

Describes how an element's style is applied before and after the animation. Common values include`none`, `forwards`, `backwards`, `both`.

- default set to `none`

```css
.element {
  animation-fill-mode: forwards;
}
```

Putting it all together:

```css=
/* @keyframes duration | easing-function | iteration-count | fill-mode | name */
.element {
  animation: 5s ease-out 500ms infinite forwards rotate;
}
```

This [example](https://codesandbox.io/p/sandbox/css-animation-vhp4q9?file=%2Fstyles.css%3A13%2C2) combines various parameters to create an animation named `rotate` with a duration of 5 second, easing out, a delay of 500 milliseconds, running infinite times, and pausing the animation in the end.

## Animation Event

- `animationstart`
- `animationend`
- `animationiteration`
- `animationcancel`

## Difficulties

- rotate to certain position
- smooth animation
  - rate
  - position

## More Info

- [Animate.style](https://animate.style/)
- [Animation examples](https://blog.hubspot.com/website/css-animation-examples)

## Conclusion

- strong
- interesting
- start and try

## References

- [Animation-W3C](https://www.w3.org/TR/css-animations-1/)
- [Animation-MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations)
