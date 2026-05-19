# BusyNow Runbook Coverage Index

BusyNow's detailed runbooks remain internal, but the public documentation can still describe the current operating coverage.

## Current Internal Coverage

Release and recovery coverage includes:

- frontend deploy
- frontend rollback
- EKS backend deploy
- EKS validation drills
- legacy ECS rollback path during the current soak period

Incident and drift coverage includes:

- Google Places upstream degradation
- runtime secret or environment mismatch
- edge protection drift
- persistence backend unavailable

## Why This Matters Publicly

- the public app is still served from the live `dev` stack
- the system should be described as it actually operates today, not as a future-state production split that is not yet live
- the presence of runbook coverage is part of the current reliability story even when the step-by-step internal instructions stay private
- the backend migration from ECS to EKS is part of the real operating story, not just a planned future change

## Current Gaps

- repeatable post-deploy verification for protected API reachability still needs to become routine
- recurring cost review and budget response procedures still need more repetition
- the future `prod` environment will need its own fully separated operating workflow once it becomes live
- the old ECS rollback path still needs a retirement decision once the EKS soak period is complete

## Related Documents

- [Operating BusyNow](operating-busynow.md)
- [Reliability Controls](reliability-controls.md)
- [Platform Engineering Roadmap](platform-roadmap.md)
