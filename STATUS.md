# Status / Action Items

Living checklist — check items off as they're done and commit the change.
Design rationale lives in [docs/architecture.md](docs/architecture.md) and
[docs/monte_carlo_plan.md](docs/monte_carlo_plan.md); this file is the fast
"what's done, what's next" view.

## Setup

- [ ] Clone/inspect FASTCASST locally, confirm the `libraries/` per-SysML-block convention
- [ ] Install the `ahrs` Python package
- [ ] Confirm what the existing "satellite" vehicle type already implements for dynamics/actuators (avoid duplicating)

## Dynamics & environment

- [ ] Verify/extend the rigid-body EOM block
- [ ] Implement the tilted-dipole environment model (`vehicle.yaml: environment.model`)
- [ ] (stretch) Wire in the `ahrs` IGRF field model as an alternate environment

## Actuators

- [ ] Port the B-dot control law in as a FASTCASST actuator block
- [ ] Cross-validate simulated detumble time against `magnetorquer-detumble` bench data (once that bench exists)
- [ ] Implement the reaction wheel actuator block (CAD-only inertia from `vehicle.yaml`)
- [ ] Implement momentum desaturation handoff to the magnetorquer

## Estimators

- [ ] Implement the TRIAD/QUEST static estimator via `ahrs`
- [ ] (stretch) Implement an MEKF dynamic estimator
- [ ] Implement the PD/LQR pointing controller closing the loop on estimated (not ground-truth) attitude

## Monte Carlo

- [ ] Write a `fast.py`-driven Monte Carlo runner
- [ ] Implement the dispersion draws per `vehicle.yaml`'s `monte_carlo` block
- [ ] Aggregate and plot settling-time / pointing-error distributions across runs

## Next up

Clone FASTCASST and confirm the `libraries/` convention — everything else
here is blocked on that.
