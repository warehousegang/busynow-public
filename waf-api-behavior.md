# BusyNow WAF API Behavior

This note documents the current live behavior observed during verification as of May 10, 2026.

## Current Behavior

- `/places/*` is protected by frontend WAF Bot Control
- `/api/places/*` is the better path for CLI and scripted verification traffic that should still reach the backend through CloudFront
- `/checkin*` and `/status*` route correctly through `CloudFront -> ALB -> ECS`
- direct ALB access is blocked before listener evaluation, so unauthenticated internet traffic does not reach the ALB routing layer

## Important Observation

Requests to `/places/*` made with `curl` or other HTTP-library style user agents may be blocked by WAF before they reach the backend.

Observed labels included:

- `awswaf:managed:aws:bot-control:bot:name:curl`
- `awswaf:managed:aws:bot-control:bot:category:http_library`

Because CloudFront also has SPA fallback behavior for the frontend origin, a blocked `/places/*` request can surface as frontend HTML. That can look like a routing problem even though the live routing is correct.

## What This Is Not

This is not a CloudFront routing defect.

The live CloudFront behaviors and ALB listener rules correctly include `/places/*`. The differing outcome between `/places/*` and `/api/places/*` is caused by WAF Bot Control enforcement on `/places/*`.

## Intended Usage Model

- browser and normal client traffic: `/places/*`
- CLI, scripted verification, and operational testing traffic: `/api/places/*`
- backend status and check-in traffic: `/status*` and `/checkin*`

## Why This Is Intentional Right Now

BusyNow previously saw expensive Google Places abuse traffic.

For the current MVP stage, stricter protection on `/places/*` is desirable because that path can trigger paid upstream search traffic. Keeping Bot Control on the public browser-facing places path is currently an intentional abuse and cost control.

## Possible Future Options

If this tradeoff stops fitting the product, likely options include:

- loosen the WAF policy on `/places/*`
- add allowlists for trusted tooling
- move selected Bot Control rules to `COUNT` mode
- add custom user-agent exceptions

## Related Documents

- [Protected API Routing](api-routing.md)
- [Current Live Architecture](architecture.md)
