# Cart Service

Reactive Cart Management Microservice.

## Port
8082

## Database
MongoDB
Database: cartdb
Collection: carts

## Responsibilities
- Create cart
- Add items to cart
- Fetch cart
- Calculate total
- Calls Product Service via WebClient

## Design Decisions
- Product snapshot stored in cart
- Prevents price inconsistency

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
