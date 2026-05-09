# AGENTS.md — instructions for AI coding agents

This file tells AI agents (Claude Code, Cursor, Codex, Aider, etc.) how to
work on this repository safely. Humans: see `README.md`.

---

## Project shape

- **Single-file SPA**: all source, styling, and React/Babel/Tailwind/KaTeX
  imports live in `index.html`.
- **No build step.** Babel-standalone transpiles JSX in the browser at load
  time. Tailwind is the JIT CDN build. KaTeX renders via the global
  `window.katex` injected by its CDN script.
- **No package.json, no node_modules, no bundler.** Don't add one unless the
  user explicitly asks; the zero-toolchain story is a feature.

---

## How to run / preview

```bash
# from the repo root
python3 -m http.server 8765
# then open  http://localhost:8765/
```

Alternatives that also work: `npx serve .`, `bunx serve .`, `caddy file-server
--listen :8765`. Opening `index.html` via `file://` works for most browsers
but sometimes blocks CDN scripts or `localStorage`; prefer a local server.

There is no test suite. Verification is visual: open the page and exercise
every slider / toggle / hover / click.

---

## Code map (where things live in `index.html`)

The file is laid out top-to-bottom as:

1. `<head>` — CDN imports for React 18, Tailwind, Babel-standalone, KaTeX.
2. `<style>` — CSS variables for light/dark themes (`html[data-theme="..."]`),
   plus all hand-written component styles. Tailwind utility classes are also
   used inline in JSX.
3. `<script type="text/babel">` — the entire React app. Sub-sections, in
   order:
   - `TeX`         — KaTeX wrapper component (uses `useEffect` + `useRef`).
   - `fmt`, `shadeOpacity` — small helpers.
   - `ArcLabel`, `LabelTeX` — SVG label helpers (foreignObject + KaTeX).
   - `RealSpaceView` — the upper-left plot.
   - `KSpaceView`   — the lower-left 2-D Ewald construction.
   - `Slider`, `NumericInput` — input controls. `NumericInput` is buffered:
     it commits on blur / Enter, accepts both `.` and `,` decimal separators.
   - `Controls`     — the right-column input card.
   - `FormulasPanel`, `KReadout` — equations and live readout cards.
   - `OrdersTable`  — the per-order table.
   - `App`          — top-level component, owns all state, lays out the
     12-column grid, handles theme + cross-panel hover linking.

The state owned by `App` includes `lambda`, `period`, `thetaI`, `side`,
`mMin`, `mMax`, `mTarget`, `nAir`, `nGlass`, `theme`, `hoveredM`, `kZoom`.
`derived` is a `useMemo` that recomputes the orders array and a few cached
quantities (`kxIn0`, `ratio`, `thetaC`, `nIn`, `nOut`).

---

## Conventions to preserve

### KaTeX in JSX

JSX **string attributes** do not process backslash escapes — `tex="\\lambda"`
passes the literal `\\lambda` to KaTeX, which renders the line-break command
`\\` followed by italic `lambda`. **Always use the expression form**:

```jsx
<TeX tex={"\\lambda"} />
<Slider latex={"\\theta_i"} ... />
```

Inside template literals, use the regular double-backslash form
(`` `\\theta = ${value}` ``).

For SVG labels with math (axis labels, ray labels, angle arcs, m labels),
render KaTeX inside `<foreignObject>` with `xmlns="http://www.w3.org/1999/xhtml"`
on the inner div, e.g.

```jsx
<foreignObject x={...} y={...} width={...} height={...} style={{overflow:'visible'}}>
  <div xmlns="http://www.w3.org/1999/xhtml" style={{...}}>
    <TeX tex={"k_x / k_0"} />
  </div>
</foreignObject>
```

### Theming

Every colour goes through a CSS variable defined in `:root` /
`html[data-theme="dark"]` and overridden in `html[data-theme="light"]`.
**Never hard-code hex colours in JSX/SVG** unless you also add a matching
variable. The theme toggle just flips `data-theme` on `<html>`.

### Color semantics

- **Transmitted** waves → blue (`var(--trans)`).
- **Reflected** waves  → green (`var(--refl)`).
- **TIR-guided** waves keep their role colour (no special amber). The
  geometry — substrate-circle position outside the air circle, plus a small
  air-side dashed ring on the kₓ-axis — conveys the trapping.
- **Evanescent** orders that exist in *neither* medium → small fine-dashed
  ring (`stroke="var(--text)"`, `dash="1.5 1.5"`) on the kₓ-axis.
- **Air** half-circle = `var(--accent)` (blue), **substrate** = `var(--accent-2)`
  (purple). These colour the *circles*, not the arrows.

### k-space geometry

The two semicircles use SVG `<circle>` clipped by a `<clipPath>` — *not*
SVG arc paths — to avoid sweep-flag ambiguity. Air is the upper half
(`clipPath="url(#upper-half)"`), substrate is the lower half. Substrate
**must** be visibly larger than air (its radius is `n_sub × scale > n_air ×
scale`).

### Buffered numeric input

`NumericInput` is `<input type="text" inputMode="decimal">`, never
`type="number"`, because `type="number"` honours the browser's locale and
displays `7,770` for some users. Accept both `.` and `,` on commit; commit
only on blur or Enter; restore the previous value on invalid input or
Escape.

### Cross-panel hover & click

Hover state lives on `App` as `hoveredM` and is passed to
`RealSpaceView`, `KSpaceView`, and `OrdersTable`. Each emits
`onHover(m | null)` and `onSelectTarget(m)`. Don't fragment this state.

---

## Things to avoid

- Adding a build system (Vite, webpack, Next.js…) without explicit user
  request. The single-file ergonomics are the whole point.
- Replacing CDN scripts with bundled copies. Same reason.
- Writing tests, READMEs, or docs files unless asked. (This file is the one
  exception.)
- Hard-coding colours, font sizes, or spacings — use CSS variables and the
  Tailwind scale.
- Touching the physics. The math is correct. If you want to refactor, keep
  the kₓ,m / k_y / θ formulas verbatim and only rename or restructure.

---

## How to make a change

1. `git pull`.
2. Edit `index.html`. Use precise edits — do not rewrite the file unless the
   change spans the entire structure.
3. Open `http://localhost:8765/` (start the server first if needed) and
   exercise the affected control / view.
4. Verify both **light and dark themes** — toggle the sun/moon button.
5. Verify the change works at the **extremes** of the parameter ranges: hard
   X-ray wavelength (λ = 0.154 nm), atomic-crystal period (Λ = 0.5 nm),
   negative incident angle, m range with very few or many orders, both
   incident sides (`air → sub` and `sub → air`).
6. Commit with a focused message; push.

---

## Common quick recipes

### Add a new readout row

In `KReadout`, append to the relevant grid:

```jsx
<span className="text-dim"><TeX tex={"\\text{your symbol}"} /></span>
<span className="mono">{fmt(value, 4)}</span>
```

### Add a new chip in the header

```jsx
<span className="chip"><TeX tex={"..."} /> {value}</span>
```

### Persist a new setting

Add a `useState`, then mirror it into / out of `localStorage` inside a
`useEffect` keyed on the state, like the `theme` already does.

---

## Contact

Maintainer: <https://github.com/Wavesflow>. Open issues / PRs on GitHub.
