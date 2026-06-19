# Typography reference

Load this when a type decision needs an exact value, pairing, or correction. The judgment lives in `SKILL.md` (type carries most of the signal, match the typeface to the product, do not default to Inter out of habit); this is the catalog of how to execute it well.

---

## Building a type scale

Pick a base size (16px for most apps, 18px for reading-heavy products) and grow it by a ratio rather than by hand-picked numbers, so steps stay proportional as you add them.

| Ratio | Name | Feels |
|---|---|---|
| 1.2 | Minor third | Dense, utilitarian (dashboards, data tools) |
| 1.25 | Major third | Balanced default for product UI |
| 1.333 | Perfect fourth | Editorial, generous |
| 1.5 | Perfect fifth | Marketing, high drama between heading and body |

A bigger ratio gives more drama between heading and body but fewer usable mid-steps; a smaller ratio gives a calm, dense hierarchy. Use one ratio for the heading ramp and consider holding small UI text (labels, captions) off the same curve, since a strict ratio often produces a caption size too close to body to read as distinct. Round every computed value to a whole pixel; sub-pixel type sizes blur on non-retina screens.

## Line-height scales inverse to size

Line-height is a ratio of size, and the ratio shrinks as size grows. Large type already has vertical presence, so tight leading reads as deliberate; body text needs air between lines to track from end of one to start of the next.

| Role | Size | Line-height |
|---|---|---|
| Display / hero | 48px and up | 1.0 to 1.1 |
| Headings | 24 to 40px | 1.1 to 1.25 |
| Body | 16 to 20px | 1.5 to 1.6 |
| Dense UI / captions | 12 to 14px | 1.3 to 1.4 |

Line-height also rises with measure: a wide column needs more leading than a narrow one to keep the eye from dropping a line. Set `line-height` unitless (`1.5`, not `24px`) so it scales with the element's font-size.

## Measure (line length)

Hold body text between 45 and 75 characters per line, 66 being the classic target. Past 75 the eye loses the start of the next line; under 45 it jumps too often and rhythm breaks. Set it with `max-width` in `ch` units (`max-width: 66ch`) so it tracks the font rather than guessing pixels. For two-column or caption text, 40 to 50 characters is right.

## Pairing typefaces

The reliable move is contrast of role, not similarity. One face for display and one for everything else, chosen so they disagree clearly:

- A high-contrast display serif against a clean grotesque body.
- A geometric or humanist sans for text against a monospace for labels, code, and figures.
- A single strong family used across weights, which always works and is the safe default when unsure.

Avoid pairing two faces that are slightly different (two humanist sans, two transitional serifs); the eye reads it as a mistake rather than a choice. Keep the pairing to two families, three only if one is a monospace doing a narrow job. Match the pairing on x-height and apparent size: two faces at the same point size often look different sizes, so adjust until their lowercase letters align.

## Optical corrections

These are the difference between type that is set and type that is composed.

- **Tracking scales inverse to size.** Large display text needs negative tracking to close the gaps that look loose at scale (`letter-spacing: -0.02em` to `-0.04em` on headings). Small text and all-caps need positive tracking to stay legible (`0.05em` to `0.1em` on uppercase labels). Body text at reading size wants none.
- **Hang punctuation and bullets.** Pull quotation marks, bullets, and list markers into the margin so the text edge stays optically straight. A quote that starts with a flush `"` looks indented.
- **Kill widows and orphans.** A single word alone on the last line of a paragraph, or a heading's last word wrapping alone, looks broken. Use `text-wrap: balance` on headings and `text-wrap: pretty` on body to let the browser fix it, or insert a non-breaking space before the last word of a heading.
- **Number alignment.** Use `font-variant-numeric: tabular-nums` for any column of figures (tables, prices, timers, dashboards) so digits share a width and columns line up. Use proportional (default) figures in running prose. Use `slashed-zero` where a zero could be read as an O.
- **Real small caps.** `font-variant: small-caps` synthesizes thin, weak caps from lowercase. Use a font's true small-cap set (`font-feature-settings: "smcp"`) where it has one; otherwise set actual caps a size down with positive tracking.

## OpenType features worth turning on

Modern fonts ship features that are off by default. Reach for these where the face supports them:

- `"liga"` standard ligatures (on by default, leave on for text, off for code).
- `"calt"` contextual alternates, which fix awkward letter collisions.
- `"ss01"` and friends, stylistic sets, often where the characterful alternates live (a single-story `a`, a different `g`).
- `"onum"` oldstyle figures for running text, `"lnum"` lining figures for tables and all-caps settings.
- `"frac"` for real fractions instead of `1/2`.

## Variable fonts

A variable font carries a continuous range on one or more axes (weight, optical size, width) in a single file, so you get every weight without a request per weight. Two payoffs beyond file size:

- **Optical sizing (`opsz`).** A type designed with an optical axis thins its strokes and tightens its spacing at display sizes and thickens them for body. Bind it to size with `font-optical-sizing: auto` so the same family reads correctly from caption to hero.
- **Animatable weight.** Because the axis is continuous, you can transition `font-variation-settings: "wght" 400` to `700` smoothly, which a static font cannot do without swapping files and flashing.

## Font loading without the flash

A web font that arrives late causes either invisible text (FOIT) or a swap that shifts layout (FOUT). Control it:

- Set `font-display: swap` to show fallback text immediately, or `optional` to skip the swap entirely on slow connections (best for body text where a late swap is more annoying than a fallback).
- `<link rel="preload" as="font" crossorigin>` the one or two fonts above the fold so they start downloading before the CSS that references them.
- Pick a fallback in the same metrics family and tune it with `size-adjust`, `ascent-override`, and `descent-override` on an `@font-face` so the fallback occupies the same space as the web font. This removes the layout shift (CLS) the swap would otherwise cause.
- Subset to the characters and weights you actually use; a full font with every language is a large blocking download for a page that uses Latin and three weights.

## Rendering

- Do not set `-webkit-font-smoothing: antialiased` reflexively. It thins strokes, which can help light-on-dark but makes dark-on-light text look anemic. Test both, default to the browser's `auto`.
- Hint that text will animate with care; transforming text triggers re-rasterization and can blur it mid-animation. Animate a wrapping element's `opacity` or `transform`, not the text's own `font-size`.
- Render real text, not images of text, so it stays selectable, searchable, accessible, and crisp at every zoom and density.
```

