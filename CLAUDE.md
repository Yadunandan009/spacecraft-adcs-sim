# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project

Simulation-only spacecraft ADCS study, built in FASTCASST: rigid-body
dynamics (ported/extended from the hardware-validated model) + reaction
wheel actuator + attitude estimators (TRIAD/QUEST/MEKF) + Monte Carlo
dispersion analysis. This is **not** a hardware-validated project — no
physical flywheel or estimator hardware exists. It is standalone, separate
from `~/ros2_ws` (BlueROV2/ArduSub) and from `~/magnetorquer-detumble`
(hardware-validated B-dot).

**Scope boundary — say this out loud, don't let it blur:** B-dot is
hardware-validated in `~/magnetorquer-detumble`. Everything added in *this*
repo (reaction wheel, estimators, dispersion analysis) is simulation-only.
When describing this work (README, CV, interview), always state that split
explicitly rather than letting the simulation work read as "I built a
reaction wheel." The framing: modeled and controlled a reaction wheel in a
SIL environment because the flywheel print didn't match the motor's
inertia/torque envelope — de-risking the algorithm before committing to a
print.

## Before writing block code

FASTCASST is not yet integrated into this repo. Check what already exists
in FASTCASST's `libraries/` for the satellite vehicle type (it already
supports SIMONLY/SIL against a "satellite" vehicle per its own README) —
extend the existing dynamics/actuator blocks rather than duplicating them.
The one-`.h`/`.cpp`-pair-per-SysML-block convention needs confirming against
the actual cloned framework before assuming it applies verbatim.

## Architecture and plan

- [docs/architecture.md](docs/architecture.md) — dynamics/actuator/estimator
  block plan, environment model choice (tilted dipole vs. `ahrs` IGRF),
  momentum desaturation logic
- [docs/monte_carlo_plan.md](docs/monte_carlo_plan.md) — dispersion campaign:
  what gets randomized, what gets measured, sample size, driver-script shape
- `vehicle.yaml` — parameter scaffold for the reaction wheel, estimators,
  and Monte Carlo dispersion ranges. Everything in it is `ASSUMED` or
  `CAD_ONLY` — there is no hardware-measured value in this repo by design.

## Relationship to magnetorquer-detumble

Cross-validate: simulated B-dot detumble time (using this repo's dynamics)
should roughly match `~/magnetorquer-detumble`'s torsion-pendulum bench data
once that bench exists, scaled for the different moment of inertia. That
comparison plot is the connective tissue between the two repos — build it
once both sides have real numbers.

## Timeline note

This track is pure software and doesn't need to wait on hardware parts
(coils/STM32/DRV8833) arriving — it can run in parallel with
`magnetorquer-detumble`'s hardware bring-up.
