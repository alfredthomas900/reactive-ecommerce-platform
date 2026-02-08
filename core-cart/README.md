# 🛒 core-cart

Reactive Cart Management Microservice.

## Port
8082

## Database
MongoDB  
Database: `cartdb`  
Collection: `carts`

## Responsibilities
- Create cart
- Add items to cart
- Fetch cart
- Maintain pricing snapshot
- Calls the downstream service core-product via WebClient

## Design Decisions
- Product snapshot stored in cart
- Prevents price inconsistency
- Cart is system-of-record for cart state

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB
- WebClient

## Example APIs

Create Cart  
POST /carts/{userId}

Add Item  
POST /carts/{cartId}/items?productId={id}&quantity=1

Get Cart  
GET /carts/{cartId}

# 🔒 Core Layer Principles

- Each service owns its persistence
- No cross-core service calls
- No orchestration logic inside core services
- Fully reactive and non-blocking
- Independently deployable and scalable