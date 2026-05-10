# BusyNow API Routing

BusyNow exposes the frontend and backend through different layers of the same public domain.

- CloudFront is the only public entry point.
- frontend routes use the default S3 origin and SPA fallback.
- API routes use explicit CloudFront path behaviors that forward to the ALB origin.
- CloudFront injects `x-internal-key` on API origin requests so the ALB can distinguish trusted edge traffic from direct internet traffic.

## Current Live API Behaviors

CloudFront explicitly routes these path families to the backend ALB origin:

- `/places/*`
- `/api/places/*`
- `/checkin*`
- `/api/checkin*`
- `/status*`
- `/api/status*`

All other public routes continue to use the frontend S3 origin and SPA fallback behavior.

## Why Explicit API Behaviors Exist

CloudFront only forwards requests to the backend when a path behavior matches first. Without explicit API behaviors, unmatched requests fall through to the frontend origin and return frontend HTML instead of backend JSON.

The current live behaviors are intentional:

- browser-facing search traffic uses `/places/*`
- CLI and scripted verification traffic is better routed through `/api/places/*`
- `/status*` and `/checkin*` share the same protected backend edge path

## ALB Protection Model

The ALB listeners keep a default fixed `403` response.

Requests only forward to the backend target group when both conditions match:

- the request path is on the approved API allowlist
- the `x-internal-key` header matches the accepted internal key set

This preserves the current security boundary:

- CloudFront can reach the API origin
- direct ALB requests without the internal header are rejected
- direct ALB internet access is not the public path
- frontend routing stays separate from backend routing

## Related Documents

- [Current Live Architecture](architecture.md)
- [WAF API Behavior](waf-api-behavior.md)
- [Redis Check-In Coordination](redis-architecture.md)
