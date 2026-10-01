
<meta
  name="description"
  content="Learn how to build lightweight data visualizations with ProvChart, a CSS-based charting system."
/>

<meta
  name="keywords"
  content="ProvChart, CSS charts, pure CSS charts, FSCSS, data visualization"
/>
# ProvChart

A pure-CSS charting system for the modern web. Build dashboards, stat cards, and data visualizations with zero JavaScript runtime — just HTML + CSS.

## What is ProvChart?

ProvChart has two parts:

| Component | Description |
|-----------|-------------|
| **st-core.fscss** | Open-source CSS charting library ([GitHub](https://github.com/fscss-ttr/st-core.fscss)) |
| **ProvChart API** | Hosted service for generating charts from data ([chart.devtem.org](https://chart.devtem.org)) |

Both render charts using **CSS `clip-path: polygon()`** and **CSS custom properties** — no Canvas, no SVG, no charting library shipped to the browser.

## Why ProvChart?

- **Zero runtime weight** — Charts are pure CSS. No chart.js, no D3, no heavy JS bundle.
- **First-paint rendering** — Charts appear instantly, no JavaScript execution required.
- **SEO-friendly** — Crawlers see your chart content immediately.
- **Simple updates** — Change CSS variables, browser repaints natively.
- **Backend-agnostic** — Use any language (Python, Node, Go) to generate chart data.

## Architecture

```
Data source (any backend)
      ↓
Compile to CSS custom properties (--st-p1 through --st-p8)
      ↓
Browser renders clip-path: polygon()
      ↓
Chart displayed via pure CSS
```

## Quick Start st-core.fscss 

A pure-CSS chart engine for pages that can't afford a runtime.

Data becomes CSS custom properties. The browser paints the geometry natively (`clip-path`, bar heights, gauge rings). After generation there's no chart-library JS on the page.

- **Site:** [chart.devtem.org](https://chart.devtem.org)
- **Docs:** [chart.devtem.org/docs](https://chart.devtem.org/docs)
- **Live demo:** [chart.devtem.org/demo](https://chart.devtem.org/demo)
- **Open-source core:** [st-core.fscss](https://github.com/fscss-ttr/st-core.fscss)
- **Optional runtime:** [provchart-runtime](https://github.com/fscss-ttr/provchart-runtime) ([npm](https://www.npmjs.com/package/provchart-runtime))

## Who this is for

- Teams that care about first paint, SEO, and Core Web Vitals
- Static sites and docs that need charts without client chart JS
- Products that generate charts from backends or agents via API

## Quick start

**Option A — Free visual builder.** Open [chart.devtem.org/get-started](https://chart.devtem.org/get-started), enter one series, copy the HTML + CSS. No account required.

**Option B — Developer API.**

```javascript
const res = await fetch("https://provchart-api.devtem.org/api/v1/generate", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-API-Key": "YOUR_API_KEY"
  },
  body: JSON.stringify({
    type: "line",
    series: [
      { name: "Revenue", color: "#8b7bff", points: [20, 35, 45, 30, 50] }
    ],
    axisX: ["Jan", "Feb", "Mar", "Apr", "May"]
  })
});
const data = await res.json();
if (data.success) {
  document.getElementById("chart").innerHTML = data.html;
  document.head.insertAdjacentHTML("beforeend", `<style>${data.css}</style>`);
}
```

Replace `YOUR_API_KEY` with a key from Dashboard → Developer API. Free accounts get limited test generations; Pro/Business raise monthly limits.

Try it live without writing any code: [chart.devtem.org/demo](https://chart.devtem.org/demo) — paste a key and run requests against `/generate` and `/generate-svg` directly in the browser, with the payload for each chart visible.

## Core concepts

1. **Compile** — points become CSS variables on a chart root.
2. **Paint** — `clip-path`, bar heights, or gauge rings draw the shape.
3. **Index** — HTML/CSS ship in the first response, so it's SEO-friendly.
4. **Update** — rewrite the generated CSS variables directly, or re-call generate with new points.

Values are typically on a **0–100** scale (100 ≈ top of the plot). Normalize upstream if your data uses other units.

Point count follows the `series[].points` array you send, not a fixed slot count — practical limits are HTTP payload size and what `clip-path` geometry stays workable in the browser, rather than a hardcoded cap. Generated chart IDs and CSS variable names (e.g. `pc-chart-3-4efv`) are scoped per chart instance so multiple charts on one page don't collide.

## Chart types

| Function | Plan |
|---|---|
| `line()` | Free / Pro |
| `area()` | Pro |
| `bar()` | Free / Pro |
| `hbar()` | Pro |
| stacked bar | Pro |
| `scatter()` | Pro |
| `combo()` | Pro |
| `gauge()` | Free / Pro |
| `stat()` | — |
| `combine()` | — |
| `update()` | — |

See the [full API reference](https://chart.devtem.org/docs#api-line) for payload shapes per type.

## Developer API

Base URL: `https://provchart-api.devtem.org/api/v1`

- `POST /generate` → returns `html` + `css`
- `POST /generate-svg` → returns `svg` + `dataUri` (same body, plus optional `width`/`height`)
- `GET /usage` → plan, used, limit, remaining

Both generate endpoints share the same monthly quota. Create and manage keys under Dashboard → Developer API. Never ship a live key in a public client bundle — proxy through your server or CI.

```
X-API-Key: pc_live_xxxxxxxx
// or
Authorization: Bearer pc_live_xxxxxxxx
```

### Error codes

| HTTP | code | Meaning |
|---|---|---|
| 401 | `INVALID_API_KEY` | Bad, missing, or revoked key |
| 403 | `SUBSCRIPTION_REQUIRED` | Plan inactive |
| 429 | `MONTHLY_LIMIT_REACHED` | Monthly quota used up |
| 400 | — | Invalid body, type, or empty series |
| 500 | `INTERNAL_ERROR` | Server error — retry, then contact support |

Full docs: [chart.devtem.org/docs#dev-api-overview](https://chart.devtem.org/docs#dev-api-overview)

## Examples

Runnable integration examples for the Developer API:

| File | Language |
|---|---|
| [`examples/node-example.js`](examples/node-example.js) | Node.js |
| [`examples/python-example.py`](examples/python-example.py) | Python |
| [`examples/curl-example.sh`](examples/curl-example.sh) | curl |
| [`examples/html-css/provchart-chart.html`](examples/html-css/provchart-chart.html) | html |
Each covers `/generate` for HTML/CSS output, with `/generate-svg` shown as a variant for README and docs embeds where a `<style>` block isn't available.

## Optional runtime

`provchart-runtime` is an additive client script — it enhances the static HTML/CSS ProvChart already generated, and never changes what the core engine outputs. It scans for `[data-provchart]` roots and adds:

- scroll-reveal / enter animation
- hover scale
- legend-driven series highlighting
- per-point tooltips, read from the generated `--pc-{id}-s{series}-p{index}` variables
- count-up on gauge/stat values
- a `MutationObserver` for charts injected after load (SPA routes, `fetch`, hydration)

It skips pure SVG output from `/generate-svg` by default.

```html
<script>
  window.ProvChartRuntimeConfig = { tooltips: true, perPointTooltips: true };
</script>
<script src="https://cdn.jsdelivr.net/npm/provchart-runtime@1.0.0/dist/provchart-runtime.min.js" defer></script>
```

Repo: [github.com/fscss-ttr/provchart-runtime](https://github.com/fscss-ttr/provchart-runtime) · npm: [provchart-runtime](https://www.npmjs.com/package/provchart-runtime)

## Framework integration

Server or build process holds the API key, calls ProvChart, and returns `html`/`css` (or `svg`) to the client — or writes the output to static files at build time.

- **Next.js** — call from a route handler, proxy the key server-side
- **Astro / other SSGs** — call in a build script or server endpoint, inject the returned HTML/CSS into the page partial
- **Node / Python** — see `examples/`
- **README + docs** — use `/generate-svg` and embed the returned `dataUri`

Full integration patterns: [chart.devtem.org/docs#integration-stacks](https://chart.devtem.org/docs#integration-stacks)

## Theming

- Named presets: `"dark"` (default), `"light"`, `"midnight"`
- Or pass an object: `bg`, `surface`, `muted`, `text`, `grid`, `radius`

## Open source

The rendering ideas and CSS engine this is built on are open source: [st-core.fscss](https://github.com/fscss-ttr/st-core.fscss) (MIT). It's the right tool on its own for hand-built, fixed-shape dashboard cards where you're authoring the markup directly. ProvChart applies the same paint model — data in custom properties, geometry via `clip-path` — to arbitrary series lengths, multi-series layouts, and server-side compilation through the Developer API.

## Support

- Docs: [chart.devtem.org/docs](https://chart.devtem.org/docs)
- Guides: [chart.devtem.org/guides](https://chart.devtem.org/guides)
- Changelog: [chart.devtem.org/changelog](https://chart.devtem.org/changelog)
- Pricing: [chart.devtem.org/pricing](https://chart.devtem.org/pricing)


---


## Documentation

| Guide | Description |
|-------|-------------|
| [Introduction](docs/01-introduction.md) | What ProvChart is and how it works |
| [Installation](docs/02-installation.md) | Setup options: CDN, CLI, npm |
| [Getting Started](docs/03-getting-started.md) | Your first chart in 5 minutes |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT License. See [LICENSE](LICENSE).

## Credits

- **st-core.fscss** by [fscss-ttr](https://github.com/fscss-ttr/st-core.fscss) — MIT License
- **FSCSS** by [figsh](https://fscss.devtem.org) — CSS preprocessor
- **ProvChart API** by [DevTemple](https://chart.devtem.org) — Hosted chart generation
