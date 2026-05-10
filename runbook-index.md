# BusyNow Runbook Coverage Index

BusyNow's detailed runbooks remain internal, but the public documentation can still describe the current operating coverage.

## Current Internal Coverage

Release and recovery coverage includes:

- frontend deploy
- frontend rollback
- backend deploy
- backend rollback

Incident and drift coverage includes:

- Google Places upstream degradation
- ECS unhealthy after deploy
- runtime secret or environment mismatch
- edge protection drift
- persistence backend unavailable

## Why This Matters Publicly

- the public app is still served from the live `dev` stack
- the system should be described as it actually operates today, not as a future-state production split that is not yet live
- the presence of runbook coverage is part of the current reliability story even when the step-by-step internal instructions stay private

## Current Gaps

- repeatable post-deploy verification for protected API reachability still needs to become routine
- recurring cost review and budget response procedures still need more repetition
- the future `prod` environment will need its own fully separated operating workflow once it becomes live

## Related Documents

- [Operating BusyNow](operating-busynow.md)
- [Reliability Controls](reliability-controls.md)
- [Platform Engineering Roadmap](platform-roadmap.md)
