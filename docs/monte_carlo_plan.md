# Monte Carlo dispersion campaign

This is the part of the study that most fresher-level portfolios skip
entirely — worth doing carefully.

## Disperse per run

- **Initial conditions:** tumble rate and attitude, random draw over a
  realistic post-deployment range
- **Sensor noise:** magnetometer noise floor; gyro bias/random-walk if the
  dynamic (MEKF) estimator is included
- **Actuator uncertainty:** motor torque constant tolerance, flywheel
  inertia manufacturing tolerance (± some %, tied to real 3D-print
  dimensional tolerance), coil resistance drift with temperature
- **Environment uncertainty:** magnetic field model error (a few % is
  realistic)
- **Disturbance torques:** residual magnetic dipole, gravity-gradient
  magnitude at the assumed altitude

## Measure per run

- Detumble settling time
- Final pointing error
- Whether the wheel saturates before desaturation kicks in
- Estimator convergence time and steady-state covariance

## Mechanics

- Basilisk ships a purpose-built Monte Carlo utility:
  `Basilisk.utilities.MonteCarlo.Controller` plus a `Dispersions` module
  (dispersion classes for uniform/normal draws over scalar and vector
  states) and a `RetentionPolicy` mechanism for choosing which logged
  variables survive per run — this replaces the earlier hand-rolled
  "wrap fast.py in a loop" plan. Confirm the exact class names/interface
  against the installed version's docs before writing the driver.
- It supports running dispersed cases in parallel (multiprocessing) —
  worth using given N in the low hundreds, even though each individual run
  is cheap.
- Uniform or Gaussian draws are fine; Latin hypercube sampling is a
  nice-to-have if a more deliberate sampling scheme is worth naming, and
  would need a custom `Dispersions` subclass if Basilisk doesn't ship one.
- N in the low hundreds is enough to show a distribution rather than a
  single seed.
- The *sampling and aggregation logic* — what to disperse, how to reduce
  the run log into the settling-time/pointing-error distributions — is
  student-written per this repo's collaboration mode; Basilisk's
  `Controller`/`RetentionPolicy` are infrastructure for running and logging
  the sweep, not a black box that produces the analysis. Log every run's
  inputs + outputs to a flat file (CSV/parquet) so aggregation is a
  separate, re-runnable script from the sim driver itself.

## Why this matters

Produces answers to questions almost no undergrad portfolio can back up
with data: "what's your 3-sigma detumble time," "how sensitive is pointing
accuracy to flywheel manufacturing tolerance." Both come straight out of
the aggregated run log — no new analysis needed once the campaign exists.

## Status

Not started. Blocked on: Basilisk installed and its Monte Carlo utility's
interface confirmed, reaction wheel + estimator blocks existing (see
[architecture.md](architecture.md)), and `vehicle.yaml`'s dispersion ranges
filled in with real (or at least deliberate, cited) values rather than the
current placeholders.
