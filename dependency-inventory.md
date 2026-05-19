# BusyNow Dependency Inventory

Last updated: 2026-05-19

This document summarizes the main live runtime and infrastructure dependencies that shape BusyNow's current public architecture.

## Core Public Path

- `AWS CloudFront`
  - the only public entry point
  - serves the frontend and routes explicit API path families to the backend origin

- `AWS WAF`
  - protects the browser-facing search path
  - intentionally treats `/places/*` more strictly than `/api/places/*`

- `AWS S3`
  - serves the frontend origin and SPA fallback path

- `AWS Application Load Balancer (ALB)`
  - sits in front of the live EKS backend ingress path
  - forwards only approved API path families from CloudFront

- `AWS EKS`
  - runs the live backend application
  - the backend migration from ECS to EKS is now part of the live production story

## Coordination, Persistence, And Secrets

- `Redis / ElastiCache`
  - used for 30 minute TTL-backed check-in dedupe coordination
  - optional by configuration, but recommended for multi-pod consistency
  - not a general cache or source of truth

- `Postgres`
  - optional persistence backend when `DATABASE_URL` is configured

- `Supabase`
  - optional persistence backend when `SUPABASE_URL` and `SUPABASE_KEY` are configured

- `AWS Secrets Manager`
  - remains part of the wider infrastructure secret story

- `Kubernetes Secret` objects
  - currently provide the live backend runtime configuration inside EKS

## External Product Dependency

- `Google Places API`
  - used only for the nearby search path
  - the highest abuse and variable-cost risk in the current request flow
  - one reason `/places/*` is protected more aggressively than the rest of the app

## Observability Dependency

- `CloudWatch Logs / Insights`
  - stores structured `[USAGE_EVENT]` and `[CHECKIN_EVENT]` logs
  - supports route-level operational queries without depending on noisy background endpoints

## Live Dependency Notes

- CloudFront, the ALB, and Amazon EKS sit on the main public serving path and have the widest blast radius
- Google Places is important, but only nearby search depends on it directly
- Postgres and Supabase are environment-dependent persistence options, so neither should be described as the only required backend by default
- Redis is intentionally scoped to ephemeral coordination so its failure mode is narrower than the core read path
- the legacy ECS service and old `busynow-alb` remain relevant only as rollback assets during the current soak period, not as part of the live public path

## Related Documents

- [Current Live Architecture](architecture.md)
- [Redis Check-In Coordination](redis-architecture.md)
- [WAF API Behavior](waf-api-behavior.md)
