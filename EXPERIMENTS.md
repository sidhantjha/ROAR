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

## F. Lateral / oscillation diagnosis (user-shared console output, 2026-11-16)
- Cross-run matched section WP493–506: failed test 21 ticks vs completed race 61 ticks. Test median speed ~29.62 m/s, steering ~-0.13, tracking error ~0.94 m; race median speed ~29.74 m/s, steering ~-0.13, error ~0.99 m. Both peak near ~3 m error. The failed test aborts on 3 m guard; successful race continued. Low normalized steering command (~0.145 max) rules out *command clipping*, not steering-actuator speed limits or lateral tire saturation.
- Separate completed instrumented race `runs/pursuer_20261116_151919.csv`: WP493–506, 63 telemetry ticks, median speed 29.63 m/s, lateral acceleration median 16.79 and max 24.02 m/s²; |speed×yaw rate| median 22.68 and max 27.75 m/s²; maximum sideslip proxy 11.66°, lateral velocity max 5.39 m/s; error max 3.00 m. In this corner, **0 steering sign reversals** and **0 lateral-velocity sign reversals** reported. Thus no evidence of repeated steering oscillation in this segment, but not a global stability proof.
- Across the race for speeds >=55 m/s: 1,100 ticks, speed median 57.75 m/s and max 65.35; median |steer| 0.01 (max 0.04), median lateral accel 2.81 m/s² (max 16.30), median sideslip proxy 1.12° (max 8.01); error max 2.77 m. These are mainly low-curvature observations; cannot infer maximum grip at those speeds.
- All are from a user-run diagnostic script, not independently inspected raw CSV here. Derived sideslip is `atan2(v_lateral,v_forward)` proxy; reported inertial acceleration projection and `v*yaw_rate` are different quantities in transients. No calibrated tire force limit yet. A separate observational script `roar_lateral_model.py` was provided in chat for future local execution; its output is not yet an experiment result.

## G. Observational lateral-bin report (user executed, 2026-11-16)
- Command: `python roar_lateral_model.py`, reading `runs/pursuer_20261116_151919.csv`. Local output: `tools/vehicle_profile/data/lateral_observed.json`; the data and script were **not committed to GitHub**.
- Usable telemetry ticks: **8,348**; `clean` under analytical filters (|tracking error| ≤ 1 m, |sideslip proxy| ≤ 5°, no collision): **6,369**. This is a selection of stable/low-error observations, **not** a measurement of maximum tire force or grip.
- Candidate trouble sector WP704–715: error p90 peaks **5.12 m** near WP709 (n=4), sideslip proxy p90 up to **17.15°** near WP710 (n=5); observed lateral acceleration p90 ~**25.97 m/s²** near WP708–709. Median speeds roughly 30–34 m/s. Few per-waypoint samples; do not claim a calibrated grip limit.
- Speed bin 65–70 m/s: 43 total/43 clean, lateral acceleration clean p90 **5.25 m/s²**, all slip p90 **2.09°**, all error p90 **0.76 m**. This bin covers mostly low-curvature driving, not a high-speed maximum-grip test.
- Earlier WP493–506 had a 3 m guard-triggered failure in a special brake-test while the completed race traced almost the same path, indicating a test-safety abort rather than proven steering-command saturation. The new observational report further prioritizes **controlled corner speed sweeps**, especially WP704–715, while postponing 65+ m/s braking characterization.
- **Next evidence gate:** vary commanded speed on the same corner/line using isolated controller configuration, record entry speed, steering, sideslip, lateral acceleration, tracking error, and race completion; only then estimate a conservative measured envelope. Preserve 417.30 s baseline.

## H. Offline corner-radius map (user executed 2026-11-17)
- User created and ran `tools/section_tuning/analyze_track.py` (local only), reading `tools/mincurv/data/monza_track.npz`, original centerline, using five-waypoint discrete curvature stride. Printed **2,775** waypoints; saved local `tools/section_tuning/monza_track_map.png` showing boundaries, colored curvature and waypoint labels.
- Eight highest distinct local curvature candidates (center index -> estimated discrete turning radius): WP2751 18.3m, WP1371 18.6m, WP874 30.8m, WP1467 30.9m, WP509 32.8m, WP1963 47.4m, WP710 49.1m, WP2073 57.8m. Candidate separation 60 waypoint indices; this is not a complete automatic section segmentation or a measured vehicle turning-radius limit.
- Historic failures close to WP509 (~512), WP1963 (~1954 with original 50% un-repaired line), and WP710 (previous large telemetry error/collision) suggest priority areas for analysis; correlation does not establish grip saturation. Terminal collision output in user screenshot preceded offline analysis and came from `competition_runner.py`, not the offline script.
- **Next:** compute/smooth curvature threshold regions and connect wrap-around section at WP2751 and WP0, inspect map, choose track section boundaries, then create a new manually coded centerline-only Pure Pursuit copy with real section limits; do not alter protected 417.30s controller.
