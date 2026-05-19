# BusyNow Redis Check-In Coordination

BusyNow uses Redis or ElastiCache as a lightweight shared coordination layer for repeat check-in suppression across EKS pods.

## What Redis Is

- ephemeral distributed coordination
- shared TTL-backed cooldown state across EKS pods
- the source of duplicate check-in suppression for the same place and anonymous browser ID

## What Redis Is Not

- not the source of truth for crowd status
- not a general shared response cache
- not a session or auth layer

## Why Redis Exists

EKS pods do not share process memory. In-memory JavaScript cooldown state would be incorrect because each pod would see a different view of recent user activity. Redis gives BusyNow one shared TTL-backed place to coordinate repeat check-ins.

## Current Key Format

```text
checkin:{place_id}:{user_id}
```

Example:

```text
checkin:starbucks-ballard:550e8400-e29b-41d4-a716-446655440000
```

## Value And TTL

Redis stores JSON values with a 30 minute TTL (`1800` seconds). A representative value looks like:

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "place_id": "starbucks-ballard",
  "status": "busy",
  "timestamp": "2026-05-19T14:30:00.000Z"
}
```

## Current Live Behavior

- the first check-in for the same `place_id + user_id` returns `checkin_action: "created"`
- a repeat check-in during the cooldown rewrites the JSON value, refreshes the TTL, and returns `checkin_action: "updated"`
- duplicate requests during the cooldown do not create another durable check-in
- if Redis is unavailable, the backend logs the problem and continues instead of crashing the API

## Request Flow

1. the browser sends `x-bn-user-id` with the check-in request
2. the backend reads `place_id` and `x-bn-user-id`
3. BusyNow checks Redis for `checkin:{place_id}:{user_id}`
4. if the key is missing, BusyNow creates it with a 30 minute TTL and persists the check-in normally
5. if the key already exists, BusyNow refreshes the TTL, skips the duplicate durable write, and returns an `updated` action

## Related Documents

- [Current Live Architecture](architecture.md)
- [Protected API Routing](api-routing.md)
- [WAF API Behavior](waf-api-behavior.md)
