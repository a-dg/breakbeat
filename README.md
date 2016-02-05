# breakbeat()

The `breakbeat()` mixin allows you to build media queries simply. It is designed to work with [Foundation 6](https://github.com/zurb/foundation-sites), and offers some additional functionality beyond the existing `breakpoint()` mixin provided with Foundation.

It’s usually as simple as this:

```scss
@include b('>= large') {}
```


## Capabilities

- Generate media queries for width and height
- Use breakpoint names and values already defined for use within Foundation
- Succinctly specify ranges between named breakpoints
- Scale breakpoint values to target portions of breakpoints
- Automatically filter out meaningless rules, like `(min-width: 0)`
- Output rules as `em` (the default) or `px`


## Configuration

Begin by setting the media query output type, and the global font size. The font size can be expressed as either pixels or a percentage.

```scss
$output_em_queries: true; // Outputs px if false
$global_font_size: 16px;
```

Define your breakpoints, using the minimum width value for each range. Breakpoints must be defined in ascending order, and the first value should almost certainly be `0`.

```scss
$breakpoints: (
  tiny: 0,
  small: 460px,
  medium: 640px,
  average: 820px,
  large: 1024px,
  xlarge: 1280px,
);
```

Optionally define a separate set of height breakpoints, using the same guidelines as the width breakpoints. Height breakpoints are completely separate from width breakpoints, so their names and values can be the same or totally different, it doesn’t matter.

```scss
$y_breakpoints: (
  wee: 0,
  petite: 300px,
  medium: 460px,
);
```

Finally, import [_breakbeat.scss](scss/_breakbeat.scss).

```scss
@import 'breakbeat';
```


## Usage

To use the mixin, specify a [comparison operator](#comparison-operators), followed by the target breakpoint, optionally prefixed by the axis.

```scss
// Greater than or equal to small
@include breakbeat('>= small') {}

// Output
@media (min-width: 460px) {}
```

There’s also a handy shortcut:

```scss
@include b('>= small') {}
```

For [height media queries](#height-media-queries), specify the axis as `y` or `h`. (For width media queries, specifying `x` or `w` will work, but they’re optional.)

```scss
@include b('y >= petite') {}
```


## Comparison operators

`breakbeat()` uses common comparison operators to isolate media queries to a specific range, with one addition:

- `>=`: greater than or equal to (results in `min-width`)
- `<=`: less than or equal to (results in `max-width`)
- `>`: greater than (results in `min-width`, using the next largest breakpoint)
- `<`: less than (results in `max-width`, using the next smallest breakpoint)
- `=`, `==`, `===`: equal to (results in both `min-width` and `max-width` for a single breakpoint)
- `><`: between (results in `min-width` and `max-width` for multiple breakpoints)

Note that the `>` and `<` operators *exclude* the breakpoint passed to the mixin. For example, `b('>= small')` will include the entire `small` breakpoint within its range, but `b('> small')` — *greater than small* — will begin with the bottom end of the next largest breakpoint, excluding `small` entirely. If the next largest breakpoint is `medium`, then `b('> small')` will produce a result identical to `b('>= medium')`.

### Between operator

The `><` operator takes two breakpoint names as arguments, and includes both of them within the resulting range. In the following example, the entire span of `small`, `medium`, and `average` breakpoints are included in the resulting media query. The next largest breakpoint, `large`, has a minimum width of `1024px`, so the top end of `average` is the maximum, at `1023px`.

```scss
// Between small and average, inclusive
@include b('>< small average') {}

// Output
@media (min-width: 460px) and (max-width: 1023px) {}
```


## Height media queries

For height media queries, specify the axis as `y` or `h` before the comparison operator.

```scss
@include b('y >= petite') {}
```

If you store a separate set of height breakpoints as `$y_breakpoints`, those will be used for all height media queries. If there is no separate set of height breakpoints, `breakbeat()` will use the same names and values specified in the standard `$breakpoints` variable.


## Scaling ranges

To allow targeting specific portions of breakpoints, `breakbeat()` accepts a `scale` argument that modifies the resulting range.

Note that `scale` does not simply perform multiplication on the initial value of a breakpoint, but rather it controls the *deviation* from a breakpoint’s initial value with respect to the adjacent breakpoints.

Let’s take a simple example, with some simple breakpoints:

```scss
$breakpoints: (
  small: 0,
  medium: 600px,
  large: 800px,
);
```

Here, the distance covered between `small` and `medium` is `600px`, whereas the distance covered between `medium` and `large` is only `200px`.

To target the upper half of `small`, we scale the range by `0.5`:

```scss
// Scale up the initial width of small by half of
// the distance to the next largest breakpoint
@include b('= small', 0.5) {}

// Output
@media (min-width: 300px) and (max-width: 599px) {}
```

The result is that we’ve narrowed the range of the media query to affect only the upper half of the range normally covered by `small`.

The same technique applied to the `medium` breakpoint results in a much smaller range, since its distance from the `large` breakpoint is smaller:

```scss
// Scale up the initial width of medium by half of
// the distance to the next largest breakpoint
@include b('= medium', 0.5) {}

// Output
@media (min-width: 700px) and (max-width: 799px) {}
```

The same `scale` value applied to the `medium` breakpoint results in only a `99px` range, due to its proximity to the `large` breakpoint.

`scale` can be any number, but it is most useful when it’s some floating point value between `-1` and `1`. Positive values narrow the range toward the top end of a breakpoint, whereas negative values narrow the range toward the bottom end.

```scss
// Scale down the maximum width of medium by half of
// the distance to the next smallest breakpoint
@include b('= medium', -0.5) {}

// Output
@media (min-width: 600px) and (max-width: 699px) {}
```

Naturally, the comparison operator used affects the meaning of the `scale` value.

```scss
// Target the top one-third of medium, and larger
@include b('>= medium', 0.66) {}

// Output
@media (min-width: 732px) {}
```

There are endless possibilities for quickly tweaking problem areas within specific ranges, without having to create additional breakpoints.

>  Since scaling controls deviation from the initial value, scaling by `0` does not obliterate the breakpoint through multiplication, but alters it by a factor of `0`, just as if the `scale` argument were omitted. Likewise, scaling by `1` does not multiply the value by `1`, but rather offsets the initial value by `100%`.


## Querial detritus

Media queries are automatically corrected or weeded out if they don’t make any sense. Here are some example nonsensical arguments, and the resulting media queries.

```scss
$breakpoints: (
  small: 0,
  medium: 600px,
  large: 800px,
);

@include b('>= small')         // No media query
@include b('<= large')         // No media query
@include b('< small')          // @media (max-width: 599px)
@include b('> large')          // @media (min-width: 800px)
@include b('>< small large')   // No media query
@include b('>< medium large')  // @media (min-width: 600px)
```


## Breakpoint name interpolation

Since breakpoint names are passed to `breakbeat()` as part of a quoted string, you can easily use meaningful variable names to identify the purpose of a media query, and enable sweeping changes to transition points.

```scss
$mobile_nav: tiny;
$tablet_nav: small;
$desktop_nav: medium;
@include b('>= #{$desktop_nav}') {}
```