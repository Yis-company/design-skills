# HTML Report Format

The design-system audit is rendered as a single self-contained HTML file in the OS temp directory. Tailwind comes from a CDN. Every visual is hand-built divs and inline SVG, drawn from the **real values** in the codebase: a swatch shows the actual hex, a type sample uses the actual size and weight. The reader should see the drift, not read about it.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Design-system audit for {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly */
      .swatch { width: 3rem; height: 3rem; border-radius: 0.375rem; box-shadow: inset 0 0 0 1px rgb(0 0 0 / 0.1); }
      .drift { outline: 2px dashed #dc2626; outline-offset: 2px; }
      .checker { background: repeating-conic-gradient(#e7e5e4 0 25%, #fff 0 50%) 0 0 / 12px 12px; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, and a compact legend: red dashed outline = drift, solid swatch = token value, `token-name` in mono = defined token, raw value in mono = hardcoded. No introduction paragraph. Straight into the candidates.

## Candidate card

The visuals carry the weight. Prose is sparse, plain, and uses the design-system terms without ceremony.

Each candidate is one `<article>`:

- **Title**: short, names the fix (e.g. "Fold six danger reds into `color-text-danger`").
- **Badge row**: recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the drift kind (`hardcoded value`, `near-duplicate`, `one-off variant`, `unused token`, `API divergence`).
- **Files**: monospaced list, `font-mono text-sm`, with a use-site count.
- **Before / After visual**: the centrepiece. Two columns, side by side. See patterns below.
- **Drift**: one sentence. What has parted from the system.
- **Proposal**: one sentence. What changes.
- **Benefit**: bullets, 6 words or fewer each. e.g. "One red for every error", "Delete 2 near-duplicate cards", "Variant matches the design source".
- **ADR callout** (if applicable): one line in an amber-tinted box.

No paragraphs of explanation. If the visual needs a paragraph to be understood, redraw it.

## Visual patterns

Pick the pattern that fits the drift. Mix them.

### Swatch row (colour drift)

Before: every distinct value found at use sites, one swatch each, with the raw value and its use count beneath, the off-token ones outlined `.drift`. After: the semantic token's single swatch, with the token name and the total count. Put text on the swatch when contrast is the point.

### Rendered type scale (type drift)

Before: each size/weight/line-height combination in use, rendered as a line of real sample copy, labelled with its values; off-scale lines outlined `.drift`. After: the scale as the tokens define it, same sample copy.

### Spacing bars (spacing drift)

Horizontal bars at true width for each spacing value in use. Before: the scale plus the off-scale values wedged between steps. After: the scale alone, with each off-scale value mapped to its nearest step.

### Variant grid (component drift)

Rebuild each near-duplicate or overridden component in HTML at its real size, side by side, labelled with its file. After: the one component with its variants in a row, each labelled with its prop value. For API divergence, put a two-column table under it: design-source variants and slots on the left, code props on the right, mismatches in red.

### Token inventory (unused tokens)

A compact grid of the unused tokens, each as a swatch or sample with its name, struck through in the After column.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly in the chrome (one accent, red for drift, amber for warnings) so the audited colours stand out.
- Keep each before/after around 320px tall so the two sit side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for labels, so they read as annotation, not as UI.
- The only script is the Tailwind CDN. The report is otherwise static: no app code, no interactivity.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain English, concise, with the nouns straight from the `design-system` skill. Concision is not an excuse to drift.

**Use exactly:** token (primitive, semantic, component), component, variant, slot, primitive, composite, drift, one-off, source of truth.

**Never substitute:** variable, style (for token) · widget, element (for component) · modifier, flavour (for variant) · inconsistency, tech debt (for drift).

**Phrasings that fit the style:**

- "Six reds where `color-text-danger` exists."
- "`PromoCard` is a near-duplicate of `Card`: one prop apart."
- "One use: keep it a one-off until a second shows up."
- "Code `Button` has no `tertiary` variant; the design source does."

**Benefit bullets** name the gain in those terms: *"one semantic token, 14 use sites"*, *"two components become one variant"*, *"code API matches the design source"*. Don't write *"cleaner UI"* or *"more polished"*, because those don't say what changed.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it.
