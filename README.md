## Order & Inventory Microservices

A lightweight, event-driven demonstration of two collaborating microservices—Inventory and Payment—coordinated via Redis Streams and backed by FastAPI, Redis-OM, and a React front-end.

---

## 🔍 Project Overview

- **Inventory Service** (`microservices_inventory`)  
  Manages product data (name, price, quantity) in Redis as Hashes. Exposes REST endpoints to create, list, retrieve, and delete products.

- **Payment Service** (`microservices_payment`)  
  Handles order creation and status tracking. When you place an order, it:
  1. Fetches product details from the Inventory API  
  2. Saves a new Order record (`status=pending`) in Redis  
  3. Schedules a 10-second background task to mark the order `completed`  
  4. Emits an `order_completed` event into a Redis Stream

- **Consumers**  
  - **Inventory Consumer** listens for `order_completed` events, decrements product quantities, and—if stock is insufficient—emits a `refund_order` event.  
  - **Payment Consumer** listens for `refund_order` events and marks the corresponding orders `refunded`.

- **React Front-End** (`inventory-frontend`)  
  Provides:
  - A **Products** page to add, view, and delete products  
  - An **Orders** checkout form that previews price (including a 20% fee), submits orders, and polls for status changes.

---

## 🏗️ Why This Architecture & Technology Stack?

1. **FastAPI**  
   - Modern Python web framework with built-in async support, automatic OpenAPI docs, and Pydantic validation.  
   - Ideal for lightweight microservices.

2. **Redis-OM (HashModel + Streams)**  
   - Stores domain objects (`Product` and `Order`) as Redis Hashes for super-fast access.  
   - Uses Redis Streams to decouple services:  
     - *Event-driven* flow—no tight coupling or blocking HTTP calls between Inventory and Payment.  
     - *Durable, replayable* message log: if a consumer is down, it will catch up on restart.

3. **BackgroundTasks**  
   - FastAPI’s simple way to schedule non-blocking work (marking orders completed).  
   - Keeps the API responsive while you perform asynchronous operations.

4. **React + Fetch API**  
   - Single-page front-end for a smooth user experience.  
   - Fetches product and order data directly from the two APIs.  
   - Polling logic in `Orders.js` ensures the UI reflects status changes without manual refresh.
