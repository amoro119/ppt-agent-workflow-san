# Build and verify the bento output

The final artifact is one `.bento.html` file. The fidelity ledger belongs in
the handoff, not inside the document, unless the user explicitly asks for a
separate report file.

## 1. Acquire and check the shell

Use the user-supplied target shell when one is in scope. Otherwise download the
current signed bento/slides release and fetch its current agent guide. Verify
the shell contains exactly the plaintext document marker:

```html
<script type="application/bento+json" id="bento-doc">
```

Do not synthesize a new HTML shell, copy the runtime out of source files, or
replace the compressed payload. The shell is the app.

If the target is an existing populated bento deck:

- parse the existing document first;
- preserve `docId` and unknown fields;
- preserve file mode unless the user asked to change it;
- warn before proceeding if `collab` carries `ownerPriv`, `writerPriv`, or
  `invite`, because the file contains live-session credentials.

For a fresh shell, author a fresh document and omit `docId` and `collab` so the
app mints a new identity. Do not set `template` or `readonly` unless requested.

## 2. Write the document block safely

The document must match the current target guide. At minimum it needs the
target's required `format`, `version`, `title`, `size`, `theme`, and non-empty
`slides` structure, with fully specified required element fields.

When serializing into `#bento-doc`:

- replace only that block's text content;
- escape every `<` in the JSON as `\u003c`;
- ensure the block contains no literal `</script>`;
- leave every byte of the surrounding shell alone when practical;
- never regenerate an existing document's `docId`.

After writing, extract the block again and JSON-parse it. Verify every
`asset:` reference resolves and every internal `link` targets a slide id.

## 3. Validate in the real shell

Open the generated `.bento.html` in a browser and run:

```js
const result = window.bento.validate()
result.findings.filter(f => f.severity !== 'info')
```

Resolve every error. Review every warning rather than deleting fields merely
to make the list shorter. Validation is read-only and reports, among other
things, text overflow, off-canvas elements, unknown fields, duplicate ids,
morph-key collisions, broken links/assets, unsupported chart options, and
effects that cannot run.

For text boxes that differ from the source or overflow, use the target shell's
real measurement API:

```js
window.bento.measure({ html, w, h, fontSize, fontFamily, fontWeight, lineHeight, letterSpacing })
```

Do not substitute character counts, canvas `measureText`, or guessed line
counts for the renderer's answer.

## 4. Visual comparison

Render source and output at matching presentation viewport/aspect. Compare
every slide and distinct state, preferably with side-by-side screenshots or an
overlay. Check:

- slide count, state count, and navigation order;
- stage size/aspect and background treatment;
- text content, wrapping, clipping, baselines, and font fallback;
- geometry, spacing, rotation, z-order, opacity, gradients, shadows, blur, and
  blend behavior;
- image crop/object-fit and svg rendering;
- table structure and chart labels/data;
- media poster, controls, autoplay, mute, and loop behavior;
- incremental builds, transitions, internal links, state return behavior, and
  hover interactions.

JSON similarity is not visual verification. A property can be accepted and
still paint differently; a missing font can look correct only on the converter's
machine; a screenshot can be captured before fonts or media posters settle.

## 5. Offline and safety check

Unless the user accepted network-dependent media, reload the deck with network
access disabled. Every visible image, svg, and font should still render. Confirm
that no source JavaScript, event handler, framework bundle, iframe, or unsafe
HTML was copied into the document data.

If the result intentionally links large media, list each network-dependent URL
or at least each affected slide in the handoff.

## 6. Fidelity review

Reconcile the working ledger against the final deck. For every
`approximated`/`dropped` item, verify that:

- the source location is identifiable;
- the statement describes the final output, not an abandoned attempt;
- repeated occurrences have a count;
- no source content or interaction disappeared without an entry.

Typical final summary:

```text
carried: 46
approximated: 5
  - responsive-layout-frozen — deck — rendered at 1440x810
  - fragment-expanded-to-slides — slides 3 and 7 — 6 build states became slides
  - canvas-rasterized — slide 9 / revenue chart — source data unavailable
dropped: 1
  - external-link-not-native — slide 12 / CTA — visible URL kept; click omitted
```

Exact counts may represent grouped occurrences, but the grouping must be clear.

## 7. Handoff

Return:

- a link/path to the `.bento.html` output;
- source slide/state count -> output slide/state count;
- whether the deck is fully self-contained/offline;
- validation counts and whether all errors were resolved;
- a concise list of approximations and losses;
- confirmation that every slide and interaction was exercised.

Do not say "pixel-perfect" unless a same-viewport rendered comparison supports
that claim. Do not say "fully editable" if any visible layer was emitted as an
opaque svg or raster image.
