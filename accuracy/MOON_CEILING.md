# The Moon ceiling — what we ship + what would push past it

**Status as of v9.6.1 (2026-04-26):** Kriya's geocentric Moon residual
against JPL Horizons is **0.87″ in apparent mode** (the production path
that every consumer hits) and **~21″ in geometric mode** (`mean`-only
output, internal). Apparent mode is sub-arcsecond — better than the
catalog's worst non-Moon body (Pallas at 2.84″).

This document is the formal close on the Phase-3 "close the geometric
Moon to sub-arcsec" question. Decision: **we ship the current Moon as
the engine's stable baseline; further closure of the geometric
residual is parked indefinitely.** The rationale + future paths are
below.

## What ships

`lib/ephemeris/bodies/moon.ts` →
- **Production path**: `moonApparentEcliptic(jdTT, nutationLongitudeDeg)` —
  ELP/MPP02 (Chapront & Francou 2003, *A&A* 404, 735) + IAU 1980
  nutation. **Worst |Δλ| 0.87″ across 1899-2100** vs Horizons (DE441).
- **Geometric path**: `moonMeanEcliptic(jdTT)` — same MPP02 series
  without nutation. ~21″ residual.
- **Importable alternatives** for cross-validation:
  `moonMeanEclipticMeeus47` (truncated Meeus ch. 47, ~8″),
  `moonMeanEclipticELP2000_82B` (~30″ vs DE441), and
  `moonMeanEclipticLEA406` (LEA-406 Poisson-series; 20.97″ vs Horizons,
  same DE405-fit ceiling).

## Why the geometric residual is 21″ and won't move with current methods

Both MPP02 and LEA-406 were originally fit to JPL DE405/406. DE441 (the
modern reference Horizons uses) shifts the lunar trajectory by 10-21″
depending on epoch — Lunar Laser Ranging accumulated a decade more
data, and the fit moved. **No analytical lunar theory in the world
fitted to DE405 can close the gap to DE441.** This isn't a bug in our
implementation; it's the published MPP02-vs-DE441 ceiling.

## Two paths that COULD push past

### Path D — DE-441 binary refit of MPP02 amplitudes

**Multi-week.** Required:
1. Download JPL DE-441 SPK file (~3 GB binary).
2. Implement a Chebyshev decoder for the SPK format (the file is a
   piecewise-Chebyshev representation; readers exist in C/Python/Fortran
   but we'd write our own to honor the home-grown rule).
3. Run a least-squares pipeline over MPP02's 25,000-35,000 perturbation
   amplitudes against DE-441 anchor dates spanning a multi-decade window.
4. Validate the refit against held-out Horizons points.
5. Wire the new coefficient table into `lib/ephemeris/bodies/mpp02/`.

**Why we didn't do this:** the home-grown rule means we can't ship
DE-441 *as a binary* (that's what "no third-party astronomy libraries
or data" means as a product position). We CAN fit our analytical
theory against DE-441 numbers — the binary stays a build-time input
not a runtime dependency — but writing the SPK decoder + LSQ pipeline
is genuinely a multi-week build. Path A (refitting MPP02's 19 fit
parameters in `scripts/refit-mpp02.mjs`) was attempted and ruled out
in v9.0.x — those parameters control polynomial drift, not the
35k-amplitude perturbation series where the DE441 residual lives.
Path B (LEA-406 port) was attempted and shipped as the importable
alternative `moonMeanEclipticLEA406`; same DE405-fit ceiling.

### Path C — Numerical n-body integration of the Moon

**Multi-week.** Scaffold parked at `scripts/integrate-moon.mjs`.
v9.6.1 fixed the documented `eclToEqu`/`equToEcl` rotation-matrix
swap (verified by spin-axis roundtrip), so the scaffold compiles and
runs without that defect. **What's still required:**
1. Moon figure perturbation (J2 of the Moon — currently absent).
2. Solar tidal terms beyond the simple point-mass Sun.
3. General-relativistic correction (Schwarzschild metric for Sun-Earth-
   Moon system).
4. Lunar Laser Ranging observation fit — the ~10-100 free parameters
   in the integrator initial conditions need solving against decades
   of LLR range data.
5. Validation against Horizons over 1899-2100.
6. Wire as the production Moon path; retire MPP02 to importable-
   fallback status.

The integrator core (`lib/ephemeris/integrator/dopri8.ts`) is already
proven on Pluto/asteroids/TNOs at sub-arcsec accuracy. Plugging the
Moon in is feasible; the rest of the physics (figure, tides, GR, LLR
fit) is the genuinely-multi-week work.

## What we tell users about Moon accuracy

`lib/api/version.ts:ENGINE.ephemeris.moon` carries the honest
description and is surfaced on `/v1/accuracy`. Users running Western
charts hit the apparent path (sub-arcsec); the geometric path is only
exposed as `moonMeanEcliptic` for internal cross-validation.

## When to revisit

Pick this back up if a customer specifically requires sub-arcsec
*geometric* Moon astrometric (no one has asked through 2026-04-26).
Astrology consumers are on the apparent path and don't see the gap.
If a customer with that requirement appears, restart from this
document — the resumption guide for either path is in
`the accuracy sprint memo (internal)`.
