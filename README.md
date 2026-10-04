# End-to-end link model for wide-and-slow micro-LED optical interconnects
### An interactive link simulator: LED → fiber → receiver → BER

*Single-file HTML/JS, no dependencies, runs offline. This is the end-to-end link
tab of a larger simulator I built to understand wide-and-slow micro-LED optical
interconnects (the architecture class published by Microsoft's MOSAIC, SIGCOMM
2025, and others). It carries the full signal chain and the link budget; the
coupling, crosstalk, etendue and receiver-circuit modules of the full tool are
not part of this release.*

---

## What it models

The chain is built in the time domain, sample by sample, and every block is
visible as a slider:

- **Transmitter** — NRZ at 1–12 Gbaud, optional analog pre-emphasis. Pre-emphasis
  is charged against extinction ratio at a configurable cost (dB ER per dB of
  peaking), because on a direct-drive LED the real price of peaking is swing, not
  noise.
- **LED response** — causal asymmetric IIR: rise is fast, fall is slow, plus a
  25% tail with a fixed time constant. This is a phenomenological model fitted to
  measured eyes and BER, not a device-physics model; the LED −3 dB bandwidth is
  itself a function of current density through a published S21-vs-J table.
- **Chromatic dispersion** — Gaussian low-pass, σ = D·L·Δλ/2.355. For an
  incoherent source CD is a well-behaved low-pass, not an interference comb; that
  distinction is what makes it equalizable here at all.
- **Modal dispersion** — launch-weighted kernel from the bundle NA, the profile
  exponent α and the far-field half-angle at 50% power, so underfilled versus
  overfilled launch is a slider rather than an assumption. The kernel is centred
  on its own centroid, so it contributes ISI on **both** sides of the cursor.
- **Receiver** — Gaussian RX filter, input-referred noise with a shaped PSD,
  transmit pattern SNR, and an optional **1-tap analog DFE** (tap fitted from the
  conditional means of the running signal, 0.15 UI settling, centre-sampled,
  driven by actual decisions).
- **Optical budget** — received power either computed from OMA, coupling,
  Fresnel and fiber attenuation terms, or pinned manually.

## What it shows

- **Eye diagram** rendered as an additive trace-density heat map, with display
  noise added per sample, so received power and ER visibly change eye thickness.
- **Electrical response** — the CD and modal low-pass curves and the analog
  peaking response on one axis, so you can see what the equalizer is and is not
  inverting.
- **BER versus reach**, three curves: no equalization, analog peaking, and
  peaking plus the 1-tap DFE, with the DFE tap refitted at every reach point.
- **Metric strip** — BER at the selected reach before and after equalization,
  total ISI in UI and in ps rms, the ISI split between LED, modal and CD, the
  effective ER after the peaking cost, and the received power with its budget
  line items.

## Key results (all reproducible in the tool)

1. **The ISI budget is dominated by the LED at short reach and by the fiber
   beyond it.** At the default 5 m, 4 Gbaud, mode-controlled setting the split
   reads LED 64%, modal 18%, CD 1%. Raise NA or reach and the modal term takes
   over. CD stays small across this whole regime — for a blue LED the modal wall
   arrives first.

2. **Half of the fiber ISI cannot be equalized by a DFE, and the tool says so.**
   The LED kernel is causal, so it produces post-cursor only. The CD and modal
   kernels are symmetric, so they produce pre-cursor and post-cursor in roughly
   equal measure. Measured on this build: with mode control the cursors are 1.3%
   pre and 4.7% post of swing; at NA 0.15 over 20 m they are 25.2% and 27.3%.
   The 1-tap DFE takes the post-cursor to 0.1% and 0.5% respectively and leaves
   the pre-cursor untouched — which is the honest ceiling of a decision-feedback
   structure on a symmetric channel.

3. **The DFE converts into optical budget, not just into metres.** Because it
   buys tolerance to delay spread, it lets the launch NA stay higher than the
   unaided optimum, and launch NA is what sets collection. The reach curves make
   that trade visible.

4. **Peaking is the wrong shape for the fiber.** Analog peaking inverts the LED's
   minimum-phase roll-off and helps there; it cannot fill the dips of a
   multi-UI symmetric kernel, and it still costs ER. On modal-limited settings
   the peaking curve and the no-equalization curve nearly coincide while the DFE
   curve separates.

## Validation anchors (public sources only)

- **Three presets ship with the tool**: the Avicena eKit 1 Tbps demo (3 Gb/s,
  5 m), the MOSAIC imaging-fiber point (2 Gb/s, 20 m) and the HotI 4 Gb/s over
  20 m OM3 point. On this build they run to 2.5e-12, 7.4e-8 and 1.2e-7
  respectively — the MOSAIC point sits inside the pass band reported in the
  paper.
- The full Fig. 12 sweep of the MOSAIC paper (Benyahya et al., SIGCOMM 2025) —
  pass and fail verdict at every point — was reproduced on the engine revision
  this one descends from, with one documented residual: the model's BER slope
  beyond roughly 25 m is steeper than their measured curve.
- Step-index modal walk-off follows the textbook NA²/(2·n·c) form; at NA 0.2 and
  n = 1.52 that is 43.9 ps/m.
- The LED bandwidth-versus-current-density table is taken from published S21
  data rather than fitted, and the commercial blue-wavelength fiber indices in
  Song et al. (SPIE) give NA = 0.216, independently consistent with the ~0.2
  launch-NA regime used here.

## Honest limits

- **Scalar, power-domain, incoherent.** Modes and wavelengths add in power. That
  is correct for an LED source and wrong for a laser, where the same fiber would
  give modal speckle and dispersion nulls instead of a clean low-pass.
- **The LED model is phenomenological.** Asymmetric IIR plus a fixed tail,
  calibrated against measured BER at 2 to 11 Gb/s. It will not predict a device
  it was not calibrated against.
- **The DFE assumes workable decisions.** Past roughly 1.3 UI of spread the
  feedback is driven by wrong decisions and makes things worse — visible in the
  tool if you push NA and reach far enough. That is physics, not a guard rail.
- **Mode coupling along the fiber is not modeled**, which is second-order at a
  few tens of metres but not at longer reach.
- Jitter, per-lane skew, and inter-lane crosstalk are outside this module.

## Why this exists

I wanted a defensible, physics-first answer to one question: *for a wide-and-slow
LED link, where does the eye actually close, and which knob moves it?* Writing
the whole chain myself — rather than driving a commercial photonic suite — was
the point: every block is one I can defend, calibrate against a public
measurement, and correct when it disagrees with one.
