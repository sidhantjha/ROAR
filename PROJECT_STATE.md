# ROAR — verified current project state

**As of:** 2026-11-16, based on this ChatGPT Project conversation, user-shared terminal outputs/screenshots, prior relevant conversation context, and GitHub repository inspection. This file distinguishes confirmed *reported local runs* from remotely inspectable files.

## Mission and success criteria
Develop a fast, reliable, rules-compliant autonomous racing controller for **Berkeley ROAR Fall 2026**, CARLA **Monza v1.1**, fixed circuit. Optimize the official three-lap simulated race time without significant collisions or off-track failures. Maintain a stable baseline; measure performance through recorded 20 Hz telemetry and repeatable tests. An earlier conversation identified optional test on **2026-12-04** and final submission **2026-12-11 23:59 PST**; these dates are **reported but not reverified** against the current official rules. The user wants a personally understandable, compact implementation and clear physics reasoning, not unvalidated complexity.

## Source and persistence status
- **Writable connected GitHub repository:** `sidhantjha/ROAR` (`main`), public, admin/push permission confirmed. At initialization its remote tree contained only `.gitignore`, `README.md`, and `competition_code/{competition_runner.py,infrastructure.py,submission.py}` starter files (latest inspected remote starter commit dated 2023). No pre-existing seven canonical Markdown files were found at the remote root.
- **Active local Windows workspace, user-confirmed:** `%USERPROFILE%\Documents\ROAR\ROAR\competition_code`; simulator `%USERPROFILE%\Documents\ROAR-Simulator\Monza\CarlaUE4.exe`. User's current code, `src/pursuer_racing`, tools, logs and `vehicle_dynamics.json` are **local only / not present in the inspected GitHub remote**. Do not confuse remote starter source with active local source.
- The project conversation supplied code snippets, tool-download scripts, CLI outputs and screenshots; an independent audit of the current entire local working tree has **not** occurred. Figures below are verified as user-shared results, not rerun here.

## Environment, interface and rules
- Windows 11, conda env `roar_competition`, Python 3.8.20 as previously reported, CARLA client API 0.9.12 / Monza simulator `0.9.12-dirty` (warning seen repeatedly, runs still executed).
- Run (after launching simulator): `call "%USERPROFILE%\anaconda3\Scripts\activate.bat" roar_competition`; from `competition_code`: `python competition_runner.py`.
- Official `competition_runner.py` spawns `vehicle.tesla.model3` at `world.maneuverable_waypoints[0]` and runs `world.set_control_steps(0.05,0.005)` / synchronous world stepping. Approximately **2,775 waypoints/lap**, repeated three laps.
- User-shared starter `submission.py` explicitly says **"Please do not change anything else but fill out the to-do sections."** Current local `submission.py` includes extended telemetry, CSV and collision abort functionality. **Submission-rule compliance of those modifications has not been audited**; keep test instrumentation in test code or permitted areas in the final artifact.
- Do not edit the official runner. Restart simulator if orphaned vehicle state or other errors persist.

## Active architecture — local, user-reported
```text
competition_code/
  competition_runner.py          # official: untouched
  submission.py                  # local adapter to active controller + 20 Hz CSV
  src/pursuer/                  # early controller
  src/pursuer_sections/         # protected successful reference (~422.05 s)
  src/pursuer_racing/
    config.py
    controller.py
    racing_line.py
    racing_line_loader.py
    vehicle_dynamics.json        # measured longitudinal data; generated locally
  tools/mincurv/                 # export, optimize, repair, validate, compare
  tools/vehicle_profile/
    profile_run.py
    fit_accel.py
    brake_test.py
    brake_test_highspeed.py
    brake_test_straight.py
    fit_braking.py
    build_vehicle_model.py
    acceleration_model.csv
    data/brake_*.csv
    data/braking_model.csv
  runs/pursuer_*.csv
```
The active race controller implements kinematic bicycle **Pure Pursuit**, adaptive speed-based lookahead, curvature-derived corner speed bounds, backward braking profile, PI throttle / P brake. It can load an optimized racing line at a configurable blend. A forward acceleration-limited pass was tested but disabled because it significantly slowed performance. Current final active config/commit is **not independently verified**; do not presume the latest experiment used exactly the best 417.30 s configuration.

## Verified accomplishments (user-reported race/test output)
- Baseline starter: approximately **1658.9 s** for three laps at ~20 m/s. Early Pure Pursuit 30 m/s: **981.9 s**. Curvature planning and increased speed targets: **778.45 s**. PI throttle implementation: **615.50 s**. Iterative grip/sections tuning reduced to **~422.05 s**.
- **Fastest logged successful configuration:** **417.30 s** (20% blend of original optimized minimum-curvature line; local grip overrides) on `runs/pursuer_20261116_135735.csv`. It was a single reported successful run, not a statistically repeated guarantee.
- **Safe repaired line:** ~**419.0 s** for 50% blend on `runs/pursuer_20261116_144551.csv`. Original un-repaired 50% blend crashed; the repaired optimizer reduced path shift and peak-curvature regressions.
- Minimum-curvature initial optimization on 2,775 waypoints gave **36.1% lower RMS curvature**, but introduced sharp local peak curvature and non-convergence/iteration limit. Geometry metrics were not a guarantee of racing performance.
- Enhanced 20 Hz telemetry implemented and successfully recorded with world/local velocity, projected acceleration and yaw rate. `runs/pursuer_20261116_151919.csv` had **8,379 rows**, ~419 s at nominal 20 Hz.
- Full-throttle acceleration-versus-speed data extracted: **13 model bins**; representative near-straight medians ~4.03 m/s² (10–15 m/s), 3.28 (30–35), 2.46 (50–55), 2.07 (60–65). Observational samples, not physical maxima.
- Controlled braking tests **completed at 20, 35, 45 and 55 m/s**, each at 50% and 100% brake (8 files). Values and filenames in EXPERIMENTS.
- `python -m tools.vehicle_profile.build_vehicle_model` printed **8 completed brake test files, 2 skipped incomplete, 13 acceleration bins, 22 braking bins**, and `SAVED: src\pursuer_racing\vehicle_dynamics.json`. Printed measured braking-bin midpoints run from 2.5 to **52.5 m/s** per brake input. It explicitly did **not** extrapolate.
- High-speed 65 m/s braking characterization is **blocked**: original test abort at **58.51 m/s**, error **3.07 > 3 m**, zero braking; straight-aware test also aborted before braking at **55.89 m/s**, error **3.14 > 3 m**, zero qualified straight and zero braking. No valid 65+ m/s braking measurements.

## Important observations and limits
- Best race telemetry: average speed ~40.54 m/s, peak ~65.51 m/s, ~53.3% ticks at throttle ≥95%; **no brake commands** and target speeds well above actual. Fixing `BRAKE_KP` alone would not cause braking; planner must request realistic lower speeds at corners.
- The existing model's braking samples measure **straight-line** total deceleration (may include aerodynamic drag), not guaranteed brake/tire performance in turns. Sparse high-speed bins and transient onset require careful filtering. `MAX_BRAKING_DECEL_MPS2=4.0` is conservative versus controlled straight-line tests, but should **not** simply be replaced by 13–17 m/s² without friction/combined-force validation.
- The simulator has five CARLA native spawn points, but the competition always uses fixed waypoint spawn; the current test route reaches tracking-error abort well before target 65 m/s. Increasing steering gain or disabling abort is not established as a solution.

## Resources / known constraints
- Nominal exported lane width ~12 m; map geometry clearance needs validation against actual collision boundaries.
- Relevant external reference software previously considered: TUM global racetrajectory optimization / trajectory_planning_helpers; preserve verified local values rather than importing untested algorithms.
- Prefer physics-informed speed planning + clean race line + telemetry and section-by-section validation; RL/MPC are not immediate priorities.
- GitHub remote and local active source are **out of sync**. Before publishing code, check `git status`, `git remote -v`, secrets, binaries and relevant competition rules. This documentation checkpoint is not a verified code backup.
