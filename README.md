# 🛒 Reactive E-Commerce Platform

An enterprise-inspired **Reactive Microservices E-Commerce Platform** built using:

- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB
- Gradle (Groovy DSL)
- Project Reactor

This platform demonstrates layered microservices architecture with clear separation between:

- Core Domain Services
- Orchestration (Experience) Layer
- Infrastructure Layer

Designed for evolutionary growth toward distributed Saga-based transaction management.

---

# 🏗️ Architecture Overview

This system follows a layered architecture:

```
                        ┌────────────────────┐
                        │     API Gateway    │
                        └─────────┬──────────┘
                                  │
     ┌────────────────────────────┼────────────────────────────┐
     │                            │                            │
orch-buy-cart             orch-buy-checkout              orch-buy-order
     │                            │                            │
     └───────────────┬────────────┼───────────────┬────────────┘
                     │            │               │
                orch-price     core-inventory   core-payment
                     │
               core-product
```

---

# 📂 Project Structure

```
reactive-commerce-platform/
│
├── core/
│     ├── core-product
│     ├── core-payment
│     └── core-inventory
│
├── orch/
│     └── orch-buy/
│           ├── orch-buy-cart
│           ├── orch-buy-checkout
│           ├── orch-buy-order
│           └── orch-price
│
├── infra/
│     └── api-gateway
│
└── README.md
```

---

# 🧠 Architectural Philosophy

This platform is built around the following principles:

- Reactive-first design (non-blocking IO)
- Database per service
- Clear separation of orchestration and domain logic
- Centralized pricing computation
- Idempotent payment handling
- Evolution-ready Saga orchestration
- Service boundary discipline (Core never depends on Orch)

---

# 🔹 Core Layer (Domain Ownership)

Core services own business logic and data.

---

## core-product

**Database:** `productdb`

Responsibilities:
- Manage product catalog
- Provide product data for pricing
- Maintain stock metadata (not reservation)

---

## core-inventory

**Database:** `inventorydb`

Responsibilities:
- Reserve stock
- Release stock
- Prevent overselling
- Maintain stock state

---

## core-payment

**Database:** `paymentdb`

Responsibilities:
- Idempotent payment processing
- Transaction audit storage
- Refund capability (future phase)

Key Feature:
- Payment requests with same idempotency key are processed only once.

---

# 🔹 Orchestration Layer (Buy Experience)

Orch services coordinate flows but do not own domain logic.

---

## orch-buy-cart

**Database:** `cartdb`

Responsibilities:
- Create cart
- Add/remove items
- Request pricing from orch-price
- Store pricing snapshot
- Maintain cart lifecycle

---

## orch-buy-checkout

Responsibilities:
- Validate cart
- Request final pricing from orch-price
- Reserve inventory via core-inventory
- Initiate idempotent payment via core-payment
- Coordinate distributed flow
- Trigger order creation

Designed to evolve into Saga Orchestrator.

---

## orch-buy-order

**Database:** `orderdb`

Responsibilities:
- Create order record
- Maintain order state
- Track lifecycle transitions

Order Status Flow:
- CREATED
- CONFIRMED
- PAYMENT_FAILED
- CANCELLED (future)

---

## orch-price

Centralized pricing computation service.

Responsibilities:
- Compute item totals
- Calculate shipping fee
- Apply tax (mock in Phase 1)
- Apply discounts (initially zero)
- Return structured pricing breakdown

Pricing Response Example:

```json
{
  "itemsTotal": 1000,
  "shippingFee": 100,
  "tax": 90,
  "discount": 0,
  "grandTotal": 1190,
  "currency": "INR"
}
```

Future Evolution:
- Promotion engine integration
- Coupon engine
- Loyalty pricing
- External tax provider integration (Vertex/Avalara)
- Dedicated `core-price` service

---

# 🌐 Infrastructure Layer

## API Gateway

Implemented using Spring Cloud Gateway.

Responsibilities:
- Central routing entry point
- Request forwarding to orch services
- Future:
    - JWT validation
    - Rate limiting
    - API versioning
    - Observability hooks

In enterprise deployments, this layer may be replaced by:
- Apigee
- Kong
- AWS API Gateway
- Azure API Management

---

# 🔄 Service Interaction Flow

## Add to Cart

Client  
→ API Gateway  
→ orch-buy-cart  
→ orch-price  
→ core-product

---

## Checkout Flow

Client  
→ API Gateway  
→ orch-buy-checkout

Checkout orchestrates:

1. Pricing calculation (orch-price)
2. Inventory reservation (core-inventory)
3. Payment processing (core-payment)
4. Order creation (orch-buy-order)

---

# 🔐 Idempotency

Payment processing is idempotent.

If the same idempotency key is received multiple times:

- The payment is processed only once.
- The original response is returned.
- Prevents double charging.

Critical for distributed systems and financial reliability.

---

# 📚 Design Patterns Applied

- Microservices Architecture
- Layered Orch + Core Separation
- Database per Service
- Adapter Pattern (Payment Gateway ready)
- Orchestration Pattern
- Idempotency Handling
- Snapshot Strategy (Cart pricing)
- Reactive Composition (Mono / Flux pipelines)

---

# ⚙️ Tech Stack

- Java 17
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB
- Gradle (Groovy DSL)
- Project Reactor
- WebClient
- Spring Cloud Gateway (API layer)

---

# 🛠️ How To Run

## 1️⃣ Start MongoDB

```bash
docker run -d -p 27017:27017 --name mongo mongo
```

---

## 2️⃣ Run Each Service

Inside each service directory:

```bash
./gradlew bootRun
```

---

# 🚀 Future Enhancements

## Phase 2 – Distributed Transactions
- Saga orchestration
- Compensation logic (inventory release + refund)
- Failure recovery handling

## Phase 3 – Event-Driven Architecture
- Kafka integration
- Choreography Saga
- Order domain events

## Phase 4 – Platform Resilience
- Circuit Breaker
- Retry policies
- Timeout management

## Phase 5 – Platform Maturity
- Redis caching
- Distributed tracing
- Centralized logging
- JWT authentication
- Metrics & monitoring

---

# 🎯 Project Objective

This platform demonstrates:

- Enterprise-grade architectural thinking
- Reactive distributed systems design
- Clear domain boundary enforcement
- Evolutionary microservices architecture
- Orchestration and domain separation strategy

---

# 👨‍💻 Author

Alfred Thomas  
Senior Java Backend Developer
