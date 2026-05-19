# BusyNow API Routing

BusyNow exposes the frontend and backend through different layers of the same public domain.

- CloudFront is the only public entry point.
- frontend routes use the default S3 origin and SPA fallback.
- API routes use explicit CloudFront path behaviors that forward to the ALB origin.
- CloudFront injects `x-internal-key` on API origin requests so the ALB can distinguish trusted edge traffic from direct internet traffic.

## Routing Flow

`browser -> CloudFront -> EKS-backed ALB ingress -> Kubernetes Service -> backend pod`

- `/places/*` and `/api/places/*` route to the backend for place lookups.
- `/checkin*` and `/api/checkin*` route to the backend for write traffic.
- `/status*` and `/api/status*` route to the backend for place status reads.
- all other routes continue to use the frontend S3 origin and SPA fallback behavior.
- the old `busynow-alb` still exists during the current soak, but it is not on the live public path.

## Why Explicit API Behaviors Exist

CloudFront only forwards requests to the live backend when a path behavior matches first. Without explicit API behaviors, unknown paths fall through to the frontend origin and return frontend HTML instead of backend JSON.

API behaviors must be declared before the frontend default behavior so CloudFront sends those requests to the ALB instead of the SPA fallback path.

## Why `x-internal-key` Still Matters

The EKS-backed ALB listeners keep a default fixed response for unmatched traffic.

- requests only forward to the backend target group when both conditions match
- the request path is on the approved API allowlist
- the `x-internal-key` header matches the accepted internal key set

This preserves the current security boundary:

- CloudFront can reach the API origin
- direct ALB requests to protected paths without the internal header do not reach the backend
- frontend routing stays separate from backend routing

## Related Documents

- [Current Live Architecture](architecture.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
