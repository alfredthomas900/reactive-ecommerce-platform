# Product Service

Reactive Product Catalog Microservice.

## Port
8081

## Database
MongoDB
Database: productdb
Collection: products

## Responsibilities
- Create product
- Fetch product
- Manage stock
- Acts as catalog service

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB
- Gradle (Groovy DSL)

## Run

./gradlew bootRun

## Example APIs

Create Product
POST /products

Get All Products
GET /products

Get Product By ID
GET /products/{id}
