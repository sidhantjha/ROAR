# ROAR — project knowledge index and operating guide

**Canonical location:** repository root on `main`, `https://github.com/sidhantjha/ROAR`. This repository is **public**. Do not record credentials, secrets, private personal information, or unreleased confidential material here.

**Scope:** Berkeley ROAR Fall 2026 autonomous racing in CARLA Monza. This file is a navigation/working-conventions guide, not a substitute for source code or test logs.

## Read in this order
1. [HANDOFF.md](HANDOFF.md) — compact restart point and exact next action.
2. [ACTIVE_WORK.md](ACTIVE_WORK.md) — detailed immediate workstream, commands, blocker and pending experiments.
3. [PROJECT_STATE.md](PROJECT_STATE.md) — verified project/system state and boundaries of verification.
4. [ROADMAP.md](ROADMAP.md) — phases, dependencies and acceptance criteria.
5. [EXPERIMENTS.md](EXPERIMENTS.md) — measured evidence, failures and raw-log pointers.
6. [DECISIONS.md](DECISIONS.md) — rationale, rejections and superseded choices.

## Operating conventions
- **Source-of-truth hierarchy:** actual local source + telemetry/test outputs for behavior; these versioned knowledge files for continuity; user-provided terminal output for runs not yet in the repo. Label assumptions and unverified claims. The remote repository currently lacks recent local experimental code: see PROJECT_STATE.
- **Keep two plans current:** ROADMAP = whole project; ACTIVE_WORK = immediate task. Keep HANDOFF aligned.
- **Checkpoint after every meaningful result, decision or change in approach**, and during long active sessions roughly every 2–3 minutes when feasible. There is no autonomous timer between chats. Update only affected sections, avoid duplicate history, verify actual writes.
- **Commands:** `checkpoint` reconciles state; `end chat` or `check out` stops new work, updates all affected files and verifies persistence; `resume project` reads HANDOFF, ACTIVE_WORK and PROJECT_STATE, then resumes the exact next action.
- **Engineering style:** one explainable improvement at a time; user implements/understands concise code; preserve the 417.30 s baseline; keep official `competition_runner.py` unchanged; isolated experiments, objective telemetry, and three-lap comparisons. Never call an aborted run a measured braking result.
- **Safety / rules:** starter `submission.py` says to change only TODO sections. Audit final submission compliance before competition; avoid blindly porting telemetry/test harness modifications.
- **Persistence:** commit updates to this repository only after confirming repo/ref; verify each remote write and check local synchronization. New-chat access to GitHub is not automatic if the connector is unavailable—provide the repo link or attach these files.

**Last knowledge reconstruction:** 2026-11-16. Current immediate next action is in HANDOFF.md.
