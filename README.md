# Grating Channel Visualizer

An interactive, single-file web visualisation of diffraction at an air / substrate
grating interface. The app shows **real-space rays** and the **2-D k-space Ewald
construction** side by side, with live readouts of every physically meaningful
quantity (k₀, G, kₓ,m, k_y in each medium, propagation condition, mode
classification, …).

> *Grating equation decides which channels exist. Metasurface design decides how
> much power goes into each open channel.*

---

## Quick start

There is **no build step**. The app is a single `index.html` that pulls React,
Tailwind, KaTeX, and Babel from CDNs.

```bash
git clone https://github.com/Wavesflow/grating-channel-visualizer.git
cd grating-channel-visualizer
# any static server will do — pick one:
python3 -m http.server 8765
# or:  npx serve .
# or:  bunx serve .
```

Then open <http://localhost:8765/>.

You can also just open `index.html` directly in a browser via `file://`, but
some browsers throttle CDN scripts on the file protocol; running a local
server is more reliable.

### GitHub Pages

The repository is structured so that enabling GitHub Pages on `main` →
`/ (root)` immediately serves the app at
`https://wavesflow.github.io/grating-channel-visualizer/`.

---

## What it visualises

### Inputs

| symbol | meaning | typical values |
| --- | --- | --- |
| `λ` | wavelength | 0.01 nm (hard X-ray) → 20 µm (mid-IR) |
| `Λ` | grating period | 0.1 nm (atomic crystal) → 50 µm (macroscopic) |
| `λ/Λ` | reciprocal-lattice ratio | 0.0001 → 50 |
| `θᵢ` | incident angle | ±89° |
| `n_air`, `n_sub` | refractive indices | any positive number |
| `m_min`, `m_max`, `m_target` | order range and selected order | integers |

### Physics

Reciprocal-space formulation (the primary equation):

```
k_{x,m} = k_{x,inc} + m G        (momentum translation by reciprocal lattice)
k_{x,inc} = n_in k₀ sin θ_i
G        = 2π / Λ
k₀       = 2π / λ
```

Mode classification per medium:

```
|k_{x,m}| ≤ n k₀  ⇒  propagating  (real k_y)
|k_{x,m}| > n k₀  ⇒  evanescent   (imaginary k_y)
```

Critical (TIR) angle:

```
θ_c = arcsin(n_air / n_sub)
```

### Three live panels

1. **Real-space — interface**: incident ray + transmitted/reflected diffraction
   rays. Transmitted rays in **blue**, reflected in **green**. TIR-guided rays
   keep their role colour but exceed θ_c geometrically. Curved arcs label
   θᵢ and θ_T,0.
2. **k-space — 2-D Ewald construction**: upper semicircle = air
   (radius `n_air k₀`), lower semicircle = substrate (radius `n_sub k₀`).
   For every order m, a vertical phase-matching line at kₓ,m intersects the
   medium circles where the order propagates. Fully evanescent orders show as
   small fine-dashed rings on the kₓ-axis. Click any order anywhere
   (including the table) to set it as the target.
3. **Orders table + reciprocal-space readout panel**: per-order
   kₓ/k₀, θ_T, θ_R, propagation flags, and a target-order block with the
   propagation condition `|k_{x,m}| ≤ n k₀` evaluated explicitly per medium.

### Cross-panel highlighting

Hover any order — in the table, the real-space rays, or the k-space markers —
and the same m lights up in all three views. Click to make it the **target**,
which drives the readout panel.

### Theme & layout

- **Light/dark theme toggle** (header, top-right). The choice persists in
  `localStorage`.
- All math typeset with KaTeX.
- Tailwind via CDN; CSS variables drive the theme.
- Layout is a 12-column grid that collapses on narrower viewports.

---

## File layout

```
grating-channel-visualizer/
├── index.html            ← the entire app (single file, CDN-driven)
├── README.md             ← this file
├── AGENTS.md             ← guide for AI agents that want to modify the code
├── LICENSE               ← MIT
└── .gitignore
```

The single-file design is intentional: no toolchain, no `npm install`, no
build artefacts — clone and open. If you fork this, keep the file structure
flat unless you add a real build pipeline.

---

## Acknowledgements

Built with [React 18](https://react.dev), [Tailwind CSS](https://tailwindcss.com),
[KaTeX](https://katex.org), and [Babel Standalone](https://babeljs.io/docs/babel-standalone).

## License

MIT — see [`LICENSE`](./LICENSE).
