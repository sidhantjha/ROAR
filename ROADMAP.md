# ROAR — overall roadmap

**State date:** 2026-11-16. **Objective:** fastest repeatable, collision-free, competition-compliant three-lap Monza controller. Status labels: COMPLETED, IN PROGRESS, BLOCKED, PLANNED. Prioritize incremental, explainable improvements.

| Phase | Status | Deliverable | Acceptance / gate |
|---|---|---|---|
| 0. Simulator / runnable submission baseline | **COMPLETED** (local runs) | CARLA Monza launch, runnable agent, three-lap timing | Complete three laps, reproduce command and collect logs |
| 1. Stable Pure Pursuit + curvature speed control | **COMPLETED** (local runs) | Reference `pursuer_sections` and active `pursuer_racing` experiment branch | Successful ~422 s reference and 417.30 s best known completed race; preserve both |
| 2. Offline racing-line optimization | **COMPLETED for first iteration; IN PROGRESS overall** | Exported track, original + repaired min-curvature line, validated blends | 20% original successful 417.30 s; 50% repaired successful ~419.0 s. Optimize further only with collision-free lap gains |
| 3. Measured longitudinal dynamics | **IN PROGRESS** | Acceleration and braking speed bins, unified `vehicle_dynamics.json` | 8 completed 20–55 m/s tests already incorporated; **blocked at 65+ m/s until safe straight-test opportunity**; reject unsupported extrapolation |
| 4. Lateral/combined-grip dynamics | **PLANNED** (depends on phase 3 tests and reliable instrumentation) | `a_y(v,curvature,steer)` envelope, understeer/saturation check, braking-in-turn limits; update same JSON | Repeated stable tests in representative corners and 20–65+ m/s feasible range, with collision/track-margin guardrails and conservative validated bound |
| 5. Forward/backward velocity optimization | **PLANNED** (depends on calibrated dynamics) | Corner speed ceilings from geometry + grip; backward braking feasibility, forward acceleration feasibility, usable throttle/brake requests | Compare generated target speeds against actual; profile respects measured envelope and yields actual braking where necessary without crashes |
| 6. Controller/line co-tuning and sector analysis | **PLANNED** | Experiment matrix for lookahead, line blend, per-section grip and controller tuning | Consistent three-lap improvements vs 417.30 s, not only offline curvature or single-sector artifacts; measure failures |
| 7. Robustness, packaging, final submission | **PLANNED** | Rule-compliant clean `submission.py`, required assets, reproducible run/test instructions | No unauthorized starter edits, correct asset paths, no debug/test-only side effects, independent clean-launch 3-lap tests, repeatability |
| 8. Deadline readiness / submission | **PLANNED** | Optional official test and final hand-in | Reconfirm prior conversation's reported Dec 4 optional test and Dec 11 23:59 PST final deadline from official current rules; meet required package format |

## Immediate dependency chain
1. Preserve the completed 20–55 m/s model and logs (**done locally**).
2. Diagnose 65 m/s test's tracking / straight-corridor failure; **do not** disable guardrails or blindly increase steering gain (**next**).
3. Redesign/validate high-speed test so acceleration and braking occur in safe, sufficient geometry; collect **65**, then **70**, then **80–90 m/s only if reachable**, with coverage labels.
4. Rebuild unified longitudinal JSON, verify consistency/sample quality and no extrapolation; document coverage separately for 0.5 and 1.0 brake.
5. Map lateral acceleration, steering response and combined-force limitation; update same JSON with provenance and safe bounds.
6. Implement and validate forward/backward planner in an isolated controller variant; run three laps and compare with baseline.
7. Improve corner sectors, tune guardrails and repeat trials; perform final rules audit and packaging.

## Engineering principles and milestone rules
- A phase is COMPLETED only when code is run and validated, not merely generated or syntax-checked.
- A test aborted before brake application supplies *no* braking coefficient.
- The geometry/straight test must confirm enough room for both **acceleration** and **braking**, with track boundaries and curvature checked; nominal lane width alone is not sufficient.
- Retain raw CSVs, input configs, script revision, simulator version and reproducible commands for every measured model value.
- Keep a proven controller rollback and best baseline. Do not trade track completion for a faster isolated sector.
- Global max speed target 80 m/s in config is a *request*, not proof the car can attain it. 70–90 m/s model coverage is conditional on actual reachability.
