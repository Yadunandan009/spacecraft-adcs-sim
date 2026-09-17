# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project

Simulation-only spacecraft ADCS study, built in Basilisk (AVS Lab, CU
Boulder — https://github.com/AVSLab/basilisk): rigid-body
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

## Collaboration mode (read this before writing any algorithm code)

This repo is where the estimator/attitude-determination work lives (TRIAD,
Davenport, QUEST, MEKF) — the exact material the user is trying to learn,
not just ship. Same 🟢🟡🔴 discipline as `~/magnetorquer-detumble` (full
rationale there):

- 🟢 Green: explain concepts, review an existing derivation, check
  conventions/units, suggest test cases, Socratic questioning.
- 🟡 Yellow: debugging, numerical methods, controller/estimator design —
  only after the user has an attempt to show.
- 🔴 Red — do not write outright: the rigid-body dynamics extension, any
  estimator (TRIAD/Davenport/QUEST/MEKF), the reaction wheel actuator
  block, the pointing controller, the momentum-desaturation logic, the
  Monte Carlo driver's core sampling/aggregation logic. These are the
  "I understand spacecraft GNC" half of the portfolio story — if Claude
  writes them, that claim becomes false. If asked to implement one
  directly, ask for the derivation/attempt first instead of complying.

Ask "what's your attempt?" before writing red-zone code, even if a fast
answer is obviously possible. Basilisk integration/plumbing (module
wiring, message setup), YAML config, docs, and test harness scaffolding are
not red-zone — those are fine to help with directly. The line: Basilisk
supplies simulated truth/sensor data and executes actuator commands; the
estimator and controller that sit between those are student-written.

## Before writing block code

Basilisk (https://github.com/AVSLab/basilisk,
https://avslab.github.io/basilisk/) is not yet installed/integrated into
this repo — replacing the earlier FASTCASST plan, which was written around
an unverified block convention. Before writing any block: confirm the
exact module/class names against the installed version's docs (Basilisk's
API shifts between versions), and note the MRP-vs-quaternion attitude
representation mismatch flagged in
[docs/architecture.md](docs/architecture.md) — Basilisk's hub state is
MRPs, this project's own dynamics work is quaternions.

## Architecture and plan

- [docs/architecture.md](docs/architecture.md) — dynamics/actuator/estimator
  block plan (Basilisk modules), environment model choice (centered-dipole
  vs. Basilisk's own WMM model), momentum desaturation logic
- [docs/monte_carlo_plan.md](docs/monte_carlo_plan.md) — dispersion campaign:
  what gets randomized, what gets measured, sample size, driver-script shape
  (Basilisk's `MonteCarlo.Controller` utility)
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
(coils/STM32/L298N) arriving — it can run in parallel with
`magnetorquer-detumble`'s hardware bring-up.
