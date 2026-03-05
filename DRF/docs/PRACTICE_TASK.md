---
title: Practice Task - Real-Time Food Delivery System API
description: Comprehensive practice task building a complete food delivery system API with Django REST Framework
---

# Practice Task: Real-Time Food Delivery System API

## Overview

Build a **Real-Time Food Delivery System** API where customers can browse restaurants, place orders, track deliveries in real-time, and receive live updates through WebSockets. This project integrates all DRF concepts from Level 0 to Level 5.

**Focus:** Complete REST API with Authentication, Relationships, Query Optimization, Caching, File Uploads, Third-Party Integration, and Real-Time WebSocket Updates

**Important:** Follow the best practices mentioned in [Django REST Best Practices](https://pragnakalp.github.io/django-rest-best-practices) for all implementations throughout this project.

---

## What You'll Build

A food delivery management API with the following endpoints and features:

### 1. User Authentication System (`/api/auth/`)
- User registration with custom user model (username, email, phone)
- Multiple user roles: Customer, Restaurant Owner, Delivery Driver
- JWT token-based authentication (login, refresh)
- User profile management with avatar upload
- Role-specific profile data (customer addresses, driver vehicle info, restaurant details)

### 2. Restaurants Management (`/api/restaurants/`)
- List all restaurants (paginated, 20 per page)
- Create/Update/Delete restaurants (restaurant owners only)
- Restaurant details: name, cuisine type, address, opening hours, delivery fee, minimum order
- Upload restaurant banner and logo images
- Search restaurants by name or cuisine
- Filter by cuisine type, rating, delivery fee, is_open status
- View restaurant menu items and reviews

### 3. Menu Items (`/api/menu-items/`)
- List menu items by restaurant (paginated)
- Create/Update/Delete menu items (restaurant owner only)
- Item details: name, description, price, category, dietary info, availability
- Upload food images
- Search items by name or description
- Filter by category (Appetizer, Main Course, Dessert, Beverage), dietary preferences (Vegetarian, Vegan, Gluten-Free)
- Mark items as available/unavailable

### 4. Orders (`/api/orders/`)
- List orders (customers see their orders, drivers see assigned orders, owners see restaurant orders)
- Create orders with multiple items (customers only)
- Order details with nested items, restaurant, customer, delivery driver
- Order status workflow: Pending → Confirmed → Preparing → Ready → Picked Up → Delivered → Cancelled
- Calculate order totals
- Assign delivery driver to order
- Rate and review completed orders

### 5. Order Management (`/api/orders/management/`)
- List active orders (drivers see assigned, customers see their orders)
- Update order status in real-time (drivers and restaurant owners only)
- Delivery status updates
- Estimated delivery time calculation
- Order history and statistics

### 6. Reviews & Ratings (`/api/reviews/`)
- List reviews for restaurants and menu items
- Create reviews (customers only, after order completion)
- Rating (1-5 stars) with optional comment
- Filter reviews by rating
- Calculate average ratings for restaurants and items

### 7. Real-Time WebSocket Endpoints
- **Order Updates** (`ws/orders/<id>/`)
  - Order status changed (Confirmed, Preparing, Ready, etc.)
  - Driver assigned
  - Estimated delivery time updated
  
- **Order Management** (`ws/orders/management/<id>/`)
  - Order status updates
  - Order completed
  
- **Restaurant Dashboard** (`ws/restaurants/<id>/`)
  - New order received
  - Order status updates
  - Real-time order queue

---

## Technical Requirements

### Models to Create

**Figure out the field types yourself based on the requirements:**

1. **CustomUser** (extends AbstractUser)
   - username (unique)
   - email (unique)
   - phone_number (unique)
   - first_name
   - last_name
   - user_type (choices: Customer, Restaurant Owner, Delivery Driver)
   - is_active (boolean)
   - created_at
   - updated_at

2. **CustomerProfile** (OneToOne with User)
   - user (ForeignKey to User)
   - avatar (image upload, max 5MB)
   - default_address (TextField)
   - saved_addresses (JSONField or separate Address model)
   - total_orders
   - loyalty_points (default 0)
   - created_at
   - updated_at

3. **DriverProfile** (OneToOne with User)
   - user (ForeignKey to User)
   - avatar (image upload, max 5MB)
   - vehicle_type (choices: Bike, Scooter, Car)
   - vehicle_number
   - license_number
   - is_available (boolean, default True)
   - total_deliveries
   - average_rating (decimal, default 0)
   - created_at
   - updated_at

4. **Restaurant**
   - owner (ForeignKey to User)
   - name
   - description
   - cuisine_type (choices: Italian, Chinese, Indian, Mexican, American, Japanese, Thai, Mediterranean)
   - address
   - phone_number
   - email
   - logo (image upload, max 5MB)
   - banner (image upload, max 10MB)
   - opening_time
   - closing_time
   - is_open (boolean)
   - delivery_fee (decimal)
   - minimum_order (decimal, default 0)
   - average_rating (decimal, default 0)
   - total_reviews
   - created_at
   - updated_at

5. **MenuItem**
   - restaurant (ForeignKey to Restaurant)
   - name
   - description
   - price (decimal)
   - category (choices: Appetizer, Main Course, Dessert, Beverage, Side Dish)
   - dietary_info (choices: Vegetarian, Vegan, Gluten-Free, Dairy-Free, None)
   - image (image upload, max 5MB)
   - is_available (boolean, default True)
   - preparation_time (integer, minutes)
   - created_at
   - updated_at

6. **Order**
   - customer (ForeignKey to User)
   - restaurant (ForeignKey to Restaurant)
   - driver (ForeignKey to User, nullable)
   - order_number (unique, auto-generated)
   - status (choices: Pending, Confirmed, Preparing, Ready, Picked Up, Delivered, Cancelled)
   - delivery_address
   - subtotal (decimal)
   - delivery_fee (decimal)
   - tax (decimal)
   - total_amount (decimal)
   - special_instructions (TextField, nullable)
   - estimated_delivery_time (DateTimeField, nullable)
   - actual_delivery_time (DateTimeField, nullable)
   - created_at
   - updated_at

7. **OrderItem** (ManyToMany through model)
   - order (ForeignKey to Order)
   - menu_item (ForeignKey to MenuItem)
   - quantity
   - price (decimal, snapshot of item price at order time)
   - special_instructions (TextField, nullable)
   - created_at

8. **Review**
   - customer (ForeignKey to User)
   - restaurant (ForeignKey to Restaurant, nullable)
   - menu_item (ForeignKey to MenuItem, nullable)
   - order (ForeignKey to Order)
   - rating (integer, 1-5)
   - comment (TextField, nullable)
   - created_at
   - updated_at


---

## What You Need to Figure Out

### 1. **Model Implementation**
- Define all models with appropriate field types (CharField, IntegerField, DecimalField, etc.)
- Set up ForeignKey relationships with correct on_delete behavior
- Create ManyToMany relationship through OrderItem model
- Add database indexes for frequently queried fields (status, created_at, restaurant, customer)
- Implement model methods:
  - Order: `calculate_total()`, `can_cancel()`, `is_delivered()`
  - Restaurant: `is_currently_open()`, `update_average_rating()`
  - DriverProfile: `update_availability()`, `get_delivery_stats()`

### 2. **Serializers**
- Create ModelSerializers for all models
- Implement nested serializers:
  - OrderDetailSerializer with nested items, restaurant, customer, driver
  - RestaurantDetailSerializer with nested menu items and reviews
  - MenuItemSerializer with restaurant info
- Add SerializerMethodField for computed fields:
  - `is_open_now`, `estimated_delivery_time` (Restaurant)
  - `average_rating`, `total_reviews` (Restaurant, MenuItem)
  - `can_review`, `can_cancel` (Order)
  - `items_count`, `final_total` (Order)
- Implement custom validation:
  - Order total must meet restaurant minimum_order
  - Delivery address is required for delivery orders
  - Menu items must belong to the same restaurant
  - Rating must be between 1-5
  - Driver can only accept orders when available
- Override `create()` for Order to:
  - Calculate subtotal, tax, delivery fee
  - Create OrderItem entries
  - Send WebSocket notification to restaurant

### 3. **Authentication & Permissions**
- Set up JWT authentication (djangorestframework-simplejwt)
- Create custom permissions:
  - `IsOwnerOrReadOnly` - owner can edit/delete, others read-only
  - `IsRestaurantOwner` - only restaurant owner can edit restaurant and menu items
  - `IsCustomer` - only customers can place orders and write reviews
  - `IsDriver` - only drivers can update delivery status and location
  - `IsOrderCustomer` - only order customer can view order details
  - `IsRestaurantOwnerOrDriver` - restaurant owner or assigned driver can update order status
- Implement user registration endpoint with role selection
- Create role-specific profile automatically on user creation (using signals)
- Add throttling:
  - Anonymous: 100/hour
  - Authenticated: 1000/hour
  - Order creation: 20/hour
  - Review creation: 10/hour
  - Location updates: 500/hour (for drivers)

### 4. **ViewSets & URL Routing**
- Create ModelViewSets for all models
- Implement custom actions:
  - `@action` for placing orders (POST /orders/place/)
  - `@action` for cancelling orders (POST /orders/{id}/cancel/)
  - `@action` for updating order status (POST /orders/{id}/update-status/)
  - `@action` for assigning driver (POST /orders/{id}/assign-driver/)
  - `@action` for updating order status (POST /orders/{id}/update-status/)
  - `@action` for restaurant menu (GET /restaurants/{id}/menu/)
  - `@action` for popular restaurants (GET /restaurants/popular/)
- Set up URL routing with DefaultRouter
- Implement API versioning (v1, v2)
- Override `get_queryset()` to filter based on user role:
  - Customers see only their orders
  - Drivers see only assigned orders
  - Restaurant owners see only their restaurant's orders

### 5. **Filtering, Searching, Ordering**
- Install and configure django-filter
- Implement filters:
  - **Restaurants:** cuisine_type, is_open, delivery_fee__lte, minimum_order__lte, average_rating__gte
  - **Menu Items:** restaurant, category, dietary_info, is_available, price__lte
  - **Orders:** status, restaurant, created_at__gte
  - **Reviews:** rating, restaurant, menu_item
- Add search functionality:
  - **Restaurants:** name, cuisine_type, description
  - **Menu Items:** name, description
  - **Orders:** order_number
- Configure ordering:
  - **Restaurants:** average_rating (descending), delivery_fee, created_at
  - **Menu Items:** price, name, created_at
  - **Orders:** created_at (descending), total_amount
  - **Reviews:** created_at (descending), rating

### 6. **Pagination**
- Restaurants: PageNumberPagination (20 per page)
- Menu Items: PageNumberPagination (30 per page)
- Orders: CursorPagination (25 per page, ordered by created_at)
- Reviews: LimitOffsetPagination (default 20, max 50)

### 7. **Query Optimization**
- Use `select_related()` for ForeignKey relationships:
  - Order: customer, restaurant, driver
  - MenuItem: restaurant
  - Review: customer, restaurant, menu_item, order
  - Restaurant: owner
- Use `prefetch_related()` for ManyToMany and reverse ForeignKey:
  - Order: items (OrderItem), items__menu_item
  - Restaurant: menu_items, reviews
  - MenuItem: reviews
- Add `annotate()` for computed fields:
  - Restaurant: items_count, reviews_count, avg_rating
  - Order: items_count, total_items_quantity
  - MenuItem: reviews_count, avg_rating
- Prevent N+1 queries in all list endpoints
- Use `only()` and `defer()` to limit fields when appropriate

### 8. **Django Signals**
- Create signals in `api/signals.py`:
  - **post_save(CustomUser, created=True):** Create role-specific profile (CustomerProfile, DriverProfile)
  - **post_save(Order, created=True):** Send WebSocket notification to restaurant
  - **post_save(Order):** When status changes to 'Delivered', update customer and driver statistics
  - **post_save(Review, created=True):** Update restaurant/menu item average rating
  - **post_save(OrderItem):** Update order subtotal when items are added
  - **pre_save(Order):** Calculate total_amount before saving
- Register signals in `api/apps.py`

### 9. **Caching with Redis**
- Install django-redis
- Configure Redis cache backend
- Implement caching strategy:
  - Restaurant list: 5 minutes
  - Restaurant detail: 10 minutes
  - Menu items by restaurant: 15 minutes
  - Popular restaurants: 30 minutes
- Invalidate cache on create/update/delete operations:
  - Clear restaurant cache when menu items change
  - Clear menu cache when availability changes
  - Clear restaurant cache when reviews are added
- Use low-level cache API for:
  - Popular restaurants calculation
  - Driver availability status

### 10. **File Uploads**
- Configure MEDIA_URL and MEDIA_ROOT
- Implement file upload validation:
  - User avatar: max 5MB, formats: jpg, jpeg, png
  - Restaurant logo: max 5MB, formats: jpg, jpeg, png
  - Restaurant banner: max 10MB, formats: jpg, jpeg, png
  - Menu item image: max 5MB, formats: jpg, jpeg, png
- Add file size and format validation in serializers
- Implement image compression using Pillow
- Set up static file serving for development
- Use unique filenames to prevent overwrites

### 11. **WebSocket Implementation (Django Channels)**
- Install channels and channels-redis
- Configure ASGI application and channel layers
- Create WebSocket consumers:
  - **OrderConsumer:** Real-time order status updates
  - **OrderManagementConsumer:** Real-time order status updates
  - **RestaurantDashboardConsumer:** Real-time new orders for restaurant
- Set up WebSocket routing
- Send WebSocket messages from views when:
  - New order is placed (notify restaurant)
  - Order status changes (notify customer, restaurant, driver)
  - Driver is assigned (notify customer and driver)
  - Order is delivered (notify all parties)
- Handle WebSocket authentication using JWT
- Implement room-based messaging:
  - Order room: `order_<order_id>`
  - Restaurant room: `restaurant_<restaurant_id>`
  - Customer room: `customer_<user_id>`
  - Driver room: `driver_<user_id>`

### 12. **API Documentation**
- Install drf-spectacular
- Configure OpenAPI 3.0 schema generation
- Add Swagger UI at `/api/docs/`
- Add ReDoc at `/api/redoc/`
- Customize schema with:
  - Detailed endpoint descriptions
  - Request/response examples
  - Authentication requirements
  - Role-based access documentation
  - WebSocket endpoint documentation
