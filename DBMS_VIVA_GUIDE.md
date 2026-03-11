# DBMS Viva Guide

This document explains the main DBMS concepts used in the Django Canteen Management System so you can defend the project clearly during a university viva.

## 1. Project Database Overview

This project stores canteen data in a relational database managed through Django ORM.

Main data areas:

- users and roles
- inventory items
- orders
- ordered item details
- payments
- receipts

The same application can run on SQLite locally and PostgreSQL in production. For Railway deployment, PostgreSQL is the correct production database because it handles multiple concurrent users more safely and efficiently.

## 2. Why A DBMS Is Needed In This Project

The system must store structured information that changes over time.

Examples:

- admins add and update food items
- customers place orders
- stock quantities decrease after checkout
- payment records and receipts must remain consistent

A DBMS is used because it provides:

- structured tables
- relationships between tables
- validation and constraints
- transaction support
- multi-user access
- reliable querying and reporting

## 3. Main Entities In The Project

### CustomUser

Represents application users such as students, teachers, and admins.

Important fields:

- username
- password
- role
- user_code

### Inventory

Represents menu items available in the canteen.

Important fields:

- item_name
- category
- price
- quantity
- food_image
- is_available

### Order

Represents a complete customer order.

Important fields:

- user
- total_amount
- order_date
- is_paid

### OrderItem

Represents the individual items inside one order.

Important fields:

- order
- item
- quantity
- price_at_purchase

### Payment

Represents payment information for an order.

Important fields:

- order
- payment_method
- amount_paid

### Receipt

Represents the receipt generated for a paid order.

Important fields:

- order
- created_at or generated receipt metadata

## 4. Relationships Used In The Project

This project follows the relational model.

### One-to-Many Relationship

One user can place many orders.

- `CustomUser -> Order`

One inventory item can appear in many order items.

- `Inventory -> OrderItem`

### Many-to-One Relationship

Each order belongs to one user.

- `Order.user`

Each order item belongs to one order.

- `OrderItem.order`

Each order item refers to one inventory item.

- `OrderItem.item`

### One-to-One or Unique-Linked Business Records

Payment and receipt records are tied to a specific order.

- `Order -> Payment`
- `Order -> Receipt`

This prevents one paid order from having ambiguous receipt or payment ownership.

## 5. Normalization In This Project

Normalization means structuring data to reduce redundancy and avoid update anomalies.

### First Normal Form (1NF)

Each table stores atomic values.

Examples:

- one `item_name` per inventory row
- one `quantity` value per order item row
- one `username` per user row

There are no repeating groups like multiple item names stored in one order column.

### Second Normal Form (2NF)

All non-key attributes depend on the whole key.

In `OrderItem`, values such as `quantity` and `price_at_purchase` depend on the specific order-item record, not only on `order` or only on `item`.

### Third Normal Form (3NF)

Non-key attributes do not depend on other non-key attributes.

Examples:

- user role is stored with the user, not copied into orders
- item category and price are stored in inventory, not duplicated unnecessarily across unrelated tables
- order totals are part of the order record, while line-level quantities remain in `OrderItem`

This design reduces duplication and makes updates cleaner.

## 6. CRUD Operations In The System

CRUD stands for Create, Read, Update, Delete.

### Create

- admin adds a new inventory item
- customer creates an order during checkout
- system creates payment and receipt records

### Read

- users read the menu page
- admin views inventory, orders, and analytics
- receipt page displays paid order details

### Update

- admin updates item name, price, quantity, and availability
- checkout updates inventory quantity and availability status
- order payment status is updated to paid

### Delete

- admin can delete inventory items

These CRUD operations are implemented through Django views, forms, models, and ORM queries.

## 7. Transactions And Consistency

One of the most important DBMS concepts in this project is transaction management.

During checkout, the application must:

1. validate cart items
2. lock the selected inventory rows
3. create the order
4. create related order items
5. reduce stock quantities
6. create payment and receipt records
7. mark the order as paid

These steps must behave like one logical unit of work.

The project uses `transaction.atomic()` so either all checkout steps succeed together or the whole operation is rolled back.

Why this matters:

- it prevents half-completed orders
- it protects inventory consistency
- it keeps payment and receipt records synchronized with the order

## 8. Concurrency: Why PostgreSQL Is Better Than SQLite

This is a strong viva topic.

### SQLite

SQLite is file-based and very good for:

- small local projects
- quick development setup
- single-user or low-concurrency use

Limitations for this project in production:

- limited write concurrency
- database locking is more restrictive
- not ideal when multiple users place orders at the same time
- less suitable for hosted multi-user systems like Railway deployments

### PostgreSQL

PostgreSQL is a server-based relational DBMS and is better for production because it offers:

- stronger concurrency handling
- MVCC behavior for better simultaneous access
- better support for multiple users reading and writing at once
- stronger production reliability and scaling options
- better performance and administration for live systems

For a canteen system where multiple customers may order at the same time while admins also update stock, PostgreSQL is the correct production choice.

## 9. Integrity Constraints In The Project

The project enforces several data integrity rules.

Examples:

- item quantity cannot be negative
- price must be a positive whole rupee amount
- custom user codes are validated
- duplicate item names are prevented in forms
- unavailable or out-of-stock items cannot be checked out

These rules are enforced through:

- model field types
- form validation
- application logic in views
- relational references between tables

## 10. How Django ORM Helps

Django ORM lets the project interact with the database using Python objects instead of raw SQL in most cases.

Examples of ORM benefits:

- easier CRUD operations
- safer queries
- database abstraction between SQLite and PostgreSQL
- support for filtering, aggregation, joins, and related lookups

Examples used in this project:

- filtering available inventory
- aggregating sales totals with `Sum`
- counting related records with `Count`
- `select_related` and `prefetch_related` for efficient relationship loading

## 11. AJAX And Database Interaction

The project uses AJAX polling to refresh inventory, analytics, and orders in the browser without reloading the page.

This does not replace the DBMS. Instead, it improves the user interface while still reading fresh data from the database through Django views that return JSON.

Example flow:

1. browser sends a request to a snapshot endpoint
2. Django queries the database
3. Django returns JSON
4. JavaScript updates the UI dynamically

This helps show near-real-time changes in stock and order information.

## 12. Why The Data Model Is Reasonable

You can explain the model in viva like this:

- `CustomUser` stores who uses the system
- `Inventory` stores what can be sold
- `Order` stores the master transaction
- `OrderItem` stores the detailed line items
- `Payment` stores how much was paid
- `Receipt` stores the final proof of payment

This separation is good database design because each table has a clear responsibility.

## 13. Common Viva Questions And Short Answers

### Why did you use a relational database?

Because the project has clearly related entities such as users, orders, items, payments, and receipts. A relational database models these relationships cleanly and enforces consistency.

### Why did you separate `Order` and `OrderItem`?

One order can contain multiple items. Splitting them avoids repeating order-level information and supports proper normalization.

### Why store `price_at_purchase` in `OrderItem`?

Because product prices may change later. Storing the purchase-time price preserves historical accuracy of old orders.

### Why use PostgreSQL on Railway instead of SQLite?

PostgreSQL handles concurrent users, transactions, and production workloads much better than SQLite, which is better suited for local development and small single-user scenarios.

### What ensures stock is not oversold?

The checkout logic validates stock inside a transaction and uses row locking behavior through database queries before reducing inventory quantities.

## 14. Strong Viva Conclusion

You can summarize the DBMS design like this:

This project uses a normalized relational database design implemented through Django ORM. It supports CRUD operations, relationships, transaction-safe checkout, and live AJAX-based UI updates. SQLite is used for simple local development, while PostgreSQL is used in production because it provides stronger concurrency handling and better reliability for multiple simultaneous users.
