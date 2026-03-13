# Layout Prototype — Sticky Header

A single-page layout prototype with a collapsible sticky header, search prompt, and infinite-scrolling image grid.

**Live:** [GitHub Pages](https://linstack-cmd.github.io/layout-prototype-sticky-header/)

---

## Architecture Notes

### 2-Phase Sticky Transition (v5)

The header collapse uses a **3-state machine** for smoother, more intentional transitions:

| State | Trigger | Visual |
|---|---|---|
| **default** | Page at top | Full header: nav, title, prompt in column layout |
| **presticky** | ~150px before dock threshold | Title fades to 30% opacity and shrinks; nav nudges upward — an "anticipation" phase |
| **docked** | Prompt reaches sticky point | Compact single-row layout, transparent + click-through (same as previous `scrolled`) |

Two `IntersectionObserver` instances watch separate sentinels:
- **prestickySentinel** uses a negative `rootMargin` to fire early (configurable via `--presticky-offset`)
- **dockSentinel** fires at the actual sticky threshold

Scrolling up reverses cleanly: `docked → presticky → default`. Title opacity and nav transforms use CSS transitions (`--transition-speed: 0.25s`); layout-thrashing properties (flex-direction, padding) switch instantly to avoid reflow jank.

### CSS-driven (no JS needed)

| Behavior | How |
|---|---|
| **Header pinning** | `position: sticky; top: 0` on `<header>` — no duplicate DOM element |
| **Title collapse/expand** | `max-height` + `opacity` driven by state classes (`presticky`, `docked`) on `<body>` |
| **Compact single-row layout** | `body.docked .header` switches to `flex-direction: row` |
| **Prompt box narrowing** | `max-width` transition on `.prompt-box` when docked |
| **Textarea auto-resize** | CSS `field-sizing: content` (modern browsers); JS fallback only when unsupported |
| **Card entrance animation** | `@keyframes fadeUp` with staggered `animation-delay` |
| **Responsive grid** | `grid-template-columns: repeat(auto-fill, minmax(180px, 1fr))` — no JS breakpoints |

### JS-only (truly dynamic)

| Behavior | How |
|---|---|
| **State machine** | Two `IntersectionObserver` instances on sentinel elements. `setState()` ensures exactly one class (`presticky` or `docked`) is active at a time. No `scroll` event listener. |
| **Infinite card loading** | `IntersectionObserver` on a sentinel below the grid. Appends 12 cards per trigger with a `loading` guard. Uses `DocumentFragment` for batch DOM insertion. Bounded fill loop for large viewports. |
| **Textarea fallback** | `input` event listener that sets `height = scrollHeight`, only attached when `CSS.supports('field-sizing', 'content')` is false. |

### Key design choices

- **Click-through sticky header** — docked header uses `pointer-events: none`; interactive children re-enable individually.
- **No backdrop/opaque layer** — docked header is fully transparent.
- **Stutter-free mode switch** — no CSS transitions on reflow properties (`flex-direction`, `padding`, `gap`). Only `opacity` and `transform` animate.
- **Anticipation phase** — the presticky state gives visual feedback before the layout shift, making the dock feel intentional rather than abrupt.
