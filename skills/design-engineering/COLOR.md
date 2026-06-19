# Color reference

Load this when a color decision needs an exact value, a ramp, or a system. The judgment lives in `SKILL.md` (commit to a real palette, choose colors that mean something, one accent doing real work, weight maps to truth); this is the catalog of how to build it well.

---

## Work in OKLCH, not hex or HSL

OKLCH describes a color as lightness (L, 0 to 100%), chroma (C, roughly 0 to 0.37), and hue (H, 0 to 360deg) in a perceptually uniform space. Two consequences that matter every day:

- **Equal L looks equally light across hues.** In HSL, `hsl(60 100% 50%)` yellow and `hsl(240 100% 50%)` blue are both "50% lightness" yet the blue looks far darker. In OKLCH the same L reads as the same lightness, so a ramp built on even L steps looks even.
- **Interpolation stays clean.** Moving from `oklch(60% 0.1 250)` to `oklch(90% 0.1 250)` holds the hue and only brightens; the hex path `#666` to `#eee` drifts grey and muddy through the middle.

Author in OKLCH and let the browser handle fallbacks, or precompute hex for old targets. The reasoning happens in OKLCH regardless.

## Building a neutral ramp

Neutrals are most of every interface (backgrounds, borders, text, dividers), so they carry more weight than the accent. Build them tinted, never pure grey.

1. Pick one hue for the whole ramp that matches the product temperature (cool 250 to 270, warm 40 to 80, green-grey around 150).
2. Hold chroma low and roughly constant (0.01 to 0.03). This is the tint; it is what separates a considered neutral from `#808080`.
3. Step lightness evenly for 10 to 12 stops from near-white to near-black. Even L steps in OKLCH read as even visual steps.

```css
--gray-1:  oklch(98% 0.005 260); /* app background */
--gray-2:  oklch(96% 0.008 260); /* subtle surface */
--gray-3:  oklch(92% 0.010 260); /* hover surface */
--gray-6:  oklch(70% 0.020 260); /* borders, dividers */
--gray-9:  oklch(45% 0.020 260); /* secondary text */
--gray-12: oklch(20% 0.020 260); /* primary text */
```

A 12-step scale (the Radix Colors model) assigns each step a job: 1 to 2 backgrounds, 3 to 5 component surfaces and hovers, 6 to 8 borders, 9 to 10 solid fills, 11 to 12 text. Naming by role rather than by raw lightness makes dark mode a remap rather than a rewrite.

## Building an accent ramp

Build the accent the same way: one hue, 10 to 12 lightness stops, but chroma rising toward the saturated middle of the ramp where the solid fills live (0.15 to 0.22 around L 55 to 65%), tapering at the light and dark ends where high chroma turns garish. An accent that shares its hue family with the neutrals reads as one coherent system; a fully independent accent hue can work but takes more care.

Most products need exactly one accent plus a neutral ramp plus three status hues (success green, warning amber, danger red). Six competing accent colors is the tell of an unconsidered palette. One accent doing real work, used at full strength only where it earns attention, beats a rainbow.

## From scale to semantic tokens

Never reference raw scale steps in components. Map them to named roles, so the meaning is stable and theming is one layer:

```css
--bg:            var(--gray-1);
--surface:       var(--gray-2);
--border:        var(--gray-6);
--text:          var(--gray-12);
--text-muted:    var(--gray-9);
--accent:        var(--accent-9);
--accent-hover:  var(--accent-10);
--accent-text:   var(--accent-11); /* accent-colored text on a light bg */
```

A component says `color: var(--text-muted)`, never `var(--gray-9)`. Now dark mode, a rebrand, or a contrast fix happens in the token layer and nothing downstream changes.

## Dark mode is a remap, not an inversion

Inverting lightness produces harsh, wrong dark mode. Build it as a deliberate second mapping of the same semantic tokens:

- **Never pure black or pure white.** Use a very dark tinted neutral for background (`oklch(18% 0.02 260)`) and an off-white for text (`oklch(92% 0.01 260)`). Pure `#000` with pure `#fff` text vibrates and crushes shadow detail.
- **Elevation gets lighter, not darker.** In light mode a raised surface casts a shadow; in dark mode shadows barely read, so a raised surface is a lighter neutral step. Higher elevation, lighter surface.
- **Desaturate accents slightly.** A chroma that sang on white is too loud on dark; pull chroma down and lightness up so the accent stays legible without glowing.
- **Re-check every contrast pair.** A pair that passed in light mode does not automatically pass inverted. Verify text-on-surface for both themes.
- **Recolor, do not just dim, shadows.** Dark-mode elevation leans on lighter surfaces and subtle borders more than on shadows.

## Contrast in practice

Hit the WCAG numbers in `SKILL.md` (4.5:1 normal text, 3:1 large text and UI components and focus rings), and beyond passing:

- **Test the real pair, not the swatch.** Text contrast is measured against the actual background behind it, including any tint, image, or overlay. Light grey placeholder text on a light input commonly fails.
- **Give text over images a guaranteed floor.** Use a gradient scrim or a solid panel behind text on photos so contrast does not depend on the image. Never rely on the picture being dark enough.
- **Do not encode meaning in hue alone.** Roughly 8% of men cannot separate red from green. Pair color with an icon, a label, a pattern, or position, so a status reads without color.
- **Adjust chroma when contrast fails.** Raising or lowering L is the usual fix; sometimes dropping chroma at constant L buys legibility while keeping the hue.

## Useful CSS color tools

- `color-mix(in oklch, var(--accent) 15%, var(--bg))` makes a tinted background from the accent without authoring a new token, ideal for the soft fill behind a selected or featured state.
- Relative color syntax derives one color from another: `oklch(from var(--accent) calc(l + 0.1) c h)` lightens the accent for a hover without a second variable.
- `@media (prefers-color-scheme: dark)` and a manual `[data-theme]` override should both flow through the same token layer, so the toggle and the system setting share one source of truth.
- Wrap wide-gamut colors so older displays get a fallback: `color: oklch(...)` after a hex or `rgb()` declaration, letting the cascade pick the best the display supports.

## Anti-patterns

- `#808080` or any pure grey at the center of a ramp: no tint, no character.
- Saturating the light and dark ends of an accent ramp: neon highlights, murky shadows.
- Pure black text on pure white, or its inversion in dark mode: harsh and fatiguing.
- A 20-step grey scale where half the steps are indistinguishable; 10 to 12 is plenty.
- Status conveyed by color only, with no icon or label backing it.
- Raw scale steps referenced directly in components, so a theme change means a find-and-replace.
```

