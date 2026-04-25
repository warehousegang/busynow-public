# BusyNow

BusyNow is a crowd visibility app for discovering nearby places, checking current crowd signals, and submitting one-tap updates without creating an account.

Live app:

- Frontend: [https://busynow.app](https://busynow.app)

## What BusyNow Does

- Finds nearby places around a user’s location
- Lets users search for a place and load nearby results
- Shows lightweight crowd signals like `empty`, `moderate`, and `busy`
- Accepts fast crowd check-ins without login friction
- Uses cloud infrastructure on AWS with edge security and deployment automation

## Why This Project Exists

BusyNow is both:

- a real product I want to ship publicly
- a hands-on platform engineering and SRE portfolio project

The app itself is intentionally lightweight. The deeper value is in how it is operated:

- infrastructure as code
- secure edge routing
- containerized backend delivery
- static frontend deployment
- GitHub Actions automation
- rollback planning
- cost and abuse controls around third-party APIs

## Platform Highlights

- Frontend hosted on S3 + CloudFront
- Backend running on ECS Fargate behind an ALB
- Infrastructure managed with Terraform
- GitHub Actions for CI/CD
- AWS OIDC for GitHub-to-AWS authentication
- WAF and edge controls to reduce abusive traffic
- Secrets Manager for sensitive runtime configuration

## Current Focus

The next phase of BusyNow is less about adding app features and more about improving operational maturity:

- observability
- service-level objectives
- safer deployment workflows
- rollback drills
- cost visibility
- incident runbooks

See the public roadmap:

- [Platform Roadmap](platform-roadmap.md)

## Public Notes

The implementation repository may remain private while this public documentation stays shareable. That lets me discuss:

- product direction
- system architecture
- reliability strategy
- deployment design
- lessons learned

without exposing the source code itself.
