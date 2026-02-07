# 🛒 Reactive E-Commerce Platform

An enterprise-style **Reactive E-Commerce Platform** built using:

* Spring Boot 3
* Spring WebFlux (fully non-blocking, backpressure-aware)
* Reactive MongoDB
* Redis
* Gradle (Groovy DSL)

The platform is modeled as a set of **independent core domain services** and **orchestration services** with:

* Strictly enforced service boundaries
* Clear ownership of durable vs ephemeral state
* Reactive end-to-end request processing

This document is structured as a lightweight **Architecture Design Document (ADD)** combining C4 views, strict dependency rules, and architectural decisions.

---

# 🏗️ 1️⃣ Architecture Overview

## C1 — System Context

```
                ┌─────────────────────────────┐
                │       Customer (Web/App)    │
                └───────────────┬─────────────┘
                                │
                                ▼
                      ┌──────────────────┐
                      │   API Gateway    │
                      └────────┬─────────┘
                               │
                               ▼
                ┌─────────────────────────────┐
                │  Reactive E-Commerce System │
                └───────────────┬─────────────┘
                                │
                                ▼
                   ┌────────────────────────┐
                   │ External Payment       │
                   │ Gateway (Abstracted)   │
                   └────────────────────────┘
```

* The **API Gateway** is the single ingress.
* External payment providers are abstracted behind `core-payment`.
* Clear system boundary separation.

---

## C2 — Container Diagram

```
                          ┌────────────────────┐
                          │     API Gateway    │
                          └─────────┬──────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
   orch-buy-cart              orch-buy-checkout             orch-buy-order
        │                           │                           │
        ▼                           ▼                           ▼

   core-product                core-cart                    core-cart
   core-cart                   core-payment                 core-payment
   core-inventory              core-checkout                core-inventory
   orch-price                  orch-price                   core-checkout
   Redis (cart cache)                                      Redis (order cache)

        ───────────────────────── CORE LAYER ─────────────────────────

   core-product   ──► MongoDB (productdb)
   core-cart      ──► MongoDB (cartdb)
   core-inventory ──► MongoDB (inventorydb)
   core-payment   ──► MongoDB (paymentdb)
   core-checkout  ──► Redis (checkout sessions)
```

---

## C3 — Component Example (orch-buy-order)

```
                    ┌──────────────────────────┐
                    │   OrderController        │
                    └─────────────┬────────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │ OrderOrchestrationService│
                    └─────────────┬────────────┘
                                  │
         ┌──────────────┬──────────────┬──────────────┬──────────────┐
         ▼              ▼              ▼              ▼              ▼
   PricingClient   InventoryClient   PaymentClient   CheckoutClient   CartClient
                                  │
                                  ▼
                          RedisOrderCache
```

---

# 🧠 2️⃣ Architecture Principles

1. **Single Ingress Principle** — All traffic flows through API Gateway.
2. **Strict Directional Dependencies** — Client → Gateway → Orch → Core → Data.
3. **Bounded Context Isolation** — Each core service owns its domain and persistence.
4. **Stateless Orchestration** — Flow coordination only, no durable ownership.
5. **Durable vs Ephemeral Separation** — Mongo = system of record, Redis = experience state.
6. **Idempotent Financial Operations** — Prevent duplicate charges.
7. **Reactive End-to-End** — No blocking calls across services.

---

# 📏 3️⃣ Explicit Dependency Rules

```
ALLOWED:
Client → API Gateway
API Gateway → Orchestration
Orchestration → Core
Core → Data Stores

FORBIDDEN:
Core → Orchestration
Core → Core
Orchestration → MongoDB Drivers
Orchestration → External Payment Gateway
```

These rules prevent distributed monolith patterns and enforce service boundaries.

---

# 🔹 4️⃣ Core Services (Bounded Contexts)

## core-product

* **Database:** productdb (MongoDB)
* Owns product catalog and metadata.
* Exposes product lookup APIs.

## core-cart

* **Database:** cartdb (MongoDB)
* Persists cart state and lifecycle.
* Stores pricing snapshots.

## core-inventory

* **Database:** inventorydb (MongoDB)
* Validates availability.
* Handles stock reservation and release.

## core-payment

* **Database:** paymentdb (MongoDB)
* Handles idempotent payment processing.
* Abstracts external gateway integration.

## core-checkout

* **Backing store:** Redis
* Maintains secure checkout session boundary.
* Implements TTL-based session lifecycle.

---

# 🔹 5️⃣ Orchestration Services

## orch-price

* Stateless pricing composition service.
* Computes subtotal, tax, shipping, discount, final amount.

## orch-buy-cart

* Coordinates cart experience.
* Validates inventory at cart stage.
* Caches cart snapshot in Redis.

## orch-buy-checkout

* Recomputes pricing.
* Initializes secure checkout session.
* Starts payment authorization.

## orch-buy-order

* Validates checkout session.
* Reserves inventory.
* Captures payment.
* Finalizes cart.
* Caches order confirmation.

---

# 🔐 6️⃣ Durable vs Ephemeral State

**Durable (MongoDB):**

* Products
* Carts
* Inventory
* Payments

**Ephemeral (Redis):**

* Cart cache snapshot
* Secure checkout session
* Order confirmation cache

Durable state is authoritative. Redis state is safe to expire and reconstruct.

---

# 🔄 7️⃣ End-to-End Flow (Place Order)

```
Client
   │
   ▼
API Gateway
   │
   ▼
orch-buy-order
   │
   ├──► core-checkout (validate session)
   ├──► orch-price (final pricing)
   ├──► core-inventory (reserve stock)
   ├──► core-payment (capture payment)
   ├──► core-cart (finalize cart)
   └──► Redis (order confirmation cache)
```

Failures are handled via orchestration-level saga compensation logic.

---

# 📂 8️⃣ Project Structure

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
│     └── redis
│
└── README.md
```

---

# 📘 9️⃣ Architectural Decision Records (ADR)

## ADR-001: Reactive Stack Adoption

**Decision:** Use Spring WebFlux and reactive drivers.
**Reason:** Better scalability under high concurrency.

## ADR-002: Core/Orchestration Split

**Decision:** Separate domain ownership from flow coordination.
**Reason:** Prevent distributed monolith.

## ADR-003: No Cross-Core Calls

**Decision:** Core services cannot call other core services.
**Reason:** Maintain bounded context integrity.

## ADR-004: Redis for Ephemeral State

**Decision:** Use Redis only for cache and session.
**Reason:** Preserve Mongo as system of record.

## ADR-005: Idempotent Payment Processing

**Decision:** All payment operations require idempotency keys.
**Reason:** Prevent double charges during retries.

---

# 🎯 Platform Objectives

* Enterprise-grade layered architecture
* Clear service boundary enforcement
* Durable vs ephemeral state separation
* Fully reactive distributed system design
* Secure checkout boundary
* Idempotent financial transaction handling

---

# 🚀 Future Evolution

* Saga compensation framework
* Kafka-based event-driven architecture
* Promotion engine
* Circuit breakers (Resilience4j)
* Distributed tracing (OpenTelemetry)
* Authentication & authorization
* core-order immutable ledger service

---

# 👨‍💻 Author

Alfred Thomas
Senior Java Backend Developer
