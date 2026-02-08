# 📦 core-inventory

Reactive Inventory Management Microservice

## Port
8083

## Database
MongoDB  
Database: `inventorydb`  
Collection: `inventory`

## Responsibilities
- Validate stock availability
- Reserve stock
- Release stock on failure
- Maintain stock levels

## Design Decisions
- Reservation-based stock handling
- Inventory validation at order stage
- No awareness of payment logic

## Tech Stack
- Spring Boot 3
- Spring WebFlux
- Reactive MongoDB

## Example APIs

Validate Stock  
POST /inventory/validate

Reserve Stock  
POST /inventory/reserve

Release Stock  
POST /inventory/release

# 🔒 Core Layer Principles

- Each service owns its persistence
- No cross-core service calls
- No orchestration logic inside core services
- Fully reactive and non-blocking
- Independently deployable and scalable