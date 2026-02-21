# Flex Grid CSS

A pure CSS solution for responsive, centered multi-column layouts using Flexbox and Container Queries. No JavaScript. No grid hacks.

## The Problem

CSS Grid's `auto-fit` / `auto-fill` stretches items to fill each row — the last row's items end up wider than the rest. Flexbox with `flex-wrap` lets items wrap naturally, but without explicit width control items either stretch unevenly or collapse.

What we actually want:
- A defined maximum number of columns (e.g. 6)
- Items that respect a minimum width and wrap to fewer columns when space runs out
- **The last row always centered**, with items the same size as in the rows above
- Works inside any container, not just the viewport (nested layouts, sidebars, cards)

## The Idea

Each flex container gets a `--items` variable that controls how many items fit per row. A width formula distributes the available space evenly, accounting for gaps:

```css
width: max(var(--min, 0px), calc(100% / var(--items) - (var(--items) - 1) * var(--gap, 0px) / var(--items)));
```

Combined with `justify-content: center` and `flex-wrap: wrap`, items that overflow wrap to the next row — and the last row is always centered.

The catch: `--items` needs to change at the right breakpoints. Since this must work inside nested containers (not just the viewport), we use **Container Queries** instead of media queries. And because container queries don't support `var()` in their conditions (a fundamental limitation, not a missing feature — it would create circular dependencies in the layout engine), every combination of gap and min-width needs its own query with a precomputed pixel breakpoint.

## Utility Classes

The system uses composable utility classes:

```html
<div class="fg fg-6321 fg-gap20 fg-min200">
  <div>Item 1</div>
  <div>Item 2</div>
  <!-- ... -->
</div>
```

| Class | Purpose |
|---|---|
| `fg` | Base class — flex, wrap, center, container query context |
| `fg-6321` | Column cascade schema: 6 → 3 → 2 → 1 columns |
| `fg-gap20` | Gap between items (20px) |
| `fg-min200` | Minimum item width (200px) |

### Schemas

A schema like `6321` defines the column cascade: start with 6 columns, then break down to 3, then 2, then 1 as the container shrinks. The breakpoints where each transition happens are calculated from `items × min-width + (items - 1) × gap`.

Other examples:
- `42` — 4 columns → 2 columns
- `321` — 3 → 2 → 1
- `8421` — 8 → 4 → 2 → 1

## Generator

The [generator page](https://mike-tuxedo.github.io/flexgrid-css/generator.html) outputs the complete CSS for your configuration. You can define:

- **Multiple schemas** at once (e.g. `6321,42,31`) — each gets its own set of container queries with auto-generated class names
- **Gap variants** — any pixel values you need
- **Min-width variants** — any pixel values you need
- **Prefix** — change `fg` to whatever fits your project

The generator shows raw size, minified size and estimated gzip size, so you can gauge the overhead before committing to a configuration.

## Why Not Grid?

`grid-template-columns: repeat(auto-fit, minmax(200px, 1fr))` handles the wrapping part well, but items in the last row stretch to fill the available grid tracks. There is no pure CSS way to center a partial last row in CSS Grid while keeping items the same width as in full rows.

## Browser Support

Requires support for Container Queries (`container-type: inline-size`) and CSS `max()`. Supported in all modern browsers since 2023.

## Links

- [Live Demo](https://mike-tuxedo.github.io/flexgrid-css/)
- [Generator](https://mike-tuxedo.github.io/flexgrid-css/generator.html)
