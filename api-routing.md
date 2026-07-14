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

## Why Explicit API Behaviors Exist

CloudFront only forwards requests to the live backend when a path behavior matches first. Without explicit API behaviors, unknown paths fall through to the frontend origin and return frontend HTML instead of backend JSON.

API behaviors must be declared before the frontend default behavior so CloudFront sends those requests to the ALB instead of the SPA fallback path.

## Why `x-internal-key` Matters

CloudFront is the supported public path to the backend. Network controls prevent direct internet traffic from reaching the protected ALB request path, and ALB listener rules provide a second routing boundary for traffic that reaches the listener.

- requests only forward to the backend target group when both conditions match
- the request path is on the approved API allowlist
- the `x-internal-key` header matches the accepted internal key set

This preserves the current security boundary:

- CloudFront can reach the API origin
- direct ALB access is not a supported public path and is blocked by the current security posture
- requests without the expected internal header do not reach the Kubernetes backend service
- frontend routing stays separate from backend routing

## Related Documents

- [Current Live Architecture](architecture.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
