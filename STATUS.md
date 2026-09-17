# Status / Action Items

Living checklist — check items off as they're done and commit the change.
Design rationale lives in [docs/architecture.md](docs/architecture.md) and
[docs/monte_carlo_plan.md](docs/monte_carlo_plan.md); this file is the fast
"what's done, what's next" view. 🔴 = core algorithm work, student-written
per `CLAUDE.md`'s collaboration-mode section — Claude reviews after an
attempt, doesn't author it.

## Setup

- [ ] Install Basilisk locally (https://avslab.github.io/basilisk/ has the
      current install method), confirm the module/class API against
      whatever version installs — names in the docs here are from memory
      and need verifying
- [ ] Install the `ahrs` Python package (reference/oracle only, not the
      estimator implementation source)
- [ ] Resolve the MRP-vs-quaternion attitude representation question (see
      `docs/architecture.md`) before writing any interface code

## Dynamics & environment

- [ ] Wire up `Basilisk.simulation.spacecraft.Spacecraft` with this
      project's mass/inertia parameters
- [ ] Implement the centered-dipole environment model
      (`Basilisk.simulation.magneticFieldCenteredDipole`)
- [ ] (stretch) Swap in `Basilisk.simulation.magneticFieldWMM` for
      higher-fidelity field

## Actuators

- [ ] 🔴 Port the B-dot control law in as the magnetic-torque-bar commanding
      logic (Basilisk executes the torque, the control law is yours)
- [ ] Cross-validate simulated detumble time against `magnetorquer-detumble`
      bench data (once that bench exists)
- [ ] Wire up `Basilisk.simulation.reactionWheelStateEffector` with the
      CAD-only flywheel inertia from `vehicle.yaml`
- [ ] 🔴 Implement momentum desaturation handoff logic to the magnetic
      torque bar

## Estimators

- [ ] 🔴 Derive and implement the TRIAD/QUEST static estimator yourself;
      check against `ahrs`'s implementation, don't call it directly
- [ ] 🔴 (stretch) Derive and implement an MEKF dynamic estimator
- [ ] 🔴 Implement the PD/LQR pointing controller closing the loop on
      estimated (not ground-truth) attitude

## Monte Carlo

- [ ] Set up a `Basilisk.utilities.MonteCarlo.Controller`-driven runner
      (confirm its interface against installed docs first)
- [ ] 🔴 Design the dispersion draws per `vehicle.yaml`'s `monte_carlo`
      block — deciding what to disperse and why is part of the exercise
- [ ] Aggregate and plot settling-time / pointing-error distributions
      across runs

## Next up

Install Basilisk and work through one of its own example scenarios first
(before touching this project's code) to get the module/message API
straight — everything else here is blocked on that.
