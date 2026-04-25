# BusyNow Engineering Principles And Tradeoffs

This document exists to show how BusyNow is operated and evolved, not just what technologies it uses.

## Design Principles

### 1. Reversibility Over Heroics

Deployments should be easy to understand, easy to verify, and easy to undo.

What that means in practice:

- explicit image tags instead of relying on `latest`
- separate frontend and backend deploy paths
- rollback workflows as part of normal delivery, not emergency improvisation

### 2. Protect Expensive Dependencies Early

Google Places is useful, but it introduces cost and abuse risk. That makes edge protection part of the product design, not just an infrastructure afterthought.

What that means in practice:

- CloudFront in front of the application
- ALB protection using an internal header pattern
- WAF and rate limiting
- deliberate thinking about cost exposure from bot traffic

### 3. Favor Boring Runtime Paths

The runtime should be simple enough that failure domains are easy to reason about. Complexity is acceptable in CI/CD, Terraform, and documentation if it makes production operations safer.

What that means in practice:

- static frontend hosting
- single ECS service instead of premature orchestration complexity
- rolling deployments first, then more advanced rollout patterns only when justified

### 4. Security Should Match The Threat Model

BusyNow does not need every enterprise control on day one, but it does need the controls that address the actual risks it faces.

Current risks that matter:

- direct origin abuse
- bot traffic triggering paid API calls
- leaked or misused credentials
- accidental deploy breakage

### 5. Cost Is A Reliability Concern

For a small service, runaway cost can be just as damaging as downtime. Budget awareness is part of production engineering.

What that means in practice:

- reviewing fixed-cost AWS components
- being intentional about WAF and bot-control settings
- understanding where cloud cost is driven by architecture, not traffic

## Important Tradeoffs

### Simplicity vs Safer Rollouts

Current preference:

- ECS rolling deployments

Reasoning:

- simpler to operate
- fewer moving parts
- easier to reason about early in the project

Future evolution:

- blue/green or canary once safer cutover value outweighs operational complexity

### Edge Security vs Friction

Current preference:

- CloudFront, ALB protections, and WAF around the expensive API path

Reasoning:

- protects cost-sensitive dependencies
- reduces direct abuse
- keeps the public product path usable while tightening the origin path

Tradeoff:

- more configuration complexity
- more places where routing and auth can drift if documentation is weak

### Product Complexity vs Operational Depth

Current preference:

- keep the app itself modest
- invest more in reliability, delivery, and control-plane maturity

Reasoning:

- a small, well-operated service is better platform evidence than a larger, sloppier one

## Why This Matters For Senior And Staff Work

Senior and staff platform work is not about picking the fanciest tools. It is about making good choices under constraints.

BusyNow is meant to demonstrate:

- clear prioritization
- tradeoff awareness
- system evolution over time
- cost and abuse awareness
- operational ownership across delivery, runtime, and edge controls

## What I Would Want Reviewers To Notice

- the architecture is intentionally understandable
- the deployment path is becoming safer and more explicit over time
- the security controls are tied to real abuse and billing pressure
- the roadmap is ordered by operational value, not just technical novelty
- complexity is introduced deliberately, not automatically
