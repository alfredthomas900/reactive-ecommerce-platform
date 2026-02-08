# 📦 core-order

Reactive Order Management Microservice

## Port
8086

## Database
MongoDB  
Database: `orderdb`  
Collection: `orders`

## Responsibilities
- Create order from confirmed cart
- Persist immutable order snapshot
- Update order status
- Maintain order lifecycle state

## Order Status Flow

- CREATED
- CONFIRMED
- PAYMENT_FAILED
- CANCELLED (future)
- SHIPPED (future)

## Design Decisions
- Order is immutable after confirmation (except status)
- Order stores cart + pricing snapshot
- No direct payment or inventory logic inside service

## Architecture Style
Domain-focused service (Orchestration handled externally)

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB

## Example APIs

Create Order  
POST /orders/{cartId}

Get Order  
GET /orders/{orderId}

Update Status  
PATCH /orders/{orderId}/status