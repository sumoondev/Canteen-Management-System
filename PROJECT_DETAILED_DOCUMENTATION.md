# Canteen Management System - Detailed Project Documentation

Updated: 2026-03-11
Workspace: `/home/suhan/CMS`

## 1. Project Summary

This project is a Django-based canteen management system designed for a college canteen or POS-style ordering workflow.

Supported roles:

- `student`
- `teacher`
- `admin`

Students and teachers use the ordering flow. Admins manage inventory and can access extra reporting pages.

## 2. Current File Structure

The project currently uses a root-level Django structure.

```
/home/suhan/CMS/
├── accounts/
├── canteen_management/
├── home/
├── inventory/
├── orders/
├── payments/
├── public/
├── manage.py
├── db.sqlite3
├── requirements.txt
├── README.md
├── PROJECT_DETAILED_DOCUMENTATION.md
└── PROJECT_DIRECTORY_DOCUMENTATION.txt
```

Important layout notes:

- `manage.py` is at the workspace root.
- Django app folders are also at the workspace root.
- the Django configuration package is `canteen_management/`.
- old references like `canteen_management/home/...` or `canteen_management/manage.py` are outdated.

## 3. Technology Stack

- Python 3.12
- Django 5.2.11
- SQLite
- Pillow
- Bootstrap
- Font Awesome
- Vanilla JavaScript
- `localStorage` for cart state on the menu page

## 4. Configuration Package

Path:

- `canteen_management/`

Key files:

- `canteen_management/settings.py`
- `canteen_management/urls.py`
- `canteen_management/asgi.py`
- `canteen_management/wsgi.py`

Current runtime facts:

- `BASE_DIR` resolves to `/home/suhan/CMS`
- database path is `/home/suhan/CMS/db.sqlite3`
- static source path is `/home/suhan/CMS/public/static`
- media upload path is `/home/suhan/CMS/public/media`
- custom user model is `accounts.CustomUser`
- time zone is `Asia/Kathmandu`

## 5. App Responsibilities

### 5.1 accounts

Path:

- `accounts/`

Purpose:

- custom user model
- role handling
- user code validation

Important file:

- `accounts/models.py`

Key rules:

- `user_code` must be exactly 5 digits
- valid roles are `admin`, `student`, and `teacher`
- superusers are forced to the `admin` role
- admin users are treated as staff users

### 5.2 home

Path:

- `home/`

Purpose:

- landing page
- login
- registration
- logout
- inventory admin dashboard
- sales analytics page
- user orders page
- update item page

Important files:

- `home/forms.py`
- `home/views.py`
- `home/tests.py`
- `home/templates/index.html`
- `home/templates/login.html`
- `home/templates/register.html`
- `home/templates/admin.html`
- `home/templates/admin_analytics.html`
- `home/templates/admin_orders.html`
- `home/templates/update_admin.html`

### 5.3 inventory

Path:

- `inventory/`

Purpose:

- inventory item model
- customer menu page

Important files:

- `inventory/models.py`
- `inventory/views.py`
- `inventory/templates/menu.html`

### 5.4 orders

Path:

- `orders/`

Purpose:

- stores orders and order items
- handles checkout

Important files:

- `orders/models.py`
- `orders/views.py`
- `orders/tests.py`

### 5.5 payments

Path:

- `payments/`

Purpose:

- stores payments and receipts
- renders receipts
- provides payment consistency checks

Important files:

- `payments/models.py`
- `payments/views.py`
- `payments/templates/receipt.html`
- `payments/management/commands/payment_consistency.py`

## 6. Main Routes

Defined in `canteen_management/urls.py`.

Primary routes:

- `/` -> landing page
- `/login/` -> login page
- `/register/` -> registration page
- `/logout/` -> logout action
- `/menu/` -> customer menu
- `/checkout/` -> checkout endpoint
- `/receipt/<order_id>/` -> receipt page
- `/admin_page/` -> custom inventory dashboard
- `/admin_page/analytics/` -> sales analytics
- `/admin_page/orders/` -> user orders
- `/admin_page/update_item/<item_id>/` -> update item page
- `/admin_page/delete_item/<item_id>/` -> delete item action
- `/admin/` -> Django admin

## 7. Access Rules

### Admin

Admins are users with:

- `role='admin'`, or
- `is_superuser=True`

Admins can access:

- `/admin_page/`
- `/admin_page/analytics/`
- `/admin_page/orders/`
- `/admin/`

### Student and Teacher

Students and teachers can:

- register through `/register/`
- log in through `/login/`
- browse `/menu/`
- add items to cart
- check out
- view their own receipts

They cannot access the custom admin pages.

## 8. Forms and Validation

### 8.1 RegistrationForm

Location:

- `home/forms.py`

Validation:

- role must be `student` or `teacher`
- `user_code` must be unique and exactly 5 digits
- username must be unique
- password uses Django password validators

### 8.2 InventoryItemForm

Location:

- `home/forms.py`

Validation currently enforced:

- item name is required
- item name must be at least 2 characters
- item name must be unique case-insensitively
- price must be a whole rupee amount
- price must be at least Rs 1
- quantity cannot be negative
- uploaded images must be valid supported image files up to 5 MB

## 9. Core Flows

### 9.1 Login Flow

View:

- `home.views.login_page`

Behavior:

1. Authenticates `username` and `password`.
2. Redirects admins to `/admin_page/`.
3. Redirects students and teachers to `/menu/`.
4. Shows an error message if login fails.

### 9.2 Registration Flow

View:

- `home.views.register_page`

Behavior:

- creates student or teacher accounts only
- redirects successful registrations to `/login/`

### 9.3 Inventory Admin Flow

Views:

- `home.views.admin_page`
- `home.views.admin_update_item`
- `home.views.admin_delete_item`

Behavior:

- admin-only access
- add item form on the dashboard
- quick edit form on the dashboard
- dedicated update page for full edits
- delete action is POST-only
- inventory stats are calculated from live data

### 9.4 Sales Analytics Flow

View:

- `home.views.admin_sales_analytics`

Behavior:

- shows daily, weekly, and monthly order counts and revenue
- shows weekly and monthly best-selling items
- shows top customers for the month
- shows recent paid orders

### 9.5 Orders Overview Flow

View:

- `home.views.admin_orders_page`

Behavior:

- lists orders with user and item details
- supports filtering by username
- supports `all`, `week`, and `month` range filters

### 9.6 Customer Menu and Cart Flow

View:

- `inventory.views.inventory_list`

Template:

- `inventory/templates/menu.html`

Behavior:

- shows only available items with stock
- cart uses `localStorage`
- cart shows per-item quantity and total item count
- checkout shows a short receipt-generation transition

### 9.7 Checkout Flow

View:

- `orders.views.checkout`

Behavior:

- validates the incoming cart payload
- locks inventory rows during validation
- rejects invalid quantities or insufficient stock
- creates `Order`, `OrderItem`, `Payment`, and `Receipt`
- marks the order as paid when checkout completes successfully

### 9.8 Receipt Flow

View:

- `payments.views.receipt_view`

Behavior:

- only the order owner can access the receipt
- the order must already be paid
- `Payment` and `Receipt` rows must exist
- date and time are rendered in 12-hour AM/PM format

## 10. Data Model Summary

### CustomUser

File:

- `accounts/models.py`

Key fields:

- `username`
- `user_code`
- `role`

### Inventory

File:

- `inventory/models.py`

Key fields:

- `item_name`
- `category`
- `price`
- `quantity`
- `food_image`
- `is_available`

### Order

File:

- `orders/models.py`

Key fields:

- `user`
- `order_date`
- `total_amount`
- `is_paid`

### OrderItem

File:

- `orders/models.py`

Key fields:

- `order`
- `item`
- `quantity`
- `price_at_purchase`

### Payment

File:

- `payments/models.py`

Key fields:

- `order`
- `payment_method`
- `amount_paid`
- `payment_time`

### Receipt

File:

- `payments/models.py`

Key fields:

- `order`
- `generated_at`

## 11. Verification Status

Verified on 2026-03-11:

- `python manage.py check` passes
- `python manage.py test --verbosity 2` passes with 41 tests
- `python manage.py makemigrations --check` reports no changes
- key templates load successfully

## 12. Most Important Maintenance Files

1. `manage.py`
2. `canteen_management/settings.py`
3. `canteen_management/urls.py`
4. `accounts/models.py`
5. `home/forms.py`
6. `home/views.py`
7. `home/templates/admin.html`
8. `home/templates/admin_analytics.html`
9. `home/templates/admin_orders.html`
10. `home/templates/update_admin.html`
11. `inventory/models.py`
12. `inventory/templates/menu.html`
13. `orders/models.py`
14. `orders/views.py`
15. `payments/models.py`
16. `payments/views.py`
17. `payments/management/commands/payment_consistency.py`