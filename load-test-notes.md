# BusyNow Load Test Notes

These notes describe how the current live routing and protection model affects verification and load testing.

## Current Scope

- current load and verification attention is still centered on `/places*`
- `/status*` and `/checkin*` are now part of the protected CloudFront API route family, but route-level load testing for them is still lighter
- future check-in tests should vary `x-bn-user-id` when validating create volume and reuse the same ID when validating cooldown refresh behavior

## Important Verification Behavior

- `/api/places/*` is the better path for CLI and scripted verification traffic
- `/places/*` can be blocked by WAF Bot Control when traffic looks like `curl` or an HTTP library
- that result should be interpreted as protective edge behavior, not as proof of a CloudFront routing defect

## What To Check After Deploys

- `/api/places/*` still reaches the backend through `CloudFront -> EKS-backed ALB -> EKS`
- `/status*` still reaches the backend correctly
- `/checkin*` still reaches the backend correctly
- WAF policy still protects `/places/*` as intended
- Redis-backed dedupe still returns sensible `created` and `updated` actions

## Operational Interpretation

- `403` blocks from WAF on `/places/*` can be expected during CLI-style verification
- `5xx` from the backend path are not expected and should be investigated
- frontend HTML returned for a blocked `/places/*` request can be a side effect of SPA fallback behavior after WAF blocking

## Related Documents

- [Protected API Routing](api-routing.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
