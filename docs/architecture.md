# Simulation architecture (Basilisk)

[Basilisk](https://github.com/AVSLab/basilisk) (AVS Lab, CU Boulder) is a
real, actively maintained C++/Python astrodynamics and GNC simulation
framework — a much better match for this repo than the earlier FASTCASST
plan, which was written around an unverified block convention we never
actually confirmed against a real checkout. Basilisk ships the physics
substrate (spacecraft hub dynamics, environment, sensors, actuators)
directly; **the estimator and controller logic stay student-written**, per
this repo's collaboration-mode rule in `CLAUDE.md` — Basilisk supplies
truth/sensor data and executes actuator commands, it does not supply the
GNC algorithm.

Exact class/module names below are from memory and need confirming against
the installed version's docs (https://avslab.github.io/basilisk/) before
writing code — Basilisk's API does shift between versions.

## 1. Dynamics block

- Spacecraft hub: `Basilisk.simulation.spacecraft.Spacecraft` — mass, inertia
  tensor, hub state.
- **Important gotcha for this project specifically:** Basilisk's internal
  attitude state representation is MRPs (Modified Rodrigues Parameters), not
  quaternions. `Basilisk.utilities.RigidBodyKinematics` has the
  MRP↔quaternion (`EP`) conversion utilities. Since the whole point of
  `magnetorquer-detumble`'s Stage 1 is deriving your own quaternion
  kinematics, decide deliberately whether this repo's dynamics stay in
  quaternions at the interface (converting to/from Basilisk's MRP state) or
  whether you work in MRPs here — don't let Basilisk's convention silently
  become a second, inconsistent attitude representation across the two
  repos without noticing.
- Environment: `Basilisk.simulation.magneticFieldCenteredDipole` for a
  simple tilted-dipole field (matches what's used on the
  `magnetorquer-detumble` bench), upgradeable to
  `Basilisk.simulation.magneticFieldWMM` (World Magnetic Model, i.e.
  real-fidelity field for a given orbit/epoch) — this replaces the earlier
  plan to pull IGRF from the external `ahrs` package; Basilisk has its own
  higher-fidelity field model built in.

## 2. Actuator blocks

- **Reaction wheel** — `Basilisk.simulation.reactionWheelStateEffector`,
  configured via the `Basilisk.utilities.simIncludeRW` helper. Use the
  flywheel inertia value from the actual CAD design (unprinted) as the
  parameter, so the simulation is literally validating the design that
  hasn't been fabricated yet — state that explicitly wherever this is
  described.
- **Magnetic torque bar** — Basilisk has a dedicated magnetic-torque-bar
  dynamic effector module (exact class name to confirm against the
  installed version's `Basilisk.simulation` namespace). Port the real
  `magnetorquer-detumble` control law in as the commanding logic, not as a
  Basilisk-native block — the control law itself is student-written.
  Cross-validate: does the simulated detumble time roughly match the
  torsion-pendulum bench data from `magnetorquer-detumble`, scaled for the
  different moment of inertia? That sim-vs-hardware comparison plot is the
  single most valuable artifact tying the two repos together.
- **Momentum desaturation** — once the wheel nears saturation, hand control
  back to the magnetic torque bar to dump momentum. This logic is
  student-written (it's the standard "how do reaction wheels and
  magnetorquers work together" systems question) — Basilisk just executes
  whatever torque command this logic produces.

## 3. Sensor blocks

- Magnetometer: `Basilisk.simulation.magnetometer` (TAM — three-axis
  magnetometer — message output).
- Gyro/IMU: `Basilisk.simulation.imuSensor`.
- Sun sensor: `Basilisk.simulation.coarseSunSensor`.

## 4. Estimator and controller — student-written, not Basilisk-native

- Static: TRIAD or QUEST, consuming simulated sun-vector + magnetic-field-
  vector measurements from the sensor blocks above. `ahrs` may be used as a
  reference/oracle to check your own implementation against (same pattern
  as `dynamics_reference.py` in `magnetorquer-detumble`) — not as the thing
  you import and call.
- Dynamic (optional): MEKF against simulated gyro + the same vector
  measurements — the spacecraft-attitude analogue of an underwater EKF.
- Pointing controller (PD or LQR) consumes the estimator's output (not
  ground truth) and commands the reaction wheel — this closes the full
  guidance + navigation + control loop in simulation.
- These plug into Basilisk as custom Python (or C, if performance matters)
  flight-software modules subscribing to sensor messages and publishing
  actuator-command messages — Basilisk's module/message architecture is
  infrastructure (fine to get help wiring up), the algorithm inside each
  module is not.

## Open questions to resolve against the real Basilisk install

- Confirm exact module/class names above against
  https://avslab.github.io/basilisk/ for whatever version gets installed.
- Confirm the magnetic-torque-bar dynamic effector's exact class name and
  configuration interface.
- Confirm `Basilisk.utilities.MonteCarlo.Controller`'s interface for driving
  repeated dispersed runs — see
  [monte_carlo_plan.md](monte_carlo_plan.md).
- Decide the MRP-vs-quaternion boundary noted above before writing any
  interface code between this repo's estimator/controller and Basilisk's
  hub state.
