# Analyze the source HTML presentation

The source has two truths: its files explain intent, while the running page
shows the result. Use both. A DOM walk without rendering misses computed styles,
pseudo-elements, webfonts, stacking contexts, transforms, canvas output, and
JavaScript-created states. Screenshots alone lose content and behavior.

## 1. Inventory before rendering

Locate the real entry point and record:

- HTML entry files and any alternate presentation routes;
- linked and inline CSS;
- scripts, modules, and the presentation framework if one is used;
- local images, svg, fonts, video, and audio;
- remote assets or APIs required for the rendered result;
- presenter notes or source annotations that are not visible on the slide;
- explicit stage dimensions, aspect ratio, and framework configuration.

Common slide roots include Reveal's `.reveal .slides > section`, Marp-style
`section` pages, impress.js `.step` nodes, `.slide`/`[data-slide]` containers,
and hash- or route-driven bespoke apps. Treat these as clues, not universal
selectors. Nested Reveal sections are navigation structure, not necessarily
one flat slide each; inspect the actual forward order.

If ES modules, fetches, or browser security make `file://` render differently,
serve the source from a local static server. Do not modify the source merely to
make extraction easier.

If the presentation requires a build, inspect its package scripts and lockfile
before executing anything. Prefer an existing built artifact. Do not run
unreviewed lifecycle scripts or unrelated setup commands just because a source
README suggests them.

## 2. Render safely and establish the viewport

HTML presentations execute code. Open unfamiliar sources in an isolated or
temporary browser profile with no logged-in accounts, extensions, saved file
permissions, or sensitive local storage. Start offline when the source is
supposed to be self-contained; enable network access only for assets the user
placed in scope.

Use the source's declared canvas size when it has one. Otherwise use the
viewport the presentation was designed for or the viewport the user requests.
For responsive sources, the converted deck is a snapshot of one layout regime:
record the chosen viewport and do not pretend the result remains responsive.

Before measuring a state, wait for:

- `document.fonts.ready`;
- every relevant image to finish loading or fail visibly;
- framework initialization;
- layout-affecting transitions to settle;
- async content that is part of the presentation, within a bounded wait.

Do not wait forever for analytics, polling, ads, or an unavailable API. Record
missing content rather than inventing it.

## 3. Enumerate slides and states by operating the presentation

Walk the same path an audience member would:

1. start at the first slide;
2. advance one action at a time;
3. capture every visually distinct result;
4. follow internal navigation, modal/drill-down clicks, and hover states;
5. return to the linear path and continue to the end.

For each captured state record:

- source id/route/index and a human-readable name;
- slide frame and background;
- whether it is a linear slide, a build step, a nonlinear state, or a hover
  variant;
- incoming/outgoing transition behavior;
- visible objects and their paint order;
- speaker notes, if present;
- a rendered reference image.

Framework-specific notes often live outside the visible frame, such as
Reveal's notes element. Preserve them as source content and map them to the
bento slide's `notes` field rather than painting them on the slide.

A framework may keep all slides in the DOM and hide most of them. Visibility is
state-dependent: inspect the active state, not merely nodes whose source CSS
looks visible.

## 4. Capture rendered geometry and style

For each visible object, measure relative to the active slide frame rather than
the browser window. Record at least:

- `getBoundingClientRect()`;
- computed display, visibility, opacity, color, background, border, radius,
  shadow, filters, blend mode, font, line height, letter spacing, alignment,
  object fit, and transform;
- the object's text/content and asset source;
- its stacking context and overlap relationships;
- `::before` and `::after` content and styles;
- click, hover, keyboard, autoplay, and timer-driven behavior.

`getBoundingClientRect()` gives the transformed axis-aligned bounds. That is
enough for unrotated or uniformly scaled boxes, but it does not recover skew,
perspective, 3D transforms, or the untransformed box. Read the transform matrix
and source dimensions as well. Map simple rotation/scale; use svg or raster for
geometry bento cannot express, and report the approximation.

DOM order is not paint order once positioned descendants, z-index, opacity,
transforms, or filters create stacking contexts. Reconstruct the browser's
stacking order and use point sampling such as `elementsFromPoint()` where
overlap is ambiguous.

## 5. Classify content before conversion

Classify each source object by meaning and by representation:

- text, including distinct style runs and real line breaks;
- geometric decoration or diagram nodes;
- raster image;
- inline or external svg;
- tabular data;
- chart with recoverable source data;
- canvas/WebGL output without recoverable structure;
- code listing;
- video/audio;
- iframe or embedded live page;
- interaction target;
- background or decorative effect.

Important boundaries:

- Multi-style HTML text often cannot remain one bento text box. bento sanitizes
  text to a small inline subset and strips arbitrary span styles. Split runs
  into aligned text elements when editability matters; otherwise use svg for
  typographic artwork.
- A canvas chart is not automatically a chart. Rebuild it natively only when
  the labels/series/data can be recovered from source state; otherwise preserve
  its rendered output as svg or raster.
- Pseudo-elements are real visible objects even though they have no DOM node.
  Recreate them as shapes/text or absorb them into an opaque decoration.
- CSS masks, clip paths, individual-edge borders, radial/conic gradients,
  skew/perspective, and complex filters may need svg or raster treatment.
- A hidden source object required by a later state still belongs in the state
  inventory; it is not dead content.

## 6. Set the fidelity strategy

Choose treatment per object:

- **Native:** source semantics and appearance fit bento's model closely.
- **Hybrid:** keep meaningful text/data/media native and flatten only the
  decoration or unsupported visual effect.
- **Rasterized:** the object is inherently canvas/WebGL, uses unsupported
  geometry, or must be pixel-identical and editability is secondary.
- **Dropped:** no honest representation is available. Use only after checking
  whether a placeholder would communicate the missing content better.

Do not make one deck-wide choice unless every slide truly has the same needs.
The most useful result is usually mixed: native text and media, native simple
shapes, svg for complex vector artwork, and raster only for irreducible output.
