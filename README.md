# spacecraft-adcs-sim

Simulation-only study — B-dot is cross-validated against hardware in the
companion [`magnetorquer-detumble`](../magnetorquer-detumble) repo; reaction
wheel control and attitude estimators here are SIL-only, pending flywheel
fabrication.

## Why this is a separate repo

`magnetorquer-detumble` is the hardware-validated story: real QMC5883L
magnetometer, real L298N-driven ferrite-core coils, torsion-pendulum bench,
a real dB/dt loop. Everything in *this* repo — rigid-body dynamics extended with a
reaction wheel, static/dynamic attitude estimators, Monte Carlo dispersion —
is a software study built in FASTCASST. No physical flywheel exists yet; its
inertia here comes from the CAD design, not a measurement.

Keeping the two repos separate keeps the hardware-validated claim clean and
lets this repo's README say up front, unambiguously, what is and isn't
flight-tested.

## Scope

- **Ports in (validated elsewhere):** B-dot control law, rigid-body 6-DOF
  dynamics — extended here, not re-derived.
- **New, simulation-only:**
  - Reaction wheel actuator block (motor torque limit, momentum saturation,
    momentum desaturation handoff to the magnetorquer)
  - Attitude estimators: TRIAD/QUEST (static, via `ahrs`) and optionally an
    MEKF (dynamic, gyro + vector measurements)
  - Pointing controller (PD/LQR) closing the loop on estimated attitude,
    not ground truth
  - Monte Carlo dispersion campaign across initial conditions, sensor
    noise, actuator/manufacturing tolerance, and environment uncertainty

See [docs/architecture.md](docs/architecture.md) for the FASTCASST block
plan and [docs/monte_carlo_plan.md](docs/monte_carlo_plan.md) for the
dispersion campaign design.

## Status / action items

Tracked in [STATUS.md](STATUS.md). Currently: not started — next action is
cloning FASTCASST and confirming its `libraries/` block convention before
writing any block code.
