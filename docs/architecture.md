# Simulation architecture (FASTCASST)

FASTCASST already supports SIMONLY/SIL against a "satellite" vehicle type.
The plan below extends that vehicle's existing blocks rather than starting
from zero — **before writing any block, check `libraries/` in the actual
FASTCASST checkout for what already exists for dynamics/actuators on the
satellite model.** Nothing here has been cross-checked against a real
FASTCASST clone yet.

## 1. Dynamics block

- Rigid-body rotational EOM (Euler's equation) — presumably already present
  in FASTCASST's satellite model; verify before re-implementing.
- Environment: start with a tilted-dipole magnetic field model (simple,
  matches what's validated on the `magnetorquer-detumble` hardware). For a
  higher-fidelity option, `ahrs`'s geodesy module gives an IGRF-based field
  for a specific orbit/epoch — using FASTCASST (dynamics) and `ahrs` (field
  model) together is worth calling out explicitly as tool integration.

## 2. Actuator blocks

- **B-dot magnetorquer** — port the real control law in as a sim actuator
  block. Cross-validate: does the simulated detumble time roughly match the
  torsion-pendulum bench data from `magnetorquer-detumble`, scaled for the
  different moment of inertia? That sim-vs-hardware comparison plot is the
  single most valuable artifact tying the two repos together.
- **Reaction wheel** — new actuator block. Model: motor torque limit, wheel
  angular momentum limit (saturation). Use the flywheel inertia value from
  the actual CAD design (unprinted) as the parameter, so the simulation is
  literally validating the design that hasn't been fabricated yet — state
  that explicitly wherever this is described.
- **Momentum desaturation** — once the wheel nears saturation, hand control
  back to the magnetorquer to dump momentum. This is the standard
  "how do reaction wheels and magnetorquers work together" systems question;
  simulating it (not just describing it) is the point.

## 3. Estimator blocks

- Static: TRIAD or QUEST from `ahrs`, for coarse attitude from simulated
  sun-vector + magnetic-field-vector measurements.
- Dynamic (optional): MEKF against simulated gyro + the same vector
  measurements — the spacecraft-attitude analogue of an underwater EKF.
- Feed the estimator's output (not ground truth) into the reaction-wheel
  pointing controller (PD or LQR) — this closes the full guidance +
  navigation + control loop in simulation.

## Open questions to resolve against the real FASTCASST checkout

- Confirm the `.h`/`.cpp`-per-SysML-block convention actually holds.
- Confirm what the existing "satellite" vehicle type already implements for
  dynamics/actuators, so this doesn't duplicate it.
- Confirm the `fast.py` Python entry point's interface for driving repeated
  SIMONLY runs (needed for the Monte Carlo campaign — see
  [monte_carlo_plan.md](monte_carlo_plan.md)).
