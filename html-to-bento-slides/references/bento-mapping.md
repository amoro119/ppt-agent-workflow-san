# Map rendered HTML to bento/slides

Read the current target-shell guide first. This reference explains conversion
judgment; it is not a frozen copy of the complete bento schema.

## Coordinate system and slide structure

Prefer `doc.size` equal to the source's intended stage size. This preserves the
source geometry and avoids unnecessary scaling. If normalization is required,
use one uniform scale and letterbox/crop deliberately; never stretch x and y
independently without reporting the distortion.

For an object inside a rendered source slide:

1. subtract the slide frame's left/top from the object's rendered position;
2. undo any framework scale applied to the entire stage;
3. apply the same scale to width, height, border widths, radii, shadows, and
   typography;
4. map simple rotation to `rotation`;
5. place elements in browser paint order because bento array order is z-order.

Use one ordinary bento slide for each linear source slide. Manual incremental
builds become adjacent ordinary slides that duplicate the prior state and add
or change only what the next click reveals. Keep conceptual object ids stable
and use `transition: "morph"` when the source visibly animates continuity;
otherwise use a clean cut or the closest honest whole-slide transition.

Nonlinear click states become adjacent `stateOf` slides and internal `link`
targets. Hover swaps may use slide `hover` and element `showOnHover` when the
source groups map cleanly.

Map source presenter notes to `slide.notes`. A source slide that is deliberately
skipped from the linear show but remains independently reachable maps to
`hidden: true`; use `stateOf` only when it is a variant of a specific parent.
Infer `role` (`title`, `subtitle`, `body`, `kicker`) from source semantics where
that improves later editing without changing appearance. Keep repeated
headers, footers, logos, and page furniture on stable ids across slides.

## Element mapping

| Source | Preferred bento representation | Boundaries |
|---|---|---|
| Heading, paragraph, label | `text` | Preserve computed font, weight, size, color, alignment, line height, letter spacing, and explicit breaks. Split incompatible styled runs. |
| `<pre><code>` | `code` when the target shell supports it; otherwise monospace `text` | Preserve source text exactly. Syntax colors are not a reason to flatten the whole slide. |
| Rectangular panel/button/card | `shape: "rect"` plus separate text | Map fill, border, radius, opacity, shadow, blur/blend/backdrop fields when supported. |
| Circle/ellipse | `shape: "ellipse"` | Keep labels as separate text elements. |
| Rule, connector, simple arrow | line/path `shape` | Map dash and tips; use `from`/`to` only when the source relationship should remain editable. |
| CSS/SVG path or irregular vector | native path `shape` when simple; otherwise `svg` | Native paths edit and morph better; svg preserves complex artwork better. |
| `<img>` or CSS background image | `image` | Map `cover`/`contain`/`fill`; embed bytes in assets. Full-slide backgrounds are ordinary full-slide image elements. |
| Inline/external svg | `svg` asset | Remove executable/unsafe content and external dependencies. Prefer native shapes/text for diagrams meant to stay editable. |
| Rectangular table | `table` | Best for unmerged rows/columns. Rebuild merged, nested, or heavily styled tables as grouped native elements or svg. |
| Data chart with recoverable series | `chart` | Emit only options the target charts-lite guide supports. If data cannot be recovered, use svg/raster and report it. |
| `<video>` / `<audio>` | `media` | Embed short clips; link large ones and report the network dependency. Preserve poster and playback flags where behavior matches. |
| Canvas/WebGL | recover data and rebuild, else raster image | Canvas pixels carry no editable structure. Never claim native conversion from pixels alone. |
| iframe/live embedded page | recursively convert same-origin static content, else poster/screenshot plus visible URL | bento document data does not carry an arbitrary live iframe runtime. Report lost interactivity. |
| Decorative icon | native shape/path or `svg` | Keep as svg when the vector is already compact and self-contained. |

## Backgrounds and visual effects

- Solid slide background -> `slide.background`.
- Linear-gradient background -> a full-slide rect with `fillGradient`, because
  `slide.background` itself is one color.
- Background image -> a full-slide image element at the back of the array.
- Radial/conic gradient, complex mask, noise texture, or layered background ->
  self-contained svg when possible; otherwise rasterize only that layer.
- Ordinary box shadows map to element `shadow`. Multiple CSS shadows may map to
  a shadow array when supported.
- Simple blur, mix-blend, and backdrop blur map to the corresponding additive
  element fields when the target guide confirms them.
- Per-edge borders, inset shadows, blend chains, CSS masks, and complex filters
  usually need multiple native elements or an svg fallback.

Opacity belongs on the bento element. If the source uses parent opacity over a
subtree, either preserve it as a grouped opaque decoration or multiply opacity
into each emitted child consistently.

## Text and fonts

Use the visible text, not raw markup copied wholesale. Preserve meaningful
line breaks. bento text allows only a sanitized inline subset; arbitrary class,
style, link, and event attributes do not survive.

One bento text element has one base font family, size, weight, color, alignment,
and line height. Split source content into multiple aligned elements when a
single line mixes sizes, colors, families, superscript-like positioning, or
other formatting the inline subset cannot express. Use an svg fallback only
for typographic artwork whose exact run geometry matters more than editing.

For fonts:

- embed available WOFF2 bytes in `doc.assets` and register them in `doc.fonts`;
- keep a complete CSS fallback stack;
- if the font bytes are inaccessible or the license forbids redistribution,
  choose a metric-compatible/system fallback and record `font-substituted`;
- never assume a font is portable merely because it renders on the converter's
  machine.

Run bento's real text measurement after import. Source and target line-breaking
can differ even with the same nominal CSS because font loading and renderer
rules differ.

## Assets

Resolve asset URLs relative to the source document or CSS file that declares
them. Decode data URLs directly. Embed each unique payload once in `doc.assets`
and reference it as `asset:<key>`.

For svg assets, inline needed images/fonts or convert them to data URLs. Remove
scripts, event attributes, `foreignObject` HTML, unsafe URLs, and unresolved
external `<use>` references. The bento renderer sanitizes svg, but the converter
should still emit deliberately safe, self-contained markup rather than relying
on downstream deletion.

Do not fetch tracking pixels, analytics, ads, or unrelated preload resources.
If a visible remote asset cannot be obtained, use a clearly named placeholder
only when that is more informative than omission, and record a dropped entry.

## Interactions and animation

Use semantics rather than matching library names:

- whole-slide hard cut/fade/slide/zoom -> nearest bento transition;
- object continuity across states -> stable `id` or `morphId` plus morph;
- timed fade/slide on arrival -> `fx.enter` when timing need not be click-driven;
- slow image zoom/pan -> ken-burns when behavior matches;
- line dash movement -> `dash-march` on a dashed path;
- path-following loop -> motion-path;
- source step/fragment advanced by the presenter -> adjacent build slides, not
  a timed entrance;
- click drill-down/modal -> linked `stateOf` slide;
- hover reveal/focus -> `hover`, `group`, and `showOnHover`;
- internal anchor/route -> element `link` to the mapped slide id.

CSS keyframes and arbitrary JavaScript timelines do not transfer as code.
Re-express supported motion in bento, preserve contained safe svg animation
when an opaque svg is the right representation, or report the lost behavior.
Do not add animation absent from the source during a fidelity conversion.

Native bento links are internal slide links. For an external hyperlink, retain
the source's visible label without exposing a previously hidden destination on
the slide. Keep the URL in the fidelity handoff or speaker notes and record the
missing native click. A safe svg `<a>` may preserve the click inside an opaque
svg object, but use it only when the link is essential and non-native editing
is acceptable.

## Deterministic identity

Ids must be stable and reproducible. Derive them from source slide identity and
semantic/source identity, then sanitize to a compact stable token. Do not derive
ids from array position alone when inserting one source node would renumber all
later objects.

Use the same id across build/state slides only for the same conceptual object.
Use `morphId` when source instances are semantically the same but need distinct
element ids. Ensure ids and effective morph keys are unique within each slide.

## Fidelity ledger

Use the bento conversion convention:

- `carried`: behavior/meaning was represented without material loss;
- `approximated`: the audience sees a reasonable substitute, including svg or
  raster flattening of an otherwise editable object;
- `dropped`: content or behavior could not be represented.

Each non-carried entry needs a stable kebab-case code, source location, and one
actionable sentence. Examples include `font-substituted`,
`responsive-layout-frozen`, `text-runs-split`, `canvas-rasterized`,
`external-link-not-native`, `fragment-expanded-to-slides`,
`animation-approximated`, and `remote-asset-missing`.

Do not flood the report with exact per-element mappings. Fold repeated entries
by code and source slide, but never hide the count.
