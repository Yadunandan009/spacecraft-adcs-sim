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

- Driver script wraps FASTCASST's `fast.py` entry point, looping N SIMONLY
  runs with randomized parameters per the list above.
- Uniform or Gaussian draws are fine; Latin hypercube sampling is a
  nice-to-have if a more deliberate sampling scheme is worth naming.
- N in the low hundreds is enough to show a distribution rather than a
  single seed — cheap, since each SIMONLY run is a few seconds, not real
  time.
- Aggregate into histograms / box plots of settling time and pointing error
  across the dispersion; log every run's inputs + outputs to a flat file
  (CSV/parquet) so the aggregation step is a separate, re-runnable script
  from the sim driver itself.

## Why this matters

Produces answers to questions almost no undergrad portfolio can back up
with data: "what's your 3-sigma detumble time," "how sensitive is pointing
accuracy to flywheel manufacturing tolerance." Both come straight out of
the aggregated run log — no new analysis needed once the campaign exists.

## Status

Not started. Blocked on: FASTCASST cloned/integrated locally, reaction
wheel + estimator blocks existing (see
[architecture.md](architecture.md)), and `vehicle.yaml`'s dispersion ranges
filled in with real (or at least deliberate, cited) values rather than the
current placeholders.
