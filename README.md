# Virtual Wind Tunnel — Potential Flow Explorer

> An educational potential flow visualizer built on classical aerodynamics theory.
> Uses stream functions, velocity potentials, and the Cauchy-Riemann equations to
> render live streamlines, equipotential lines, and aerodynamic metrics in the browser.

---

## What This Is

This is a single-file browser tool that lets you input a flow configuration and instantly
see the resulting flow field — streamlines (ψ), equipotential lines (φ), velocity vectors,
and pressure distribution. It also computes circulation (Γ) via numerical line integral and
predicts lift per unit span using the Kutta-Joukowski theorem.

**This is not a CFD solver.** Potential flow assumes inviscid, irrotational, incompressible
flow — no separation, no boundary layers, no shocks. That's a feature, not a bug: it means
every result is analytically traceable and you can verify the output by hand.

---

## Theory

### Stream Function ψ and Velocity Potential φ

For a 2D incompressible, irrotational flow, every flow field can be described by two scalar
functions:

- **Stream function ψ(x, y):** Lines of constant ψ are streamlines. Fluid particles follow these.
- **Velocity potential φ(x, y):** Lines of constant φ are equipotential lines. These are always perpendicular to streamlines.

The velocity components are recovered from either function:

```
u = ∂φ/∂x = ∂ψ/∂y
v = ∂φ/∂y = −∂ψ/∂x
```

### Cauchy-Riemann Equations

The perpendicularity of streamlines and equipotentials follows directly from the Cauchy-Riemann equations:

```
∂φ/∂x = ∂ψ/∂y    →    u component
∂φ/∂y = −∂ψ/∂x   →    v component
```

Both ψ and φ individually satisfy Laplace's equation:

```
∇²ψ = 0    ∇²φ = 0
```

Together, w(z) = φ + iψ forms the **complex potential**, the foundation of conformal mapping
used historically to analyze real airfoil shapes.

### Circulation and Lift

Circulation is computed via a closed line integral around a contour enclosing the body:

```
Γ = ∮ V · ds
```

Lift per unit span follows from the **Kutta-Joukowski theorem**:

```
L' = ρ U∞ Γ
```

where ρ = 1.225 kg/m³ (sea-level standard) and U∞ is the freestream velocity.

---

## Flow Types

| Preset | Stream Function ψ(x, y) | Notes |
|---|---|---|
| Uniform Flow | U·y | Parallel horizontal streamlines |
| Source / Sink | (m/2π)·arctan(y/x) | Radial streamlines; m < 0 = sink |
| Doublet | −(κ/2π)·y/(x²+y²) | Limit of source-sink pair |
| Free Vortex | −(Γ/4π)·ln(x²+y²) | Concentric circular streamlines |
| Rankine Oval | U·y + (m/2π)·[arctan(y/(x+1)) − arctan(y/(x−1))] | Source + sink + uniform |
| Cylinder + Vortex | U·(1−a²/r²)·y − (Γ/4π)·ln(r²) | Generates lift when Γ ≠ 0 |
| Flow Past Cylinder | U·(1−a²/r²)·y | Γ = 0 case; stagnation at (±a, 0) |
| Custom ψ(x, y) | User-defined expression | Evaluated via JavaScript; numerical gradients for u, v |

---

## Controls

### Parameters

| Slider | Symbol | Description |
|---|---|---|
| Free-Stream Velocity | U∞ | Freestream speed in m/s |
| Source Strength | m | Positive = source, negative = sink |
| Doublet Strength | κ | Strength of source-sink pair in the limit |
| Circulation | Γ | Positive = counterclockwise; drives lift |
| Body Radius | a | Radius of cylinder body |
| Streamline Count | — | Number of ψ-contour levels drawn |
| Resolution | N | Grid size for numerical computation (N×N) |

### Display Toggles

- **ψ STREAMLINES** — contours of constant stream function (green)
- **φ EQUIPOTENTIAL** — contours of constant velocity potential (cyan dashed)
- **∇φ VELOCITY** — velocity vector field (amber arrows)
- **Show Pressure Colormap** — Cp = 1 − (V/V∞)² mapped to blue→green→red
- **Mark Stagnation Points** — detects grid cells where |V| → 0, marks with red crosshair
- **Show Body Outline** — draws the cylinder boundary for relevant presets

### Custom ψ Mode

Type any expression of `x`, `y`, `U`, `m`, `Gamma`, `a` in the text field. Standard
JavaScript math applies. `PI` is available.

Examples:

```
U*y + m/(2*PI)*Math.atan2(y, x)          // uniform + source
U*y - Gamma/(4*PI)*Math.log(x*x+y*y)    // uniform + vortex
Math.sin(x)*Math.cos(y)                  // saddle-point field
```

Velocity components are derived numerically via central differencing (h = 0.001).

---

## Verification Checklist

Use these known results to confirm correctness:

| Test | Expected Result |
|---|---|
| Uniform Flow | Streamlines are horizontal; ψ = U·y is linear in y |
| Source at origin | Streamlines radiate outward symmetrically |
| Cylinder, Γ = 0 | Stagnation points at exactly (±a, 0) |
| Cylinder, Γ ≠ 0 | Stagnation points move to θ = arcsin(−Γ / 4πU∞a) |
| Kutta-Joukowski | Γ = 2π, ρ = 1.225, U∞ = 1 → L' = 7.696 N/m |
| Cauchy-Riemann | Streamlines (green) and equipotentials (cyan) are perpendicular everywhere |

**Reference:** Anderson, J.D. — *Fundamentals of Aerodynamics*, Chapter 3 (Table 3.1 for all ψ formulas).

---

## Numerical Methods

| Task | Method |
|---|---|
| Contour drawing | Marching squares on N×N grid |
| Circulation | Trapezoidal line integral, 500-point circular path |
| Custom flow gradients | Central finite difference, h = 0.001 |
| Stagnation detection | Grid scan; threshold |V| < 0.05·U∞, deduplication by proximity |
| Pressure coefficient | Cp = 1 − V²/V∞² at each grid cell |

---

## Limitations

Potential flow theory makes several assumptions that break down in real situations:

- **No viscosity** — no boundary layer, no skin friction drag
- **No separation** — d'Alembert's paradox means zero pressure drag on a cylinder (real cylinders separate)
- **Incompressible** — invalid above Mach ≈ 0.3
- **Irrotational** — vorticity is zero everywhere except at singular points

These are known, understood limitations. The tool is accurate within its assumptions.
When results look physically wrong (e.g., zero drag on a bluff body), that is correct
potential flow behavior — not a bug.

---

## File Structure

```
virtual_wind_tunnel.html    ← entire application; self-contained, no dependencies
README.md                   ← this file
```

---

## Usage

Open `virtual_wind_tunnel.html` in any modern browser. No server, no install, no build step.

```bash
# Option 1: double-click the file in your file manager

# Option 2: serve locally
python -m http.server 8000
# then open http://localhost:8000/virtual_wind_tunnel.html
```

---

## Academic Context

This tool corresponds to the following undergraduate aerodynamics topics:

- **2D Potential Flow Theory** — AE/ME fluid mechanics, Year 2–3
- **Complex Potential and Conformal Mapping** — basis for Joukowski airfoil analysis
- **Kutta-Joukowski Theorem** — the theoretical origin of lift before Navier-Stokes solvers
- **Preliminary Aerodynamic Design** — classical methods used before CFD (panel methods, thin airfoil theory)

---

*Built as an educational tool to bridge the gap between potential flow mathematics and physical intuition.*
*Not intended for engineering design calculations.*
