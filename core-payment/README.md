# 💳 core-payment

Reactive Payment Processing Microservice

## Port
8084

## Database
MongoDB  
Database: `paymentdb`  
Collection: `payments`

## Responsibilities
- Create payment intent
- Capture payment
- Store transaction audit
- Handle idempotency

## Design Decisions
- Idempotency key required
- Prevent duplicate charges
- Abstract external gateway integration

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB
- WebClient

## Example APIs

Create Payment Intent  
POST /payments/intents

Capture Payment  
POST /payments/capture

Get Payment  
GET /payments/{paymentId}

# 🔒 Core Layer Principles

- Each service owns its persistence
- No cross-core service calls
- No orchestration logic inside core services
- Fully reactive and non-blocking
- Independently deployable and scalable