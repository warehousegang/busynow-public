# BusyNow Runbook Coverage Index

BusyNow's detailed runbooks remain internal, but the public documentation can still describe the current operating coverage.

## Current Internal Coverage

Release and recovery coverage includes:

- frontend deploy
- frontend rollback
- EKS backend deploy
- EKS validation drills
- GitOps and Argo CD drift
- Karpenter capacity validation

Incident and drift coverage includes:

- Google Places upstream degradation
- runtime secret or environment mismatch
- edge protection drift
- persistence backend unavailable
- frontend map-provider degradation
- DNS or TLS drift

## Why This Matters Publicly

- the public app is served from the production stack, with protected backend traffic running on EKS
- development infrastructure is ephemeral and is not part of the live request path
- the presence of runbook coverage is part of the current reliability story even when the step-by-step internal instructions stay private
- release, recovery, GitOps drift, and capacity procedures are documented against the current runtime

## Current Gaps

- repeatable post-deploy verification for protected API reachability still needs to become routine
- recurring cost review and budget response procedures still need more repetition
- rollback and failure drills need a regular cadence
- externalized secret delivery and rotation procedures need broader coverage

## Related Documents

- [Operating BusyNow](operating-busynow.md)
- [Reliability Controls](reliability-controls.md)
- [Platform Engineering Roadmap](platform-roadmap.md)
