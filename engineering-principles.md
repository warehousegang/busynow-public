# BusyNow Engineering Principles And Tradeoffs

This document describes the main engineering choices behind BusyNow and the tradeoffs that shaped them.

## Core Principles

### 1. Reversibility Over Heroics

Deployments should be easy to understand, easy to verify, and easy to undo.

In practice, that means:

- explicit image tags instead of relying on `latest`
- separate frontend and backend deploy paths
- rollback workflows treated as a normal part of delivery

### 2. Protect Expensive Dependencies With Bulkheads

Google Places improves the product, but it also creates cost and abuse risk. That makes edge protection part of the application design, not a separate cleanup task.

In practice, that means:

- CloudFront in front of the application
- origin protection around the backend path
- WAF rules and rate limiting
- cached or degraded behavior before uncontrolled upstream spend
- attention to traffic patterns that can trigger unnecessary paid API usage

### 3. Favor Simple Runtime Paths

The runtime should stay simple enough that failure modes are understandable. Additional complexity is only useful when it makes the system safer or easier to operate.

In practice, that means:

- static frontend hosting
- a single ECS service instead of heavier orchestration
- rolling deployments first, with more advanced rollout patterns added only when justified

### 4. Match Controls To Real Risks

BusyNow does not need every possible control at once, but it does need the controls that address the risks it actually has.

Current risks include:

- direct origin abuse
- bot traffic causing paid upstream API calls
- leaked or misused credentials
- accidental release breakage

### 5. Treat Cost As An Engineering Constraint

For a small service, cost problems can be as disruptive as availability problems. Budget awareness is part of operating the system responsibly.

In practice, that means:

- reviewing fixed-cost AWS components
- being intentional about WAF and bot-control settings
- understanding where architecture, not traffic, is driving spend

## Key Tradeoffs

### Simplicity vs Safer Rollouts

Current preference:

- ECS rolling deployments

Why:

- simpler to operate
- fewer moving parts
- easier to reason about at the current size of the service

Future direction:

- blue/green or canary rollout only if the added safety clearly outweighs the added complexity

### Edge Security vs Configuration Overhead

Current preference:

- CloudFront, backend origin protection, and WAF around the expensive API path

Why:

- protects cost-sensitive dependencies
- reduces direct abuse
- keeps the public product usable while tightening the backend path

Tradeoff:

- more infrastructure configuration
- more places where routing or protection can drift if documentation is weak

### Protective Bulkheads vs Platform Simplicity

Current preference:

- a few explicit boundaries around the risky paths instead of a broader platform layer

Why:

- keeps the system understandable
- reduces the blast radius of deploy, edge, and upstream failures
- fits the current size of the service better than a more abstract control plane

Tradeoff:

- some protections are still process-driven rather than fully automated
- the environment split is still incomplete while the public app runs on the live `dev` stack

### Product Scope vs Operational Depth

Current preference:

- keep the product focused
- invest more heavily in reliability, delivery, and runtime controls

Why:

- a smaller service is easier to operate well
- operational quality creates more value than adding loosely maintained product surface area

## Decision Style

BusyNow is intended to evolve through small, reviewable changes.

That means:

- choosing clear systems over impressive ones
- documenting why a control or workflow exists
- delaying complexity until there is a concrete operational reason for it
- preferring tools and patterns that are easy to explain and maintain

## Related Documents

- [Architecture](architecture.md)
- [Reliability Controls: Runbooks And Bulkheads](reliability-controls.md)
- [Implementation Roadmap](platform-roadmap.md)
- [Operating BusyNow](operating-busynow.md)
