# ROAR — decisions and rationale

**Reconstructed on 2026-11-16.** Date entries refer to prior project work where recoverable. Mark alternatives and outcomes; don't treat a proposed adjustment as approved implementation.

## 2026-11-13 — Prioritize a minimal, explainable racing baseline
**Decision:** Begin with Pure Pursuit, curvature-based speed planning, telemetry and iterative tests rather than large MPC/RL systems. **Reason:** fixed Monza track, tight development schedule, proven competitive reference approaches, ease of debugging. **Alternative deferred:** MPC, end-to-end RL as initial controller. **Consequence:** direct optimization against measured three-lap times.

## 2026-11-14 to 2026-11-16 — Protect stable `pursuer_sections`; experiment separately
**Decision:** Maintain working ~422.05 s section-tuned reference; place racing-line modifications in `src/pursuer_racing` and `tools/mincurv`. **Reason:** avoid baseline regressions and support fair A/B tests. **Rejected:** overwriting stable controller with untested optimizer outputs. **Status:** active convention.

## 2026-11-15 to 2026-11-16 — Curvature-based corner limits and PI throttle
**Decision:** Increase speed ceiling incrementally, compute local curvature bounds, and use PI throttle/P brake. **Evidence:** time decreased from early >1000 s baselines to ~615.50 s with PI and later ~422 s through section/grip tuning. **Consequence:** throttle saturation and high targets identified as new bottlenecks.

## 2026-11-16 — Disable artificial forward acceleration pass in live planner (SUPERSEDES use of naive 3 m/s² forward-pass)
**Decision:** Keep previously constructed acceleration pass out of actual race control until a model is calibrated. **Evidence:** artificial `MAX_ACCELERATION_MPS2=3.0` forward profile increased race time from ~615.50 to ~694.05 s. **Reason:** too-conservative assumptions plus PI/controller interaction. **Alternative:** backward braking profile used for actual targets. **Caveat:** a later measured acceleration-constrained planner remains a goal.

## 2026-11-16 — Use minimum-curvature line only after driving validation
**Decision:** Retain 20%-blend original minimum-curvature line as fastest measured successful configuration, ~417.30 s; keep 50%-blend repaired path as stable ~419.0 s alternative. **Evidence:** original 50% failed near waypoint ~1954; repair of local peaks and reduced blend mitigated failure. **Rejected:** choose solely by offline curvature RMS; blindly increase global lateral acceleration (34 m/s² crashed near waypoint ~512 absent local limit).

## 2026-11-16 — Instrument before retuning braking
**Decision:** Log 20 Hz world/local velocities, accelerations, yaw rate, commanded control, path error and collisions. **Reason:** successful racing log issued zero brake commands despite high speed targets; gain changes alone are ineffective. **Alternative rejected:** raise `BRAKE_KP` without measuring demand or physical braking. **Status:** implemented locally; new CSV verified.

## 2026-11-16 — Controlled tests and one reusable dynamics JSON
**Decision:** Measure full-throttle acceleration by speed bins, 50%/100% braking by speed, keep raw CSVs and generate `src/pursuer_racing/vehicle_dynamics.json`. **Reason:** single source for later forward/backward planner, transparent speed coverage and provenance. **Evidence:** 8 completed 20–55 m/s brake runs + 13 acceleration bins; builder printed successful local save, skipped two aborted tests. **Rejected:** derive maximum braking from normal race where brake=0; extrapolate unmeasured high-speed bins.

## 2026-11-16 — High-speed validity gate before lateral model
**Decision:** Extend braking toward 65, 70 and optionally 80–90 m/s **only if those speeds are actually reachable with enough track to brake**. **Reason:** race reached ~65.5 m/s while initial tests covered only 20–55 m/s. **Status:** in progress and blocked at 65.

## 2026-11-16 — Keep track-error guard; diagnose steering rather than increase gain blindly
**Decision:** Do not simply relax 3 m error abort, increase steering command strength, or rerun at 70–90 after aborted 65. **Evidence:** 65 target aborted at 58.51 m/s / 3.07 m error; straight-aware variant at 55.89 m/s / 3.14 m error and no brake samples. **Alternative to test:** characterize track curvature and available braking corridor; compare high-speed racing-state error/yaw/steer; redesign test entry opportunity. **Consequence:** next exact action is diagnostic review, not steering retuning.

## 2026-11-16 — Canonical project continuity
**Decision:** Maintain seven root Markdown documents with ROADMAP + ACTIVE_WORK and HANDOFF for restart; checkpoint on substantive progress, no invented background timer. **Reason:** accumulated single-chat history was difficult to transfer. **Persistence:** connected `sidhantjha/ROAR` main (public), with local snapshots optionally downloadable. **Boundary:** documents record user-shared local test output; remote code still starter version and cannot be mistaken for fully synced current code.

## 2026-11-16 — Reprioritize lateral model before high-speed braking extension
**Decision:** Defer further 65+ m/s brake trials; fit/validate lateral and combined-grip operating envelope first, then develop an isolated optimized velocity profile using existing 20–55 m/s longitudinal measurements. Return to high-speed braking after geometry/controller diagnosis. **Evidence:** failed 65 m/s test stopped at WP506 with ~3.14 m tracking error before braking; in a successful race, the same corner showed nearly identical speed, steer and lateral-error trajectory. At WP493–506 the successful telemetry showed increasing sideslip proxy and lateral acceleration with no steering sign flips; this does **not** prove physical tire saturation. **Reason:** get a safer, testable planner and race-time comparison without treating aborted runs as coefficients. **Status:** elected, first offline model report prepared locally in chat, not run or committed.
