# BusyNow Platform Roadmap

BusyNow is a small production system designed to showcase practical platform engineering and SRE work, not just application development.

## Positioning

This project is meant to demonstrate:

- production-minded cloud architecture
- infrastructure as code
- secure delivery workflows
- reliability thinking
- cost-aware operations

The app does not need to be huge to be credible. It needs to be well-run.

## Current Architecture

- React frontend served through CloudFront
- Static assets hosted in S3
- Express backend deployed on ECS Fargate
- ALB in front of the API
- Terraform-managed AWS infrastructure
- GitHub Actions for build and deploy automation
- Secrets handled through AWS Secrets Manager

## Why It Is Valuable As A Platform Project

BusyNow already includes real operational concerns:

- edge routing and caching
- bot and abuse prevention
- authenticated CI/CD into AWS with OIDC
- third-party dependency management
- backend rollout and rollback workflows
- separation between frontend and backend delivery

That is enough to tell a strong platform story when paired with good documentation, metrics, and operating practice.

## Phase 1: Production Readiness

Goal:
Make the system understandable and operable by another engineer.

Deliverables:

- architecture diagram
- deployment flow documentation
- environment inventory
- secrets inventory
- dependency map for external services

## Phase 2: Observability

Goal:
Make health, failure, and change visible.

Deliverables:

- structured request logging
- CloudWatch dashboards
- latency, error, and traffic metrics
- alarms for unhealthy targets and elevated 5xx responses
- uptime checks for homepage and health endpoints

## Phase 3: Reliability

Goal:
Operate with explicit service expectations.

Deliverables:

- SLIs for availability and latency
- simple SLOs
- error budget notes
- degraded-mode behavior for upstream dependency failures
- failure mode catalog

## Phase 4: Safer Delivery

Goal:
Make changes repeatable, observable, and reversible.

Deliverables:

- stable frontend deploy workflow
- stable backend deploy workflow
- explicit rollback steps
- release checklist
- post-deploy smoke verification

Stretch options:

- canary rollout
- blue/green rollout
- alarm-based rollback

## Phase 5: Environment Promotion

Goal:
Show that production changes move through environments intentionally.

Deliverables:

- staging environment
- build-once promote-forward artifact policy
- GitHub environment protections
- smoke test gates

## Phase 6: Security And Cost Governance

Goal:
Demonstrate production discipline beyond shipping features.

Deliverables:

- threat model
- least-privilege reviews
- secret rotation process
- monthly cost review
- budget alerting
- WAF tuning and abuse protection notes

## Phase 7: Incident Response

Goal:
Show that the system can be operated under failure.

Deliverables:

- runbooks for common incidents
- incident review template
- rollback drills
- dependency outage exercises

## What This Shows In Interviews

BusyNow becomes more compelling when described as:

"A small cloud service that I built and operated end to end, including infrastructure, deployment, security, edge protection, and reliability improvements."

That framing is stronger than simply presenting it as a frontend/backend app.
