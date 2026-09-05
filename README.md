# Modal dispersion of LED-class sources over multicore imaging fiber
### An interactive physics module from my micro-LED link simulator

*Single-file HTML/JS, no dependencies, runs offline. Part of a larger end-to-end
link simulator I built to understand wide-and-slow micro-LED optical interconnects
(the architecture class published by Microsoft's MOSAIC, SIGCOMM 2025, and others).
This module is the fiber-propagation core; the receiver-architecture and
equalization portions of the full tool are not included in this release.*

---

## What it models

A micro-LED is a Lambertian, broadband (Δλ > 10 nm) emitter launched into a
step- or graded-index multimode core. This module computes, from first
principles and interactively:

- **Guided-mode structure** — V-number, guided LP mode count (one and two
  polarizations), and per-mode effective index n_eff for a user-set core
  diameter, NA, and refractive-index profile exponent α (α = 2 graded → α = 99
  step).
- **Launch-weighted modal impulse response** — each ray/mode group is weighted
  by its solid-angle share (sin θ·cos θ) times a Gaussian launch distribution
  set by the far-field half-angle at 50% power, so *underfilled vs. overfilled
  launch is a slider, not an assumption*. The resulting h(t) kernel is what the
  rest of the simulator convolves — the displayed "spread" and "rms" numbers
  are descriptors, not the model.
- **Spread per unit interval** — modal delay spread expressed in UI at the
  selected baud, which is the quantity that actually decides link feasibility.
- **ISI taps at the decision point** — the sampled pulse response of a 1-UI
  rectangular pulse through the delay kernel, following the standard
  pulse-response convention (the sampling phase is scanned across one UI and
  the cursor is placed where the main tap is largest, i.e. the eye centre).
  The chart reads out pre-cursor, post-cursor, and beyond-±3-UI energy
  separately, so nothing is silently dropped.

## Key results (all reproducible in the tool)

1. **For LED sources, fiber NA — not attenuation, not chromatic dispersion — is
   the reach wall at multi-Gb/s.** A fully excited NA = 0.2 step-index core has
   a modal walk-off of NA²/(2·n·c) ≈ **43.9 ps/m**. The spread reaches one full
   UI at ~11 m at 2 Gb/s and ~6 m at 4 Gb/s — before equalization, that is the
   unaided eye-closure point. By comparison, chromatic walk-off for a 20 nm-wide
   blue LED (D ≈ 500 ps/nm·km) is ~10 ps/m: same order, but smaller — and it is
   Gaussian-soft, while the modal kernel is a hard rectangular wall.

2. **NA is a quadratic lever.** Dropping launch NA 0.2 → 0.15 → 0.125 cuts the
   walk-off 43.9 → 24.7 → 17.1 ps/m, moving the 1-UI point at 2 Gb/s from 11 m
   to 20 m to 29 m. This is why launch conditioning (collimation, underfilled
   launch) is the cheapest reach knob in this architecture — consistent with
   the launch-optimization direction reported publicly by MOSAIC (§3.4).

3. **The collection–dispersion trade-off.** The same NA that collects more
   Lambertian light (coupling ∝ NA²) also creates the modal spread (walk-off ∝
   NA²). Collection and ISI are bought with the same currency, so there is an
   optimal operating ridge in the (NA, reach) plane rather than a "more light
   is better" monotone. The module makes this ridge visible interactively.

4. **Launch conditioning reshapes the *structure* of the ISI, not just its
   size.** The tap chart shows that an overfilled launch on a step-index core
   produces a flat-top kernel whose ISI lands on both sides of the cursor,
   while an underfilled launch collapses the pre-cursor side and leaves only a
   small post-cursor tail — the two channels have very different character even
   at similar rms spread.

5. **Graded index buys a large but launch-sensitive factor.** The α-profile
   slider shows the classic result — an ideal α ≈ 2 profile collapses the modal
   spread by more than an order of magnitude vs. step — and also its fragility:
   the benefit shrinks quickly as the launch overfills or α drifts.

## Validation anchors (public sources only)

- Step-index walk-off reproduces the textbook NA²/2nc form exactly (43.9 ps/m
  at NA 0.2, n = 1.52).
- The commercial blue-wavelength multicore fiber indices published in Song et
  al. (SPIE, FDTD study of micro-LED chip-to-fiber coupling), n = 1.465/1.449,
  give NA = 0.216 — independently consistent with the ~0.2 launch-NA regime
  this module defaults to.
- The full simulator (of which this is one module) reproduces the pass/fail
  verdict of every point in Fig. 12 of the MOSAIC paper (Benyahya et al.,
  SIGCOMM 2025) — 2 Gb/s passing FEC at 20 m and failing at 30 m, 1.6 Gb/s
  passing at 30 m, error-free floors at 10–20 m — with one documented residual
  (the model's BER slope beyond ~25 m is steeper than their measured curve).

## Honest limits

- Ray/mode-group treatment with a launch-weighted kernel — not a full vectorial
  mode solver; mode coupling along the fiber is not modeled (short-reach,
  few-tens-of-meters regime, where it is second-order).
- The kernel is deterministic per configuration; polarization and per-core
  manufacturing variation are not modeled.
- Numbers above are for the fiber physics in isolation; a real link budget adds
  source bandwidth, receiver bandwidth and noise, which the full tool handles.

## Why this exists

I wanted a defensible, physics-first answer to one question: *what actually
limits the reach of an LED-based wide-and-slow link, and which knob moves it?*
The answer this module gives — modal walk-off, quadratic in NA, traded directly
against collection — shaped every downstream architecture conclusion in the
larger study.
