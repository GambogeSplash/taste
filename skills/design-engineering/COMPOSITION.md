# Composition reference

Load this when a layout decision needs a structure, a grid, or a hierarchy method. The judgment lives in `SKILL.md` (lead with the frame, restraint when it is noise, spacing on a system, optical alignment); this is the catalog of how to compose well.

---

## Hierarchy: one clear entry point per screen

Every screen should answer "where do I look first" in under a second. If two elements compete for first place, neither wins and the screen feels flat. Establish order with several channels at once, not size alone:

- **Size and weight.** The most important element is largest or heaviest, but a small element in lots of empty space can outrank a large one in a crowd. Scale is relative to surroundings.
- **Color and contrast.** The single saturated element pulls the eye in a field of neutrals. This is why one accent doing real work beats six.
- **Position.** The eye starts top-left in left-to-right reading cultures and follows an F or Z path. Put the primary action where the path lands, not where the grid is symmetric.
- **Isolation.** Whitespace around an element is the cheapest emphasis there is. A button alone in a band of space reads as primary without any color.

A useful test: squint until detail blurs. The shapes that still stand out are your real hierarchy. If everything blurs to one even grey, there is no hierarchy yet.

## The grid

A grid is the invisible skeleton that makes a layout feel built rather than placed.

- **Columns.** 12 columns is the flexible default (it divides into halves, thirds, quarters, sixths). Marketing pages often want fewer, wider columns; dense apps want more. Define a gutter (the gap between columns, commonly 16 to 24px) and a margin (the space outside the outermost columns).
- **Align to it, then break it on purpose.** Most elements snap to column edges. A deliberate break (one element bleeding past the grid, an asymmetric hero) reads as intentional energy precisely because the rest holds the line. An accidental near-miss reads as a bug.
- **A baseline grid for text.** Snapping line boxes to a consistent vertical rhythm (a multiple of your base spacing) makes columns of text line up across a layout. Worth it in editorial and content-dense work; optional in app chrome.
- **Container queries over viewport queries for components.** A card should respond to the width of its container, not the window, so it composes correctly in a sidebar, a grid, or full width. `@container` is the right tool; viewport media queries are for page-level layout.

## Spacing as a system

Spacing is composition's main instrument. The numbers are in `SKILL.md` (4px base, 8pt grid, steps at least ~25% apart). Beyond the scale:

- **Proximity encodes grouping.** Related things sit close, unrelated things sit apart, and the eye reads the grouping before it reads a single word. This is more powerful than borders or boxes for showing structure. A label tight to its input and far from the next field needs no divider.
- **Inner padding is less than or equal to outer spacing.** An element's internal padding should not exceed the space around it, or the group dissolves into its neighbors. This one rule fixes most "everything feels mushy" layouts.
- **Space scales with the size of what it separates.** The gap between two sections is larger than the gap between two cards, which is larger than the gap between two lines. Reusing one gap everywhere flattens hierarchy.

## Whitespace and density

Whitespace is not wasted space; it is the silence that makes the content audible. But density is a deliberate choice tuned to the audience, not a fixed virtue.

- **Generous (marketing, onboarding, reading).** Large type, wide margins, one idea per viewport. The product feels calm and confident, and the message lands.
- **Dense (dashboards, tools, tables, power users).** Tighter spacing, smaller type, more per screen, because an expert scanning a hundred rows wants information density, and forcing airy spacing on them is its own usability failure. Density done well is still systematic: consistent row heights, aligned columns, clear grouping, just at a tighter scale.

Match the density to the job. A trading terminal and a landing page are both correct and look nothing alike.

## Scannability

Most users scan before they read. Compose for the scan:

- **Front-load the line.** Put the information-carrying word first in a label, heading, or list item, so a scan down the left edge surfaces meaning. "Delivery: 3 days," not "It takes 3 days for delivery."
- **Break prose into chunks.** Short paragraphs, subheadings every few paragraphs, lists for anything enumerable. A wall of text is skipped.
- **Make headings do work.** A reader should get the gist from headings alone. Descriptive beats clever for anything functional.
- **One idea per row.** In lists and tables, keep each row to a single scannable unit, with the differentiating value in a consistent position so the eye can run straight down it.

## Alignment and the optical layer

- **Pick an edge and hold it.** A strong shared alignment edge (usually left) does more for order than centering. Centered blocks have a ragged left edge that the eye has to re-find on every line, so reserve centering for short, symmetric content (a hero headline, an empty state), not for paragraphs or form fields.
- **Optical over mathematical.** Trust the eye over the bounding box. A play triangle centered by its rectangle looks left of center and needs a nudge right. A capital letter and a circle of the "same" size need the round shape slightly larger to look equal. Overshoot is correct: round and pointed shapes must extend slightly past a flat baseline to appear aligned with it.
- **Align to content, not to the box.** Padding that is mathematically equal can look unequal when a glyph has more space on one side; adjust until it looks centered, then move on.

## Cardlessness: structure without boxes

The reflexive layout (wrap every group in a bordered, shadowed card) fragments a page into competing rectangles and reads as assembled rather than designed. Reach for lighter structure first:

- **Whitespace and grouping** to separate sections, no border needed.
- **A hairline divider** or a subtle background-tint band where a boundary genuinely helps.
- **Alignment and rhythm** to imply the grid that a card would otherwise draw.

Use a real card when an element is genuinely a discrete, liftable object (a draggable item, a selectable tile, a piece of content in a feed). A card earns its border by being a thing; a card around a form section is just noise. When cards are right, keep their elevation consistent and nest their radius correctly (inner radius equals outer minus padding).

## Responsive composition

- **Design the small screen first.** It forces a priority order: what is essential survives the narrow column, and the wide layout becomes that priority order with room added, rather than the desktop layout crammed down.
- **Reflow, do not just shrink.** A three-column grid becomes one column stacked in reading order, not three tiny columns. Decide the source order so the stack reads correctly without CSS.
- **Tap-friendly at every width.** Hit targets and spacing from `SKILL.md` apply on touch; a layout that works with a cursor can still fail a thumb.
- **Let intrinsic layout do the work.** `flex-wrap`, `grid` with `auto-fit`/`minmax`, and `clamp()` for fluid type and spacing remove most breakpoints. Fewer hard breakpoints means fewer awkward in-between widths.
```

