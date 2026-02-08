# 🔐 core-checkout

Reactive Checkout Session Microservice.

## Port
8085

## Database
Redis  
Key Pattern: `checkout:{sessionId}`

## Responsibilities
- Create secure checkout session
- Store shipping details
- Store delivery mode
- Store pricing snapshot
- Enforce TTL

## Design Decisions
- Ephemeral state only
- Redis for fast session retrieval
- Checkout boundary prevents tampering

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive Redis

## Example APIs

Create Checkout Session  
POST /checkout-sessions

Get Checkout Session  
GET /checkout-sessions/{sessionId}

Invalidate Session  
DELETE /checkout-sessions/{sessionId}

# 🔒 Core Layer Principles

- Each service owns its persistence
- No cross-core service calls
- No orchestration logic inside core services
- Fully reactive and non-blocking
- Independently deployable and scalable