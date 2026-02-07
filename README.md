# 🛒 Reactive E-Commerce Platform

An enterprise-style Reactive E-Commerce Platform built using:

- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB
- Redis
- Gradle (Groovy DSL)

This platform follows strict layered architecture principles separating:

- Core Domain Services (Mongo / Redis ownership)
- Orchestration Layer (Buy Experience)
- Infrastructure Layer

This architecture is intentionally designed to enforce clean service boundaries and prevent cross-layer leakage.

---

# 🏗️ Architecture Overview (Final)

```
                           ┌────────────────────┐
                           │     API Gateway    │
                           └─────────┬──────────┘
                                     │
        ┌────────────────────────────┼────────────────────────────┐
        │                            │                            │
   orch-buy-cart              orch-buy-checkout              orch-buy-order
        │                            │                            │
        │                            │                            │
        │                            │                            │
        │                            │                            │
        ▼                            ▼                            ▼

   core-cart                    core-cart                    core-cart
   core-product                 orch-price                   orch-price
   core-inventory               core-payment                 core-inventory
   orch-price                   core-checkout                core-payment
   Redis (cart cache)                                        core-checkout
                                                             Redis (order cache)
```

---

# 📂 Project Structure

```
reactive-commerce-platform/
│
├── core/
│     ├── core-product        (Mongo)
│     ├── core-cart           (Mongo)
│     ├── core-inventory      (Mongo)
│     ├── core-payment        (Mongo + Gateway)
│     └── core-checkout       (Redis session)
│
├── orch/
│     └── orch-buy/
│           ├── orch-buy-cart
│           ├── orch-buy-checkout
│           ├── orch-buy-order
│           └── orch-price
│
├── infra/
│     ├── api-gateway
│     ├── docker-compose.yml
│     └── redis (with Redis Insight)
│
└── README.md
```

---

# 🧠 Architectural Rules (Strictly Enforced)

## Core Layer

Core services:

- Own MongoDB or Redis connections
- Own business logic
- Own third-party integrations
- Never call other core services
- Never call orchestration services

Each core service is independent.

---

## Orchestration Layer

Orchestration services:

- Do NOT connect to MongoDB
- Do NOT integrate directly with payment gateways
- Do NOT own domain persistence
- Coordinate flows across core services
- May use Redis for experience-level caching

Dependency direction:

Orch → Core  
Never Core → Core  
Never Core → Orch

---

# 🔹 Core Services

## core-product
Database: `productdb`

Responsibilities:
- Product catalog management
- Product metadata retrieval

---

## core-cart
Database: `cartdb`

Responsibilities:
- Persist cart state
- Maintain cart lifecycle
- Store pricing snapshot

---

## core-inventory
Database: `inventorydb`

Responsibilities:
- Validate availability
- Reserve stock
- Release stock

---

## core-payment
Database: `paymentdb`

Responsibilities:
- Idempotent payment processing
- External payment gateway integration
- Transaction audit storage

Idempotency ensures:
- Same request processed once
- Duplicate charges prevented

---

## core-checkout
Redis-backed secure checkout session.

Responsibilities:
- Store shipping details
- Store delivery mode
- Store payment selection
- Store pricing summary snapshot
- Prevent checkout tampering
- Temporary session lifecycle

This service forms the **Secure Checkout Boundary**.

---

# 🔹 Orchestration Services

## orch-price (Stateless)

Responsibilities:
- Compute pricing breakdown
- Calculate item total
- Apply shipping fee
- Apply tax (mock initially)
- Apply discount (initially zero)
- Return structured pricing response

Future:
- May evolve into core-price if pricing complexity grows.

---

## orch-buy-cart

Calls:
- core-cart
- core-product
- core-inventory
- orch-price
- Redis (cart cache)

Does NOT call:
- core-payment
- core-checkout

Purpose:
- Cart experience orchestration
- Inventory validation at cart stage
- Pricing snapshot caching

---

## orch-buy-checkout

Calls:
- core-cart
- orch-price
- core-payment
- core-checkout

Does NOT call:
- core-inventory

Purpose:
- Secure checkout session creation
- Repricing before checkout
- Payment initiation

---

## orch-buy-order

Calls:
- core-cart
- orch-price
- core-inventory
- core-payment
- core-checkout
- Redis (order confirmation cache)

Purpose:
- Validate secure session
- Reserve inventory
- Trigger payment
- Convert cart to order
- Cache order confirmation snapshot

---

# 🔐 Durable vs Ephemeral State

## Durable (Mongo – Core Layer)
- Products
- Cart
- Inventory
- Payments

## Ephemeral (Redis – Experience Layer)
- Cart cache snapshot
- Secure checkout session
- Order confirmation cache

---

# 🔄 Flow Summary

## 🛒 Add to Cart
API → orch-buy-cart  
→ core-product  
→ orch-price  
→ core-inventory  
→ core-cart  
→ Redis cache

---

## 🔐 Checkout
API → orch-buy-checkout  
→ core-cart  
→ orch-price  
→ core-payment  
→ core-checkout (Redis session)

---

## 💳 Place Order
API → orch-buy-order  
→ core-checkout  
→ orch-price  
→ core-inventory (reserve)  
→ core-payment  
→ core-cart  
→ Redis (order cache)

---

# 🎯 Platform Objectives

This platform demonstrates:

- Enterprise-grade layered architecture
- Strict service boundary enforcement
- Clear durable vs ephemeral state separation
- Reactive distributed system design
- Secure checkout boundary modeling
- Idempotent financial transaction handling

---

# 🚀 Future Evolution

- Saga compensation logic
- Event-driven architecture (Kafka)
- Promotion engine
- Circuit breaker & resilience
- Distributed tracing & observability
- Authentication & authorization

---

# 👨‍💻 Author

Alfred Thomas  
Senior Java Backend Developer
