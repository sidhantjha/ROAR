# ROAR — active workstream: section-tuned Pure Pursuit

## Current active task — centerline section-tuned Pure Pursuit (2026-11-18)
**User's chosen development sequence:** protect the 417.30 s controller (20% original racing line), create a manual copy as `src/pursuer_tuned/{__init__.py,config.py,controller.py}` based on user-uploaded working `src/pursuer_racing/controller.py`. Keep **exact Pure Pursuit, path-segment projection, adaptive lookahead, curvature speed cap, backward braking and PI throttle**; change **only** `self.waypoints = list(waypoints)` instead of optimized-line loader and delete the unused fixed-3m/s² forward profile, racing-line fields and unimplemented measured-planner/combined-grip flags. Config must contain only referenced parameters. Do not modify source files or create downloadable code on behalf of the user; provide full inline copy/paste code with clear explanation. New centerline controller is **not** expected to reproduce 417.30 s, since that run used 20% original min-curvature blend.

**Initial config for A/B isolation:** use original proven global lateral target 34 m/s², braking assumption 4m/s², existing sections WP470–550 and WP1880–2020 at 32m/s², while keeping other analytically identified corner windows at global 34 (not newly slowing all corners). Already identified eight candidate tuning windows from user-run offline track map; start by adjusting one section at a time after a centerline baseline. Record first collision/section, entry and exit speed, path error, throttle/brake and sector elapsed time. A one-lap pass is an intermediate gate; repeat full 3 laps to qualify a faster build. Optimize braking/accel once failure modes and speed bottlenecks are identified using existing 13 acceleration/22 braking measured bins; true corner and combined-grip limits not yet calibrated. Then introduce selected local racing-line offsets at demonstrated geometry-limited corners and retune section speeds. Use existing raw track geometry export `tools/mincurv/data/monza_track.npz` (2,775 centerline WPs).

**Next concrete action:** user creates `src/pursuer_tuned` in VS Code from supplied inline copy/paste, checks the two import references in `submission.py` (exact file not inspected) and runs syntax check then simulator. Preserve protected original and restore imports after tests.

**Historical work below:** prior 65 m/s braking tests and their unresolved 3 m tracking abort remain evidence but are not the immediate task.


**Updated:** 2026-11-16 after last screenshot/terminal outcome. **Current phase:** ROADMAP phase 3 (IN PROGRESS, high-speed extension BLOCKED).

## Immediate objective
Extend the **measured, reusable** longitudinal dynamics profile from confirmed 20–55 m/s to the highest *attainable and safely testable* speeds (preferably 65, then 70, optionally 80–90 m/s). Avoid inferring braking data from non-braking runs. Keep the best racing configuration intact.

## Why and chosen approach
The 417.30 s reference agent peaked near 65.51 m/s, but its commanded speed stayed substantially above actual speed and it **never applied brakes** in that run. A useful forward/backward planner needs speed-dependent acceleration *and* braking, later combined with lateral grip; blind changes to brake gain or global decel caps won't solve absent brake requests.

Use controlled repeatable CARLA tests in standalone `tools/vehicle_profile` scripts, logging measured outcomes; publish to a **single** `src/pursuer_racing/vehicle_dynamics.json` only when validated. The test script must find sufficient track for high-speed acceleration and braking, follow the lane robustly and preserve aborts.

## Last verified sequence
1. User instrumented `submission.py` (20 Hz world/local velocity, acceleration, yaw rate, control/collision CSV). Latest logged telemetry `runs/pursuer_20261116_151919.csv`: 8,379 samples.
2. `fit_accel.py` produced 5 m/s acceleration bins; last 60–65 bin near 2.07 m/s² median, 22 samples.
3. Controlled braking `brake_test.py` completed 20/35 m/s at 0.5 and 1.0 brake; `brake_test_highspeed.py` completed 45/55 m/s likewise. See EXPERIMENTS.
4. Two 65 m/s high-speed tests aborted **before braking** at 58.51 m/s, cross-track error 3.07 m > 3.00 m.
5. `python -m tools.vehicle_profile.build_vehicle_model` succeeded: 8 valid files, 2 incomplete skipped, 13 acceleration bins, 22 braking bins. Local `src/pursuer_racing/vehicle_dynamics.json` printed as saved. Max reported braking model midpoint 52.5 m/s (not extrapolated).
6. The new `brake_test_straight.py` intended to use the race-tested racing steering / wait for straight also **aborted**, with terminal screenshot reporting **tracking error 3.14 m > 3.00 m**, max **55.89 m/s** of 65 target, **0 ticks** within 2 m/s of target, **0 m** qualifying straight near target, required straight to trigger **324 m**, and **0.0 m braking distance**. CSV path reported: `tools/vehicle_profile/data/brake_20261116_182603_v65_b100.csv`. No valid brake measurement. The exact local contents of this helper script have not been independently inspected by this checkpoint.

## Key technical diagnosis — tentative vs verified
**Verified:** both test approaches abort on **cross-track error**, before any 65 m/s braking phase. A 65.5 m/s race speed was seen earlier in another configuration/run. **Not yet established:** whether lack of eligible straight, control/lookahead mismatch, racing-line curvature, aggressive throttle, lane geometry, different controller config, or lateral grip dominates. Do **not** assert motor top-speed limit or weak steering motors. Do not increase steering gain or expand 3 m error limit blindly.

**Candidate next test:** inspect script and short pre-abort telemetry, derive exact waypoint segment + curvature/heading/steering behavior, find an adequate braking corridor in exported track. Compare speed / path error for `brake_test_straight` against the completed racing lap at that waypoint. A 324 m criterion with 0 qualifying spans may be overconservative or intrinsically unavailable at 65 m/s in this layout. Determine feasible acceleration+braking entry placement or use a race-state-triggered decel test with a verified clear corridor. A failed 65 test should not be followed by 70–90 attempts.

## Files and commands
Local workspace: `%USERPROFILE%\Documents\ROAR\ROAR\competition_code`.
Simulator: `%USERPROFILE%\Documents\ROAR-Simulator\Monza\CarlaUE4.exe`.
Environment: `roar_competition` (conda), CARLA API 0.9.12 versus simulator 0.9.12-dirty.

```bat
call "%USERPROFILE%\anaconda3\Scripts\activate.bat" roar_competition
cd /d "%USERPROFILE%\Documents\ROAR\ROAR\competition_code"
python -m tools.vehicle_profile.build_vehicle_model
python -m tools.vehicle_profile.brake_test_straight --target-speed 65 --brake 1.0
```
The last command currently reproduces an **abort**, and should **not** just be rerun unchanged. Use `python competition_runner.py` only for clean full-race validations (with simulator launched and no concurrent standalone test).

Important local files:
- `tools/vehicle_profile/brake_test_straight.py`, `brake_test_highspeed.py`, `brake_test.py`: controlled test logic.
- `tools/mincurv/data/monza_track.npz`: exported geometry and waypoints.
- `src/pursuer_racing/controller.py`, `config.py`, `racing_line_loader.py`: steering/line/speed behavior.
- `src/pursuer_racing/vehicle_dynamics.json`: successful 20–55 m/s model checkpoint.
- `runs/pursuer_20261116_135735.csv`: best successful 417.30 s race telemetry.
- `tools/vehicle_profile/data/brake_*.csv`: original raw braking evidence; distinguish COMPLETED vs ABORTED.

## Next exact action
**Inspect the current local `brake_test_straight.py` and the aborted CSV `brake_20261116_182603_v65_b100.csv`, plus relevant `controller.py` / `config.py` and track geometry.** Identify why error reaches 3.14 m before 65 m/s and why the script finds zero qualifying straight at 55.89 m/s; measure available straight length and demanded braking distance. Do not write a new controller until the failure mechanism is observed.

## Following 5 actions
1. Produce a short diagnostic table by waypoint/distance: speed, lateral error, steer command, heading/yaw, path curvature, qualifying straight length, and target-speed gate.
2. Design a safe, straight-aware or race-triggered 65 m/s braking experiment with adequate **acceleration and stopping distance**, explicit guards and cleanup. Preserve the 3 m error abort unless justified by actual track margins.
3. Run **65 m/s / 100% brake**; inspect actual brake onset and all braking samples. Add 50% and then 70, 80, 90 only if feasible and safe; stop escalation on abort.
4. Rebuild `vehicle_dynamics.json`; assert per-bin measured coverage and sample counts, and never extrapolate unmeasured high-speed coefficients.
5. Proceed to lateral grip / combined friction testing, then forward/backward profile integration and three-lap comparison vs 417.30 s.

## Current blockers / unknowns
- Full current local source and latest script implementations are **not present in GitHub remote** and could not be audited here. User must share files or use a connected local workflow before exact fixes can be specified safely.
- Whether Monza offers a suitable clear/straight stretch for target 65–90 m/s testing is unverified.
- Actual final submission rule interpretation and currently effective `pursuer_racing/config.py` require inspection.

