# figma-token-sync

Change a color, a spacing value, or a type style in Figma, run one command, and the site's CSS updates to match.

## the problem

Figma and code drift apart. A designer updates the brand red or tightens a heading, an engineer copies the values over by hand, and some of them never make it. Six months later the site and the design file disagree and nobody knows which one is right.

This pipeline makes Figma the source of truth. Design tokens live as Figma Variables (colors, themes, grid, type, motion). One command pulls them into the codebase and regenerates the CSS. Every sync is checked against a strict schema first, so a bad export fails loudly instead of silently shipping wrong type.

## before and after

A designer nudges the brand red in Figma and exports. The snapshot changes:

```diff
 "red": {
-  "r": 0.9120299816131592,
-  "g": 0.08732099831104279,
-  "b": 0.10317300260066986
+  "r": 0.86,
+  "g": 0.12,
+  "b": 0.11
 }
```

Run the sync:

```bash
bun run figma:import
```

The generated CSS follows, everywhere that red is used:

```diff
 :root {
-  --color-red: oklch(0.592 0.2339 27.95);
+  --color-red: oklch(0.571 0.2193 28.26);
```

Same story for the grid, type sizes, and motion durations. No hand-copying, no "is this the current value" Slack thread.

## run it

The repo bundles a real export ([`figma/figma-tokens.json`](figma/figma-tokens.json)), so you can run the whole pipeline without Figma access or an API key. You need [Bun](https://bun.sh).

```bash
bun install
bun run figma:import
```

That validates the snapshot, regenerates the source config in `styles/`, and writes `css/root.css`, `css/theme.css`, and `css/typography.css`. Open them to see the palette, the theme overrides, the fluid grid variables, and one CSS class per text style.

## how the pieces fit

| Piece | What it does |
|---|---|
| [`figma/export.figma.js`](figma/export.figma.js) | Runs inside Figma (Plugin API). Reads the variable collections and text styles, returns the interchange JSON. |
| [`figma/figma-tokens.json`](figma/figma-tokens.json) | The committed snapshot of that export. The bundled sample. |
| [`figma/schema.ts`](figma/schema.ts) | The contract (Zod). A malformed or incomplete export fails here, with a clear error, before anything is written. |
| [`figma/import.ts`](figma/import.ts) | The importer. Converts Figma's sRGB to `oklch()`, regenerates the source config, rebuilds the CSS, and warns when a code override masks a value that changed in Figma. |
| [`styles/`](styles) | The regenerated config (`colors.ts`, `layout.mjs`, `typography.figma.ts`, `motion.ts`) plus the hand-authored typography override shell (`typography.ts`). |
| [`scripts/setup-styles.ts`](scripts/setup-styles.ts) | Compiles the config into plain CSS. |
| [`css/`](css) | The generated output. Never edited by hand. |

The pipeline's own docs, written alongside the code, go deeper: [`figma/README.md`](figma/README.md). They cover the Figma file structure, the typography constraints (why line-height can't be a bound variable, how vertical trim maps to `text-box-trim`), and the code override layer.

## provenance

Built as a pull request against the [satus](https://github.com/darkroomengineering/satus) starter, extracted here to stand on its own. In that integration the same config fed satus's Tailwind v4 generator, and every sync was additionally gated by a WCAG AA contrast test. The generator here is a minimal plain-CSS stand-in; the importer, schema, and Figma export are unchanged.

MIT licensed.
