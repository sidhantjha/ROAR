# ROAR — experiment and evidence log

**Reconstructed:** 2026-11-16. Values come from user-shared runs and summaries of earlier project exchanges; raw files reside in the user's local workspace, **not** in the inspected GitHub remote. For future trials record date, simulator config, controller config/commit, command, CSV, raw sample counts, success/abort status, best/median/p20 and failure reason.

## A. Race/controller progression (three laps, approximate elapsed sim time)
| Intervention / configuration | Outcome | Evidence / caveat |
|---|---:|---|
| Official beginner ~20 m/s speed target | 1658.9 s | Reported early run |
| Pure Pursuit ~30 m/s | 981.9 s | Reported successful run |
| Curvature speed limits and speed target increase | 778.45 s | Reported |
| PI throttle implementation | 615.50 s | Reported |
| Artificial 3 m/s² acceleration forward-pass | ~694.05 s | **Regression** from previous ~615 s; disabled |
| Grip ~32–33 m/s² + per-section calibration | ~425–423.65 s | Reported |
| Global lateral assumption 34 m/s² without local bound | **CRASH** near WP512 | Unsafe, not faster completion |
| Stable `pursuer_sections` with local override | ~422.05 s | Protected comparison reference |
| 20% original min-curvature blend + two local grip overrides | **417.30 s completed** | `runs/pursuer_20261116_135735.csv`; best observed |
| 50% repaired min-curvature blend | **~419.0 s completed** | `runs/pursuer_20261116_144551.csv` |
| Original 50% line without repair | **CRASH** near WP1954 | Excessive local curvature / path behavior |

Best race telemetry (417.30 s): 8,346 ticks; mean speed **40.54 m/s**, maximum **65.51 m/s**, mean planned target **59.86 m/s**, throttle ≥0.95 for **53.3%** of ticks, **zero measured braking commands**, mean target-minus-actual speed **19.32 m/s**, mean path tracking error ~0.68 m. Thus lack of brake demand, not just brake response gain, is important.

## B. Offline racing-line experiment
- Exported approximately **2,775** Monza waypoints, nominal ~12 m lane width, `tools/mincurv/data/monza_track.npz`.
- Original min-curvature optimization: ~**36.1% lower RMS curvature** (0.010728→0.006854), objective ~52.3% lower, but 2 m max lateral shift, increased local peak curvature and SciPy iteration-limit non-convergence; not sufficient as a safe driving solution.
- Repaired optimization penalized peaks and smoothed WP1880–2020, yielding candidate 50% blend with ~0.987 m max lateral shift; optimizer still hit iteration limit and reported geometry clearance (~5.013 m) is a nominal geometric criterion, **not proof** of collision clearance. At 50% repaired path the race completed ~419 s.
- `tools/mincurv/{export_track,optimize_line,validate_line,optimize_line_repaired,validate_repaired_line,compare_runs}.py` are reported local utilities. Line file / blend values must match loader metadata; mismatch previously caused initialization failure and left orphan actor until simulator restart.

## C. Instrumented vehicle telemetry
`runs/pursuer_20261116_151919.csv`: **8,379** rows at nominal dt=0.05 s, columns include speed, global position, heading, waypoint index/error, target speed, steer/throttle/brake/collision, `vx_world,vy_world,vz_world,v_forward_mps,v_lateral_mps,a_forward_mps2,a_lateral_mps2,yaw_rate_rads`. Last example sample: forward speed **22.9957 m/s**, lateral **−0.0719 m/s**, forward acceleration **3.6495 m/s²**, lateral **1.3184 m/s²**, yaw **0.07051 rad/s**. These are instantaneous, not maxima. Differencing global velocity and projecting onto vehicle heading estimates inertial acceleration; assume nominal 20 Hz and align *previous* commanded controls with following observed acceleration.

### High-throttle near-straight acceleration bins (`tools/vehicle_profile/acceleration_model.csv`)
| Speed range (m/s) | n | Median forward accel (m/s²) |
|---|---:|---:|
| 5–10 | 24 | 4.16 |
| 10–15 | 25 | 4.03 |
| 15–20 | 26 | 3.85 |
| 20–25 | 92 | 3.68 |
| 25–30 | 155 | 3.47 |
| 30–35 | 245 | 3.28 |
| 35–40 | 272 | 3.09 |
| 40–45 | 210 | 2.90 |
| 45–50 | 187 | 2.71 |
| 50–55 | 216 | 2.46 |
| 55–60 | 155 | 2.28 |
| 60–65 | 22 | 2.07 |
0–5 m/s: n=38 median ~0 and high P90; treated as transient/noisy for fit. These are observed high-throttle performance on near straights, **not guaranteed maximum acceleration**.

## D. Controlled braking results (all COMPLETED; from same fixed competition spawn; straight-biased)
| Requested starting speed | Brake | Actual max (m/s) | Median decel (m/s²) | P90 decel | Distance to ~2 m/s | Duration | CSV timestamp |
|---|---:|---:|---:|---:|---:|---:|---|
| 20 m/s | 0.50 | 20.30 | 6.82 | 8.68 | 30.2 m | 2.85 s | `154617_v20_b050` |
| 20 m/s | 1.00 | 20.30 | 10.42 | 11.83 | 21.7 m | 1.95 s | `154725_v20_b100` |
| 35 m/s | 0.50 | 35.21 | 8.25 | 11.69 | 68.3 m | 4.15 s | `154805_v35_b050` |
| 35 m/s | 1.00 | 35.19 | 11.94 | 14.55 | 52.5 m | 2.95 s | `154814_v35_b100` |
| 45 m/s | 0.50 | 45.11 | 9.03 | 13.36 | 97.0 m | 4.80 s | `164754_v45_b050` |
| 45 m/s | 1.00 | 45.10 | 12.86 | 16.00 | 77.2 m | 3.50 s | `164742_v45_b100` |
| 55 m/s | 0.50 | 55.05 | 9.83 | 14.36 | 126.8 m | 5.30 s | `164816_v55_b050` |
| 55 m/s | 1.00 | 55.05 | 13.85 | 17.01 | 103.9 m | 3.95 s | `164804_v55_b100` |
CSV prefix for all rows: `tools/vehicle_profile/data/brake_20261116_<timestamp>.csv`.
**Caution:** median deceleration is a sample statistic, not constant effective decel, a guarantee, or full 60–90 m/s coverage. Onset transients and aerodynamic drag may contribute. Some bins have very few samples.

### Brake model fit and persistence
`python -m tools.vehicle_profile.fit_braking` reported **4 files, 220 usable brake-response samples** on the initial 20/35 tests, grouping by 5 m/s and brake 0.5 / 1.0. P20 is a conservative *observed quantile*, not guaranteed limit.

`python -m tools.vehicle_profile.build_vehicle_model` after 55 m/s tests printed:
```text
Completed brake test files: 8
Skipped incomplete brake tests: 2
Acceleration bins: 13
Braking bins: 22
Measured brake speed midpoints: {'0.5': [2.5, 7.5, 12.5, 17.5, 22.5, 27.5, 32.5, 37.5, 42.5, 47.5, 52.5],
                                 '1.0': [2.5, 7.5, 12.5, 17.5, 22.5, 27.5, 32.5, 37.5, 42.5, 47.5, 52.5]}
SAVED: src\pursuer_racing\vehicle_dynamics.json
No unmeasured performance has been estimated or extrapolated.
```
The braking model is a **local successfully generated snapshot**, not yet externally verified by reading its contents.

## E. High-speed failures — DO NOT USE AS BRAKING DATA
| Test | Target | Last/max speed | Abort | Brake samples |
|---|---:|---:|---|---|
| `brake_test_highspeed` 65 m/s, brake 1.0 | 65 | 58.51 m/s | tracking error **3.07 m > 3.00 m** | none, distance 0.0 m |
| `brake_test_highspeed` 65 m/s, brake 0.5 | 65 | 58.51 m/s | same **3.07 m** | none, distance 0.0 m |
| `brake_test_straight` 65 m/s, brake 1.0 | 65 | 55.89 m/s | tracking error **3.14 m > 3.00 m**; **0** ticks within 2 m/s target; **0 m** qualifying straight; required **324 m** | none, distance 0.0 m |

Last file shown: `tools/vehicle_profile/data/brake_20261116_182603_v65_b100.csv`. Its raw contents have not been reviewed here. A target of 90 m/s is an *aspiration*, not an established physically achievable speed.

## Next experiment gate
First diagnose local `brake_test_straight.py` and its pre-abort CSV: waypoint, speed, lateral error, steer, heading, curvature and available clear distance. Only then design a viable 65 m/s braking event with sufficient stopping corridor; repeat and keep measurement only if brake onset and response are confirmed. Update `vehicle_dynamics.json` only after successfully completed tests. For each next run, store command, config, new CSV, status and observed usable coverage here.
