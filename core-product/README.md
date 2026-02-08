# 📦 core-product

Reactive Product Catalog Microservice.

## Port
8081

## Database
MongoDB  
Database: `productdb`  
Collection: `products`

## Responsibilities
- Store product catalog
- Fetch product details
- Provide pricing metadata
- Support bulk product lookup

## Design Decisions
- Product is source of truth for catalog metadata
- No awareness of cart, checkout, or payment
- Optimized for read-heavy workloads

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB

## Example APIs

Create Product  
POST /products

Get Product  
GET /products/{productId}

Bulk Fetch  
GET /products?ids={id1,id2,id3}
