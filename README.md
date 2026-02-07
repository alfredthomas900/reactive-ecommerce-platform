# 🛒 Reactive E-Commerce Microservices Platform

A production-style **Reactive Microservices-based E-Commerce Backend System** built using **Spring Boot 3, Spring WebFlux, Reactive MongoDB, and Gradle (Groovy DSL)**.

This project demonstrates:

- Reactive programming using Project Reactor
- Microservices architecture
- Service-to-service communication using WebClient
- Database per service design
- Idempotent payment processing
- Clean orchestration flow
- Saga-ready distributed transaction design (Phase 2)

---

# 🏗️ Architecture Overview

Current Phase (REST Orchestration Based)

                ┌──────────────────┐
                │  Product Service │ (8081)
                └─────────┬────────┘
                          │
                ┌─────────▼────────┐
                │   Cart Service   │ (8082)
                └─────────┬────────┘
                          │
                ┌─────────▼────────┐
                │  Order Service   │ (8083)
                └─────────┬────────┘
                          │
                ┌─────────▼────────┐
                │ Payment Service  │ (8084)
                └──────────────────┘

MongoDB (Single Instance)
mongodb://localhost:27017

- productdb
- cartdb
- orderdb
- paymentdb

Each microservice owns its database.

---

# 🚀 Tech Stack

- Java 17
- Spring Boot 3
- Spring WebFlux (Reactive)
- Reactive MongoDB
- Gradle (Groovy DSL)
- Project Reactor (Mono / Flux)
- WebClient (Inter-service communication)
- Docker (Planned)
- Kafka (Planned – Saga Phase)

---

# 📦 Services

## 1️⃣ Product Service (Port 8081)

**Database:** `productdb`  
**Collection:** `products`

### Responsibilities:
- Create product
- Fetch product
- Manage stock (basic)
- Acts as catalog service

Reactive CRUD using `ReactiveMongoRepository`.

---

## 2️⃣ Cart Service (Port 8082)

**Database:** `cartdb`  
**Collection:** `carts`

### Responsibilities:
- Create cart
- Add items to cart
- Fetch cart
- Calculate total
- Calls Product Service using WebClient

Important design decision:
- Product price snapshot is stored in cart
- Prevents price inconsistency later

---

## 3️⃣ Order Service (Port 8083)

**Database:** `orderdb`  
**Collection:** `orders`

### Responsibilities:
- Fetch cart
- Process payment
- Create order
- Update order status

Order Status Flow:
- CREATED
- CONFIRMED
- PAYMENT_FAILED

Acts as orchestration layer in Phase 1.

---

## 4️⃣ Payment Service (Port 8084)

**Database:** `paymentdb`  
**Collection:** `payments`

### Responsibilities:
- Process payment (Mock Gateway)
- Idempotency validation
- Store audit record
- Return consistent response

Implements idempotent payment handling.

---

# 🔐 Idempotency (Important Design Feature)

If a payment request is sent multiple times with the same idempotency key:

- Payment is processed only once
- Existing result is returned
- Prevents double charging

This is critical in distributed systems and fintech-grade applications.

---

# 📚 Design Patterns Used

- Microservices Architecture
- Database per Service
- Adapter Pattern (Payment Gateway ready)
- Orchestration Pattern
- Idempotency Handling
- Reactive Programming Model
- Non-blocking IO
- Snapshot Strategy (Cart pricing)

---

# 🧠 Reactive Programming Principles Followed

- No blocking calls
- No `.block()`
- End-to-end non-blocking pipelines
- Mono and Flux composition
- WebClient for async inter-service calls

---

# 🛠️ How To Run

## 1️⃣ Start MongoDB

Using Docker:

docker run -d -p 27017:27017 --name mongo mongo

---

## 2️⃣ Run Each Service

Inside each service directory:

./gradlew bootRun

---

# 🧪 Sample Flow

1. Create Product
2. Create Cart
3. Add Product to Cart
4. Create Order
5. Payment Processed
6. Order Confirmed

---

# 📈 Future Enhancements (Planned Phases)

## Phase 2 – Distributed Transactions
- Inventory Service
- Saga Pattern (Orchestration)
- Compensation logic (Refund + Stock release)

## Phase 3 – Event Driven Architecture
- Kafka Integration
- Choreography Saga
- Order Events

## Phase 4 – Resilience
- Circuit Breaker
- Retry mechanism
- Timeout handling

## Phase 5 – Platform Enhancements
- API Gateway (Spring Cloud Gateway)
- Service Discovery
- Centralized Logging
- Distributed Tracing
- JWT Authentication

---

# 🏛️ Architectural Principles Followed

- Clean service boundaries
- No shared entity models across services
- Reactive-first development
- Evolutionary design (iterative improvement)
- Production-ready thinking

---

# 🎯 Project Goal

This project is built to:

- Deeply understand reactive microservices
- Demonstrate real distributed system thinking
- Implement payment-grade idempotency
- Build Saga-based transactional consistency
- Serve as a production-style portfolio project

---

# 👨‍💻 Author

Alfred Thomas  
Senior Java Backend Developer  

---

