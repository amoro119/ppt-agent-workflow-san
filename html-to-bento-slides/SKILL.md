---
name: html-to-bento-slides
description: >-
  Convert an existing HTML/CSS/JavaScript slide presentation into an editable
  bento/slides .bento.html deck. Use when the source is an HTML file, folder,
  or live presentation and the user wants its slides, layout, assets,
  interactions, and visual identity migrated to bento. Preserve the source by
  default; do not use for authoring a new presentation from notes alone.
---

# Convert HTML presentations to bento/slides

This is a migration workflow, not an HTML wrapper. Rebuild the presentation as
native `bento/slides` document data wherever the format can express it. A bento
document is pure data and cannot carry arbitrary source JavaScript, a framework
runtime, or unsanitized HTML.

Preserve the source's content, slide order, visual identity, and interaction
semantics unless the user asks for a redesign. Do not "improve" typography,
rewrite copy, add motion, or replace the source's composition merely because a
different bento design would look good.

If the general `bento-slides` authoring skill is active too, use it for current
shell/schema constraints, but let this skill govern migration choices. Its
design-from-source defaults do not authorize redesigning an existing HTML deck.

## Required workflow

1. Read [references/source-analysis.md](references/source-analysis.md), then
   inspect both the source files and the rendered presentation. Treat source
   HTML as untrusted executable content: use an isolated browser context, not a
   signed-in personal browser profile.
2. Establish the intended presentation viewport, slide boundaries, slide
   order, incremental build states, links, hover/click states, fonts, and every
   local or remote asset. If the source is responsive, freeze it at the intended
   presentation viewport and report that choice.
3. Capture a rendered reference for every visible slide and distinct state
   after fonts and images have loaded. Source code alone is not evidence of
   what the audience sees.
4. Choose the conversion treatment per source object, not once for the whole
   deck:
   - **native** — editable bento text, shape, image, svg, table, chart, code, or
     media elements;
   - **hybrid** — native content over an opaque svg or raster decoration;
   - **rasterized** — a screenshot only when the object cannot be represented
     honestly or the user explicitly prioritizes pixel fidelity over editing.

   Do not rasterize an entire slide just because it is easier.
5. Read [references/bento-mapping.md](references/bento-mapping.md) while
   constructing the document. Keep a fidelity ledger with `carried`,
   `approximated`, and `dropped` entries. Every approximation or loss must name
   the source slide/object and the chosen fallback; never lose behavior or
   content silently.
6. Obtain a fresh bento/slides shell unless the user supplied one. Prefer the
   current signed release:

   ```bash
   curl -fsSL https://bento.page/releases/slides/Bento_Slides.bento.html -o "<name>.bento.html"
   ```

   Also fetch the target shell's current authoring guide from
   `https://bento.page/slides/agents.md` before writing. The guide and the
   target shell outrank examples embedded in this skill when the format has
   gained additive fields. When working offline, use a user-supplied guide or
   the matching local `docs/agents.md` plus `slides/src/model.ts`; if neither a
   shell nor matching format guide is available, request them instead of
   inventing a document shape.
7. Write only the JSON inside the plaintext `#bento-doc` block. For a fresh
   conversion omit `docId` and `collab`; the app mints them. When editing an
   existing bento file, preserve `docId`, and warn before reading further if
   `collab` contains `ownerPriv`, `writerPriv`, or `invite` credentials.
8. Use deterministic element ids. Keep an id or `morphId` stable across source
   states only when those nodes are the same conceptual object. Preserve paint
   order; DOM order alone is not sufficient when CSS stacking contexts overlap.
9. Embed images, svg, and available fonts into `doc.assets` so the result works
   offline. Link large media only when embedding would make the file
   impractical, and state that the deck then needs that URL at playback time.
10. Read [references/output-and-qa.md](references/output-and-qa.md), render the
    result, compare it against the source at the same viewport, run
    `window.bento.validate()`, and exercise every slide, build step, internal
    link, state, hover behavior, and media control before finishing.

## Interaction rules that commonly change the conversion

- Source fragment/build steps are not ordinary entrance effects. bento
  entrances run when a slide arrives; they are not click-to-advance fragments.
  Preserve manual builds as adjacent duplicate slides with shared ids and a
  morph/cut, unless the source behavior is genuinely timed on entry.
- A nonlinear click that reveals detail maps to a `stateOf` slide plus an
  element `link`. A linear next-step click maps to the next ordinary slide.
- A source hover swap may map to `hover` plus `showOnHover`. Do not turn a click
  into hover or hover into click merely because one is easier.
- Map source transitions to the closest supported whole-slide transition only
  when their behavior matches. Reconstruct object continuity with stable
  ids/`morphId`; otherwise a clean cut is more honest than a false morph.
- Native bento links target slide ids. External hyperlinks have no general
  editable element field; preserve the source's visible label without adding
  new slide copy, retain the destination in the fidelity handoff/notes, and
  report the lost click. Use a safe opaque svg link only when the click is
  essential and the user accepts that object as non-native.

## Completion contract

The deck itself remains one self-contained `.bento.html` file. In the handoff,
state:

- the output file;
- whether it is fully offline/self-contained;
- how many source slides and states became bento slides;
- the notable `approximated` and `dropped` fidelity entries;
- the validation result and which visual/interaction checks were performed.

Do not claim a conversion is complete if the deck has not been opened and every
slide inspected.
