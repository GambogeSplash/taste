# Motion reference

Load this when a motion decision needs an exact value, curve, or technique. The judgment lives in `SKILL.md`; this is the catalog of how to execute it well. Nothing here overrides the six tests (natural, fast, purposeful, 60fps, interruptible, accessible) or the sub-300ms ceiling.

---

## Easing curves

Built-in CSS keywords (`ease`, `ease-in`, `ease-out`) are too weak for deliberate UI motion; they lack the punch that reads as intentional. Reach for custom curves. A small named set covers almost everything:

```css
/* Strong ease-out, for UI interactions and most enter/exit */
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
/* Strong ease-in-out, for on-screen movement and morphs */
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
/* iOS-like drawer/sheet curve */
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
/* Material standard, a calmer all-rounder */
--ease-standard: cubic-bezier(0.4, 0, 0.2, 1);
```

Direction is the decision, the exact curve is the tuning. Things arriving decelerate (ease-out), things leaving accelerate (ease-in). **Never `ease-in` on a UI interaction:** it delays the first moment of movement, which is exactly the moment the user is watching, so a 300ms `ease-in` dropdown *feels* slower than the same 300ms `ease-out`. `linear` is only for spinners, marquees, and continuous progress.

## Duration budgets

| Element | Duration |
|---|---|
| Button / press feedback | 100 to 160ms |
| Tooltips, small popovers | 125 to 200ms |
| Dropdowns, selects | 150 to 250ms |
| Modals, drawers, sheets | 200 to 300ms (up to 500ms for large physical drawers) |
| Marketing / explanatory | Can be longer, it is the content |

UI motion stays under 300ms. A 180ms dropdown feels more responsive than a 400ms one. Exits run a touch faster than entrances (a common pairing is 300ms in, 200ms out) because the user has already moved on.

## Springs

Springs simulate physics, so they have no fixed duration; they settle from parameters. Use them for anything interruptible or gesture-driven: drags with momentum, elements that should feel alive, mouse-tracking decoration. Their advantage is that they keep velocity when interrupted, where fixed-duration easing restarts from zero and feels broken under rapid input.

```js
// Apple-style model, easier to reason about
{ type: "spring", duration: 0.5, bounce: 0.2 }
// Traditional physics, more control
{ type: "spring", mass: 1, stiffness: 100, damping: 10 }
```

Keep bounce subtle (0.1 to 0.3) and reserve it for drag-to-dismiss and playful surfaces; a professional dashboard wants no bounce. For mouse-driven values, interpolate through a spring rather than binding directly to the pointer, which feels artificial because it has no momentum:

```jsx
import { useSpring } from 'framer-motion';
// Binding directly feels instant and dead:
const rotation = mouseX * 0.1;
// Through a spring it gains momentum:
const springRotation = useSpring(mouseX * 0.1, { stiffness: 100, damping: 10 });
```

This is worth it only because the motion is decorative. A functional graph in a banking app wants no animation at all.

## Interruptible motion: transitions vs keyframes

CSS transitions can be interrupted and retargeted from their current position. Keyframes restart from zero. For anything triggered rapidly (toasts being added, toggles, drag state), use transitions or springs, not keyframes.

```css
/* Interruptible, good for dynamic UI */
.toast { transition: transform 400ms var(--ease-out); }

/* Restarts from zero on interrupt, avoid for dynamic UI */
@keyframes slideIn { from { transform: translateY(100%); } to { transform: translateY(0); } }
```

## CSS beats JS under load

CSS animations run off the main thread. When the browser is busy (loading a route, running scripts), JavaScript animations driven by `requestAnimationFrame` drop frames while CSS stays smooth. Use **CSS for predetermined motion, JS or springs for dynamic, interruptible, gesture-driven motion.**

A specific trap: Framer Motion's shorthand props (`x`, `y`, `scale`) are **not** hardware-accelerated. They animate on the main thread via rAF, so they drop frames when the page is busy. Animate the full `transform` string instead to stay on the compositor:

```jsx
<motion.div animate={{ x: 100 }} />                          // not hardware accelerated
<motion.div animate={{ transform: "translateX(100px)" }} />  // hardware accelerated
```

For programmatic motion that still wants CSS performance, use WAAPI: JS control, compositor execution, interruptible, no library.

```js
element.animate(
  [{ clipPath: 'inset(0 0 100% 0)' }, { clipPath: 'inset(0 0 0 0)' }],
  { duration: 1000, fill: 'forwards', easing: 'cubic-bezier(0.77, 0, 0.175, 1)' }
);
```

## Entry without JavaScript

`@starting-style` animates an element from its pre-mount values without the `useEffect(() => setMounted(true))` dance:

```css
.toast {
  opacity: 1; transform: translateY(0);
  transition: opacity 400ms var(--ease-out), transform 400ms var(--ease-out);
  @starting-style { opacity: 0; transform: translateY(100%); }
}
```

Fall back to the `data-mounted` attribute pattern where browser support is not there yet.

## Physical correctness

**Never animate from `scale(0)`.** Nothing in the real world appears from nothing, so it reads as coming out of nowhere. Start from `scale(0.9)` to `scale(0.97)` with opacity. Even a barely-visible initial scale gives the entrance a shape to grow from.

```css
.entering { transform: scale(0.95); opacity: 0; }  /* not scale(0) */
```

**Origin-aware popovers.** A popover, dropdown, or tooltip should scale in from its trigger, not from center. The default `transform-origin: center` is wrong for almost every anchored surface. Modals are the exception: they are not anchored, so they stay centered.

```css
.popover { transform-origin: var(--radix-popover-content-transform-origin); } /* Radix */
.popover { transform-origin: var(--transform-origin); }                       /* Base UI */
```

**Tooltips skip the delay on subsequent hovers.** The first tooltip delays to avoid accidental activation. Once one is open, hovering an adjacent trigger should open instantly with no animation, which makes the whole toolbar feel faster without defeating the initial delay (`transition-duration: 0ms` on a `data-instant` state).

**Asymmetric timing.** Slow the phase where the user is deciding, snap the phase where the system responds. A hold-to-delete press can run 2s linear; the release should snap back in ~200ms ease-out. Symmetric timing on a press-and-release or hold interaction is a tell.

## Masking imperfect crossfades with blur

When a crossfade between two states looks like two distinct objects overlapping, and no easing or duration fixes it, add a subtle `filter: blur(2px)` during the transition. Blur blends the two states so the eye reads one transformation instead of a swap. Keep blur under 20px, it is expensive, especially in Safari.

```css
.button-content { transition: filter 200ms ease, opacity 200ms ease; }
.button-content.transitioning { filter: blur(2px); opacity: 0.7; }
```

## clip-path as an animation tool

`clip-path: inset(top right bottom left)` defines a rectangular reveal; each value eats in from that side. It is one of the most useful and underused animation primitives.

```css
.hidden  { clip-path: inset(0 100% 0 0); } /* fully clipped from the right */
.visible { clip-path: inset(0 0 0 0); }    /* fully revealed */
```

- **Animated tabs with perfect color transition.** Duplicate the tab list, style the copy as the active state (different background and text color), then clip the copy so only the active tab shows. Animate the clip on tab change. This gives a seamless color swap that timing individual color transitions never matches.
- **Hold-to-delete.** A colored overlay at `inset(0 100% 0 0)`; on `:active` transition to `inset(0 0 0 0)` over 2s linear; on release snap back over 200ms ease-out. Pair with `scale(0.97)` on the button for press feedback.
- **Scroll reveals.** Start at `inset(0 0 100% 0)` (hidden from the bottom), animate to `inset(0 0 0 0)` on enter via `IntersectionObserver` or `useInView({ once: true, margin: "-100px" })`.
- **Comparison sliders.** Overlay two images, clip the top one with `inset(0 50% 0 0)`, drive the right inset from drag position. No extra DOM, fully compositor-accelerated.

## Transform mastery

- **Percentage translate is relative to the element's own size.** `translateY(100%)` moves an element by exactly its own height regardless of dimensions. This is how toasts enter from above the stack and how drawers hide off-screen before animating in. Prefer percentages over hardcoded pixels, they adapt to content.
- **`scale()` scales children too**, unlike `width`/`height`. Scaling a button on press scales its label and icons proportionally, which is what you want.
- **3D depth in pure CSS.** `rotateX()` / `rotateY()` with `transform-style: preserve-3d` give real depth (coin flips, orbits) without JavaScript.

## Gesture and drag

- **Momentum dismissal.** Do not require crossing a distance threshold. Compute velocity as `Math.abs(distance) / elapsedTime`; if it exceeds ~`0.11`, dismiss regardless of distance. A quick flick should be enough.
- **Damping at boundaries.** When the user drags past a natural limit, move the element less the further they pull. Real things slow before they stop, they do not hit an invisible wall.
- **Pointer capture.** Once a drag starts, capture pointer events so it continues even when the pointer leaves the element's bounds.
- **Multi-touch protection.** Ignore additional touch points after a drag begins, or switching fingers mid-drag jumps the element.

## Stagger

When a group enters together, stagger each item 30 to 80ms after the previous one for a cascade that feels more natural than everything at once. Keep delays short; long ones make the interface feel slow. Stagger is decorative, so never block interaction while it plays.

## Performance traps

- **Animate only `transform` and `opacity`** (and where needed `filter` / `clip-path`). These skip layout and paint. Animating `width`, `height`, `top`, `left`, `margin`, `padding` triggers all three stages and thrashes.
- **Do not drive a child transform from a parent CSS variable.** Changing a variable on a parent recalculates styles for every child. In a list, update `transform` directly on the moving element instead of setting `--swipe-amount` on the container.
- **`will-change` right before the animation, not as a blanket.** Standing `will-change` wastes memory and can hurt.

## Platform-native motion (prefer these in 2026)

Much of what used to need JavaScript and a library now runs natively, off the main thread, and survives interruption for free. Reach for these before reaching for a library; fall back only where browser support forces it.

**View Transitions API.** Animate between two DOM states (or two pages) by snapshotting before and after and tweening the difference, instead of hand-choreographing both ends. For same-document state changes, wrap the mutation:

```js
document.startViewTransition(() => updateTheDOM());
```

Tag the elements that should morph across the change with a shared `view-transition-name`, and the browser animates position, size, and crossfade between them. For multi-page apps, opt into cross-document transitions with no JS at all:

```css
@view-transition { navigation: auto; }
::view-transition-old(root), ::view-transition-new(root) { animation-duration: 250ms; }
```

This is the right tool for shared-element transitions (a card expanding into a detail page, a thumbnail growing into a hero), which were previously a Framer Motion `layoutId` job.

**Scroll-driven animations.** Drive an animation by scroll position or by an element's visibility, declaratively and on the compositor, replacing the `IntersectionObserver` + JS pattern for reveals and progress bars:

```css
/* progress bar tied to document scroll */
.bar { animation: grow linear; animation-timeline: scroll(root block); }

/* reveal tied to the element entering the viewport */
.card {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}
@keyframes reveal { from { opacity: 0; transform: translateY(16px); } }
```

Because it runs off the main thread, it stays smooth during heavy scroll where a JS scroll listener would jank.

**Animate to `height: auto`.** The long-impossible animation now works with `interpolate-size` and `calc-size()`, so accordions and disclosure panels no longer need measuring their scroll height in JS:

```css
:root { interpolate-size: allow-keywords; }
.panel { height: 0; transition: height 250ms var(--ease-out); overflow: clip; }
.panel[data-open] { height: auto; } /* or height: calc-size(auto, size) */
```

**Enter and exit without JS, including `display`.** Pair `@starting-style` (enter) with `transition-behavior: allow-discrete` (exit) so an element can animate as it is added and as it is removed from the layout, with no mount-state bookkeeping:

```css
.toast {
  opacity: 1; transition: opacity 200ms, display 200ms allow-discrete;
  @starting-style { opacity: 0; }
}
.toast[hidden] { opacity: 0; display: none; } /* fades out, then un-displays */
```

**Springs in pure CSS with `linear()`.** The `linear()` timing function approximates spring and bounce curves with a string of sampled points, so a spring feel no longer requires a JS animation when the motion is not interruptible. Generate the points from a spring tool and drop them into a regular `transition`. Keep JS springs for genuinely interruptible, gesture-driven motion; use `linear()` for predetermined springy entrances.

**Animate custom properties with `@property`.** Registering a custom property with a type lets you animate things CSS otherwise treats as discrete, such as a gradient angle or a single color stop:

```css
@property --angle { syntax: "<angle>"; initial-value: 0deg; inherits: false; }
.ring { background: conic-gradient(from var(--angle), …); transition: --angle 400ms linear; }
```

## Verify motion by watching it

Motion is the one thing you cannot review by reading the code. A curve, a duration, and an origin all look fine in a diff and still feel wrong on screen. So the standard's verify loop is non-negotiable here: play the animation, watch it, and only then judge it. "The values look right" is not a stop signal; "I watched it run and it feels right" is.

- **Slow motion.** Temporarily run animations at 2 to 5x duration, or use the DevTools animation inspector. Watch for two states overlapping in a crossfade, abrupt easing, a wrong transform-origin, and properties drifting out of sync.
- **Frame by frame.** Step through in the Chrome DevTools Animations panel to catch timing mismatches between coordinated properties that are invisible at full speed.
- **Real devices for touch.** Test drawers and swipe gestures on physical hardware over USB with remote DevTools, not just a simulator.
- **Fresh eyes the next day.** You notice imperfections after a night that you missed while building. When unsure whether a motion feels right, the strongest move is often to delete it.
