# Layout Prototype — Sticky Header

A single-page layout prototype with a collapsible sticky header, search prompt, and infinite-scrolling image grid.

**Live:** [GitHub Pages](https://linstack-cmd.github.io/layout-prototype-sticky-header/)

---

## Architecture Notes

### CSS-driven (no JS needed)

| Behavior | How |
|---|---|
| **Header pinning** | `position: sticky; top: 0` on `<header>` — no duplicate DOM element |
| **Title collapse/expand** | `max-height` + `opacity` transition triggered by `.scrolled` class on `<body>` |
| **Backdrop blur on scroll** | `backdrop-filter: blur()` + semi-transparent background, applied via `.scrolled` |
| **Compact single-row layout** | `body.scrolled .header` switches to `flex-direction: row`, placing nav controls and prompt box on the same horizontal line with matched 36px heights |
| **Prompt box narrowing** | `max-width` transition on `.prompt-box` when scrolled |
| **Textarea auto-resize** | CSS `field-sizing: content` (modern browsers); JS fallback only when unsupported |
| **Card entrance animation** | `@keyframes fadeUp` with staggered `animation-delay` |
| **Responsive grid** | `grid-template-columns: repeat(auto-fill, minmax(180px, 1fr))` — no JS breakpoints |

### JS-only (truly dynamic)

| Behavior | How |
|---|---|
| **Scroll class toggle** | `IntersectionObserver` on a sentinel `<div>` above the header. Adds/removes `body.scrolled`. No `scroll` event listener. |
| **Infinite card loading** | `IntersectionObserver` on a sentinel below the grid. Appends 12 cards per trigger with a `loading` guard to prevent duplicates. Uses `DocumentFragment` for batch DOM insertion. Uncapped — loads indefinitely. After each batch, the sentinel is unobserved/reobserved to reset intersection state, plus a bounded fill loop ensures enough content loads to make the page scrollable on very large viewports (capped at 10 extra batches per cycle). |
| **Textarea fallback** | `input` event listener that sets `height = scrollHeight`, only attached when `CSS.supports('field-sizing', 'content')` is false. |

### What was removed from the original

- **Duplicate sticky bar HTML** — replaced by a single `<header>` that transforms via CSS.
- **`scroll` event listener** — replaced by `IntersectionObserver` (passive, no jank).
- **Capped infinite scroll** — now truly unlimited with a loading guard.
