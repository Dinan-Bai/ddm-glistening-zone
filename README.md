# DDM Glistening Zone Explorer

An interactive demo of how a GNSS-R **delay–Doppler map** maps onto the **ocean surface**.

Point at a cell on the DDM and its footprint lights up on the sea — usually in two
places at once. Point at the sea and the matching DDM cell is highlighted. Everything
is a single self-contained `index.html`: no build step, no dependencies, no data files.

## What it shows

Each DDM cell collects power from where an **iso-range ellipse** crosses an
**iso-Doppler hyperbola**. An ellipse and a hyperbola cross *twice*, so one cell sums
power from two patches on opposite sides of the specular point — the two-fold ambiguity
that a single measurement cannot resolve. Only at zero delay do the two merge.

Three linked panels:

| Panel | Contents |
|---|---|
| **Delay–Doppler map** | The horseshoe, formed by binning the surface integral. Viridis or Turbo. |
| **Ocean surface** | Specular-centred plane with iso-delay and iso-Doppler contours and the live footprint. |
| **Scene geometry** | Draggable 3-D schematic of the transmitter, specular point, receiver and glistening zone. |

The DDM and the surface highlight come from the *same* computation, so the mapping is
exact rather than illustrative — the footprint area reported in the readout is measured
by counting the surface pixels that actually fall in the selected bin.

## Model

Flat-plane bistatic geometry in a specular-centred ENU frame.

- **Delay** `(|r − R_t| + |R_r − r| − d₀) / c`, expressed in GPS L1 C/A chips (1 chip = 293.05 m)
- **Doppler** `(û₁·v_t − û₂·v_r) / λ`, referenced to the specular point
- **Surface scattering** Zavorotny–Voronovich geometric optics,
  `σ⁰ = π|ℜ|² (q/q_z)⁴ P(−q⊥/q_z)`, with a Gaussian slope PDF and Katzberg-scaled
  Cox–Munk mean-square slope driven by the wind-speed control
- **DDM formation** the surface integral binned on a 440×440 grid, then convolved with
  `|Λ(Δτ)|² · |sinc(Δf·T_i)|²` for `T_i = 1 ms`

Omitted: Earth curvature, antenna pattern, receiver noise, and any sea-state directionality.
The 3-D scene keeps **angles exact** but compresses distances so the 20 200 km transmitter
range fits on screen.

## Absolute values

The DDM is reported in absolute units, not normalised to its own peak:

- **Received power (dBW)** from the bistatic radar equation
  `P_r = P_t G_t G_r λ² σ / ((4π)³ R_t² R_r²)`, with a *stated* link budget — 26.6 dBW
  transmit EIRP and a flat 14.5 dBi receive gain. These are model values under that budget,
  not calibrated instrument counts.
- **σ⁰ (NBRCS, dB)** — the bin's bistatic radar cross section divided by its effective
  scattering area, so it is nearly independent of bin size. The dBW figure is not, and rises
  as you widen the delay span.

σ⁰ carries the wind signal. Sweeping the wind control at the default geometry:

| Wind | σ⁰ | DDM peak |
|---|---|---|
| 3 m/s | 18.2 dB | −171.7 dBW |
| 5 m/s | 16.4 dB | −173.6 dBW |
| 8 m/s | 14.6 dB | −175.4 dBW |
| 12 m/s | 13.0 dB | −177.1 dBW |
| 20 m/s | 11.0 dB | −179.2 dBW |

That ~7 dB fall from 3 to 20 m/s is the geophysical model function GNSS-R wind retrieval
rests on: rougher sea tilts facets away from the specular direction, so less power returns.

A `rel pk` button restores the conventional peak-normalised view.

## Controls

Incidence angle, receiver altitude, receiver heading ψ, wind speed, and DDM delay span.
Hover either 2-D panel; click to pin; arrow keys step cell by cell; drag the 3-D scene to orbit.

## Resolution

Three distinct quantities, at the default geometry (θ = 30°, 520 km, 12-chip span):

**DDM bin** — 0.464 chip × 278 Hz on a 36 × 28 grid (0.464 chip = 136 m of path).

**Instrument resolution (the WAF)** — coarser than the bins:

| | −3 dB width | first null |
|---|---|---|
| Delay, C/A triangle \|Λ\|² | 0.586 chip ≈ 172 m | 2 chips (586 m) |
| Doppler, \|sinc(Δf·T_i)\|², T_i = 1 ms | 886 Hz | 1000 Hz |

So the grid oversamples by 1.26× in delay and 3.2× in Doppler — adjacent Doppler cells
are not independent.

**Ground footprint** — set by the angle at which the two contour families cross, not by
the bin sizes:

| Location | Across iso-delay | Across iso-Doppler | Crossing angle | Patch area |
|---|---|---|---|---|
| 2 km from specular | 53 km | 5.6 km | 1° | ~34 000 km² |
| τ ≈ 3 chips | 2.7 km | 5.3 km | 50° | 18.5 km² |
| τ ≈ 6 chips, cross-plane | 1.7 km | 5.6 km | 85° | 9.8 km² |
| τ = 12 chips, arch tip | 1.3 km | 4.7 km | 0° | ~840 km² |

Resolution is best where the ellipse and hyperbola meet near-perpendicular, and collapses
where they run tangent — on the incidence-plane axis, at both the specular peak and the
arch tips. That is the same degeneracy that merges the two ambiguous patches at zero delay.

## Running it

Open `index.html` in any modern browser. Fonts load from Google Fonts; everything else is inline.
