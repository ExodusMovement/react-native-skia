---
id: image-svg
title: SVG Images
sidebar_label: SVG
slug: /images-svg
---

Draw an SVG (see [SVG Support](#svg-support)).

If the root dimensions are in absolute units, the width/height properties have no effect since the initial viewport is fixed.

| Name      | Type      |  Description                                                  |
|:----------|:----------|:--------------------------------------------------------------|
| svg       | `SVG` | SVG Image. |
| width?    | `number`  | Width of the destination image. This is used to resolve the initial viewport when the root SVG width is specified in relative units. |
| height?   | `number`  | Height of the destination image. This is used to resolve the initial viewport when the root SVG height is specified in relative units.                              |
| x?    | `number`  | Optional displayed x coordinate of the svg container.  |
| y?   | `number`  | Optional displayed y coordinate of the svg container.                            |

:::info

The `ImageSVG` component doesn't follow the same painting rules as other components.
[see applying effects](#applying-effects).

:::

### Example

```tsx twoslash
import {
  Canvas,
  ImageSVG,
  useSVG
} from "@exodus/react-native-skia";

const ImageSVGDemo = () => {
  // Alternatively, you can pass an SVG URL directly
  // for instance: const svg = useSVG("https://upload.wikimedia.org/wikipedia/commons/f/fd/Ghostscript_Tiger.svg");
  const svg = useSVG(require("../../assets/tiger.svg"));
  return (
    <Canvas style={{ flex: 1 }}>
      { svg && (
        <ImageSVG
          svg={svg}
          width={256}
          height={256}
        />)
      }
    </Canvas>
  );
};
```

You can also use an inlined string as SVG (using `Skia.SVG.MakeFromString`):

```tsx twoslash
import React from "react";
import { Canvas, ImageSVG, Skia } from "@exodus/react-native-skia";

const svg = Skia.SVG.MakeFromString(
  `<svg viewBox='0 0 290 500' xmlns='http://www.w3.org/2000/svg'>
    <circle cx='31' cy='325' r='120px' fill='#c02aaa'/>
  </svg>`
)!;

export const SVG = () => {
  return (
    <Canvas style={{ flex: 1 }}>
      <ImageSVG
        svg={svg}
        x={0}
        y={0}
        width={290}
        height={500}
      />
    </Canvas>
  );
};
```

## Scaling the SVG

As mentionned above, if the root dimensions are in absolute units, the width/height properties have no effect since the initial viewport is fixed. However you can access these values and use the fitbox function.

### Example

In the example below we scale the SVG to the canvas width and height.

```tsx twoslash
import React from "react";
import { Canvas, ImageSVG, Skia, rect, fitbox, Group } from "@exodus/react-native-skia";

const svg = Skia.SVG.MakeFromString(
  `<svg viewBox='0 0 20 20' width="20" height="20" xmlns='http://www.w3.org/2000/svg'>
    <circle cx='10' cy='10' r='10' fill='#00ffff'/>
  </svg>`
)!;

const width = 256;
const height = 256;
const src = rect(0, 0, svg.width(), svg.height());
const dst = rect(0, 0, width, height);

export const SVG = () => {
  return (
    <Canvas style={{ flex: 1 }}>
    <Group transform={fitbox("contain", src, dst)}>
      <ImageSVG svg={svg} x={0} y={0} width={20} height={20} />
      </Group>
    </Canvas>
  );
};
```

<img src={require("/static/img/svg.png").default} width="256" height="256" />

## Applying Effects

The `ImageSVG` component doesn't follow the same painting rules as other components.
This is because behind the scene, we use the SVG module from Skia.
However you can apply effets using the `layer` property.

### Opacity Example

In the example below we apply an opacity effect via the `ColorMatrix` component.

```tsx twoslash
import React from "react";
import { Canvas, ImageSVG, Skia, rect, fitbox, useSVG, Group, Paint, OpacityMatrix, ColorMatrix } from "@exodus/react-native-skia";

const width = 256;
const height = 256;

export const SVG = () => {
  const tiger = useSVG(require("./tiger.svg"));
  if (!tiger) {
    return null;
  }
  const src = rect(0, 0, tiger.width(), tiger.height());
  const dst = rect(0, 0, width, height);
  return (
    <Canvas style={{ flex: 1 }}>
      <Group
        transform={fitbox("contain", src, dst)}
        layer={<Paint><ColorMatrix matrix={OpacityMatrix(0.5)} /></Paint>}
      >
        <ImageSVG svg={tiger} x={0} y={0} width={800} height={800} />
      </Group>
    </Canvas>
  );
};
```

<img src={require("/static/img/opacity-tiger.png").default} width="256" height="256" />

### Blur Example

In the example below we apply a blur image filter to the SVG.

```tsx twoslash
import React from "react";
import { Canvas, ImageSVG, Skia, rect, fitbox, useSVG, Group, Paint, Blur } from "@exodus/react-native-skia";

const width = 256;
const height = 256;

export const SVG = () => {
  const tiger = useSVG(require("./tiger.svg"));
  if (!tiger) {
    return null;
  }
  const src = rect(0, 0, tiger.width(), tiger.height());
  const dst = rect(0, 0, width, height);
  return (
    <Canvas style={{ flex: 1 }}>
      <Group transform={fitbox("contain", src, dst)} layer={<Paint><Blur blur={10} /></Paint>}>
        <ImageSVG svg={tiger} x={0} y={0} width={800} height={800} />
      </Group>
    </Canvas>
  );
};
```

### Result

<img src={require("/static/img/blurred-tiger.png").default} width="256" height="256" />

## SVG Support

The [SVG module from Skia](https://github.com/google/skia/tree/main/modules/svg) displays SVGs as images.
We expect most SVG files to render correctly out of the box, especially if they come from Figma or Illustrator.
However, please be aware of some of the quirks below when using it.
Text elements current won't render and any external XML elements such as XLink or CSS won't render.
If your SVG doesn't render correctly and you've considered all the items below, please file [an issue](https://github.com/Shopify/react-native-skia/issues/new).

### Text

Currently text rendering is not supported

### CSS Styles

CSS styles included in SVG are not supported.
A tool like [SVGO](#using-svgo) can help with converting CSS style attributes to SVG attributes if possible. 
You can use it online [here](https://jakearchibald.github.io/svgomg/).
For instance, it can normalize CSS style attributes that contain transformations to the proper `transform` property.

### RGBA Colors

The RGBA color syntax is not supported. Instead, it would help if you used the `fill-opacity` and `stroke-opacity` attributes. Consider the example below.

```xml
<circle
  r="10"
  cx="10"
  cy="10"
  fill="rgba(100, 200, 300, 0.5)"
  stroke="rgba(100, 200, 300, 0.8)"
/>
```

Would need to be rewritten as:

```xml
<circle
  r="10"
  cx="10"
  cy="10"
  fill="rgb(100, 200, 300)"
  fill-opacity="0.5"
  stroke="rgb(100, 200, 300)"
  stroke-opacity="0.8"
/>
```

The `opacity` attribute also applies to both the `fill` and `stroke` attributes.

### Non Supported Elements

Below is the list of non-supported element. Often these SVGs can be rewritten to not use these elements.
  * `<altGlyph>` (deprecated)
  * `<animate>`
  * `<cursor>` (deprecated)
  * `<feComponentTransfer>`
  * `<feConvolveMatrix>`
  * `<feTile>`
  * `<feDropShadow>` 
  * `<font>` (deprecated)
  * `<foreignObject>`
  * `<glyph>` (deprecated)
  * `<script>`
  * `<view>`


### Font Family

When rendering your SVG with Skia, all fonts available in your app are also available to your SVG.
However, the way you can set the `font-family` attribute is as flexible as on the web.
```jsx
// ✅ This is really all that is supported:
<text font-family="MyFont" />
// ❌ This won't work. If MyFont is available, this syntax will be accepted.
// but it will never fallback to monospace
<text font-family="MyFont, monospace" />
// ❌ The single quote syntax won't work either.
<text font-family="'MyFont'" />
```

### Inlined SVGs

Some SVGs contain inlined SVGs via the `<image>` or `<feImage>` elements. This is not supported.

### Gradient Templates

The deprecated `xlink:href` attribute is not supported in gradients.
You can use the `href` attribute instead.
However, we found that it doesn't appear to be adequately supported.
We would recommend avoiding using it.

### Fallbacks

Some SVG with issues display nicely in the browser because they are very tolerant of errors. We found that the Skia SVG module is much less forgiving.
