# Software Requirements & API Specification

## 1. Project Overview

It is a location-based agricultural marketplace that directly connects farmers with buyers.

Farmers can list their agricultural products with information such as price, quantity, location, harvest date, and images. Buyers can search and purchase products directly from farmers without unnecessary intermediaries.

The platform also provides:

1. **Location-based product discovery**
2. **Local market price information**
3. **Farmer verification**
4. **Order and inventory management**
5. **Ratings and reviews**
6. **AI-powered marketplace assistant**
7. **Admin management and moderation**

### Core Concept

```
Farmer
   │
   │ Product Listing
   ▼
Website
   │
   ├── Product Marketplace
   ├── Market Price Intelligence
   ├── AI Assistant
   └── Order Management
   │
   ▼
Buyer
```

---

# 2. Technology Stack

## Frontend

- Next.js
- TypeScript

## Backend

- Node.js
- Express.js
- TypeScript
- Modular Architecture
- REST API

## Database

- PostgreSQL
- Prisma ORM

## Authentication

- JWT Access Token
- Refresh Token
- Password hashing using bcrypt

## Validation

- Zod

## AI

- LLM API
- Retrieval from application/database data before generating responses

## Testing (We will do it letter)

- Jest
- Supertest

## API Documentation

- Swagger / OpenAPI / Postman

---

# 3. User Roles

The system will have three primary roles.

### 3.1 Buyer

A buyer can:

- Register and log in
- Browse products
- Search products
- Filter products by location, price, category, etc.
- View farmer profiles
- Add products to cart
- Place orders
- Cancel eligible orders
- Track orders
- Review purchased products/farmers
- Use AI Assistant
- Manage wishlist
- Receive notifications

### 3.2 Farmer

A farmer can:

- Register and log in as a farmer
- Manage farmer profile
- Submit verification information
- Create product listings
- Update product listings
- Delete/hide listings
- Manage product quantity
- View incoming orders
- Accept/reject orders
- Update order status
- View sales information
- View market prices
- Use AI Assistant
- Receive notifications

### 3.3 Admin

An admin can:

- Manage users
- Verify farmers
- Suspend users
- Manage categories
- Moderate products
- Manage market price data
- View orders
- Manage reports
- Moderate reviews
- View platform analytics

---

# 4. Functional Requirements

## 4.1 Authentication

The system shall provide:

- User registration
- User login
- Logout
- Access token generation
- Refresh token generation
- Password update
- Current user information
- Role-based access control

### Registration

A user must provide:

```
name
email
phone
password
role
location
```

Allowed roles:

```
BUYER
FARMER
```

`ADMIN` cannot be selected during public registration.

---

# 5. Farmer Verification

Every farmer must have a verification status.

```
PENDING
VERIFIED
REJECTED
SUSPENDED
```

Flow:

```
Farmer Registration
       ↓
PENDING
       ↓
Admin Review
       ↓
VERIFIED / REJECTED
```

Only verified farmers should be allowed to publish active product listings.

---

# 6. Product Management

A farmer can create a product listing containing:

```
Product Name
Description
Category
Price
Unit
Quantity
Available Quantity
Location
Harvest Date
Available Until
Images
```

### Product status

```
ACTIVE
SOLD_OUT
EXPIRED
HIDDEN
REJECTED
```

---

# 7. Product Search

Users can search products using:

- Product name
- Category
- District
- Upazila
- Area
- Price range
- Availability
- Farmer
- Distance

Example:

```
GET /api/v1/products?search=tomato&district=Rajshahi&minPrice=40&maxPrice=60
```

---

# 8. Location System

The application will support hierarchical locations:

```
Division
   ↓
District
   ↓
Upazila
   ↓
Area
```

Example:

```
Rajshahi Division
    ↓
Rajshahi District
    ↓
Paba Upazila
    ↓
Area
```

Products and users can be associated with locations.

---

# 9. Market Price Intelligence

The system will maintain market price records.

Example:

```
Product: Tomato
Location: Rajshahi
Average Price: ৳55/kg
Minimum: ৳48/kg
Maximum: ৳62/kg
Date: 2026-10-05
```

Farmers and buyers can view market price information.

The system can also compare:

```
Market Average
Platform Average
Individual Farmer Price
```

---

# 10. Shopping Cart

Buyers can:

- Add products
- Update quantities
- Remove products
- Clear cart
- View cart totals

The backend must verify that:

- Product is active
- Product is available
- Requested quantity is available
- Product price is valid

---

# 11. Order Management

Order lifecycle:

```
PENDING
    ↓
CONFIRMED
    ↓
PROCESSING
    ↓
READY
    ↓
SHIPPED
    ↓
DELIVERED
```

Alternative states:

```
CANCELLED
REJECTED
```

Only authorized users can change order status.

---

# 12. Reviews & Ratings

A buyer can review a product/farmer only after purchasing the product.

Rating:

```
1–5
```

Review contains:

```
rating
comment
product
farmer
order
buyer
```

The backend must verify that the buyer actually purchased the product.

---

# 13. AI Assistant

The AI Assistant will support:

### Product Search

> "Find tomatoes under 60 taka near Rajshahi."
> 

### Market Price

> "What is the current tomato price in Rajshahi?"
> 

### Product Recommendation

> "Which farmer has the cheapest potatoes near me?"
> 

### Farmer Assistance

> "How should I create a product listing?"
> 

### Order Assistance

> "Where is my order?"
> 

---

# 14. AI Architecture

The frontend must **not** directly call the LLM provider.

Instead:

```
Next.js
   ↓
Express API
   ↓
AI Service
   ↓
Database / Marketplace Search
   ↓
LLM API
   ↓
AI Response
```

For marketplace-related questions, the system should retrieve relevant application data before asking the LLM to formulate the response.

Example:

```
User:
"Where can I buy tomatoes under 60 taka in Rajshahi?"

        ↓

AI Intent Detection

        ↓

Product Search

        ↓

PostgreSQL

        ↓

Matching Products

        ↓

LLM

        ↓

Natural Language Response
```

This makes the AI assistant more useful than a generic chatbot.

---

# 15. Database Schema

The following is the recommended initial relational model.

## User

```
User
----
id
name
email
phone
passwordHash
role
status
locationId
createdAt
updatedAt
```

### Role

```
BUYER
FARMER
ADMIN
```

### Status

```
ACTIVE
SUSPENDED
PENDING
```

---

# 16. FarmerProfile

```
FarmerProfile
-------------
id
userId
farmName
farmDescription
verificationStatus
createdAt
updatedAt
```

Relationship:

```
User 1 ─── 1 FarmerProfile
```

---

# 17. BuyerProfile

```
BuyerProfile
------------
id
userId
address
createdAt
updatedAt
```

---

# 18. Location

```
Location
--------
id
division
district
upazila
area
latitude
longitude
createdAt
updatedAt
```

---

# 19. Category

```
Category
--------
id
name
description
isActive
createdAt
updatedAt
```

---

# 20. Product

```
Product
-------
id
farmerId
categoryId
locationId
name
description
price
unit
quantity
availableQuantity
harvestDate
availableUntil
status
createdAt
updatedAt
```

---

# 21. ProductImage

```
ProductImage
------------
id
productId
imageUrl
isPrimary
createdAt
```

---

# 22. Cart

```
Cart
----
id
buyerId
createdAt
updatedAt
```

---

# 23. CartItem

```
CartItem
--------
id
cartId
productId
quantity
createdAt
updatedAt
```

---

# 24. Order

```
Order
-----
id
buyerId
totalAmount
status
paymentMethod
paymentStatus
deliveryAddress
deliveryPhone
deliveryNote
createdAt
updatedAt
```

---

# 25. OrderItem

```
OrderItem
---------
id
orderId
productId
farmerId
quantity
unitPrice
subtotal
createdAt
```

`unitPrice` must be stored because the product price can change after an order is created.

---

# 26. Review

```
Review
------
id
buyerId
productId
farmerId
orderId
rating
comment
createdAt
updatedAt
```

---

# 27. Wishlist

```
Wishlist
--------
id
buyerId
createdAt
updatedAt
```

```
WishlistItem
------------
id
wishlistId
productId
createdAt
```

---

# 28. MarketPrice

```
MarketPrice
-----------
id
categoryId
locationId
price
unit
source
recordedAt
createdAt
```

---

# 29. Notification

```
Notification
------------
id
userId
type
title
message
isRead
createdAt
```

---

# 30. AIConversation

```
AIConversation
-------------
id
userId
title
createdAt
updatedAt
```

---

# 31. AIMessage

```
AIMessage
---------
id
conversationId
role
content
createdAt
```

---

# 32. Report

```
Report
------
id
reporterId
reportedUserId
productId
reason
description
status
createdAt
updatedAt
```

---

# 33. Main Database Relationships

```
User
 ├── FarmerProfile
 │       │
 │       └── Products
 │
 ├── BuyerProfile
 │
 ├── Orders
 │
 ├── Reviews
 │
 ├── Wishlist
 │
 ├── Notifications
 │
 └── AIConversations

Product
 ├── Category
 ├── Location
 ├── ProductImages
 ├── CartItems
 ├── OrderItems
 ├── Reviews
 └── WishlistItems

Order
 └── OrderItems

Location
 ├── Products
 └── MarketPrices
```

---

# 34. Backend Architecture

Follow modular pattern architechture

## Recommended Structure

```
backend/
│
├── src/
│   │
│   ├── app.ts
│   ├── server.ts
│   │
│   ├── config/
│   │   ├── env.ts
│   │   └── database.ts
│   │
│   ├── middlewares/
│   │   ├── auth.middleware.ts
│   │   ├── role.middleware.ts
│   │   ├── error.middleware.ts
│   │   └── rate-limit.middleware.ts
│   │
│   ├── modules/
│   │
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.routes.ts
│   │   │   ├── auth.validation.ts
│   │   │   ├── auth.types.ts
│   │   │   └── auth.constants.ts
│   │   │
│   │   ├── user/
│   │   │   ├── user.controller.ts
│   │   │   ├── user.service.ts
│   │   │   ├── user.routes.ts
│   │   │   ├── user.validation.ts
│   │   │   └── user.types.ts
│   │   │
│   │   ├── farmer/
│   │   │   ├── farmer.controller.ts
│   │   │   ├── farmer.service.ts
│   │   │   ├── farmer.routes.ts
│   │   │   ├── farmer.validation.ts
│   │   │   └── farmer.types.ts
│   │   │
│   │   ├── product/
│   │   │   ├── product.controller.ts
│   │   │   ├── product.service.ts
│   │   │   ├── product.routes.ts
│   │   │   ├── product.validation.ts
│   │   │   └── product.types.ts
│   │   │
│   │   ├── category/
│   │   ├── location/
│   │   ├── cart/
│   │   ├── order/
│   │   ├── review/
│   │   ├── wishlist/
│   │   ├── market-price/
│   │   ├── notification/
│   │   ├── ai/
│   │   ├── report/
│   │   └── admin/
│   │
│   ├── utils/
│   │   ├── jwt.ts
│   │   ├── password.ts
│   │   ├── pagination.ts
│   │   └── response.ts
│   │
│   └── types/
│       └── express.d.ts
│
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
│
├── tests/
│
└── package.json
```

---

# 35. Module Responsibility

Every module should follow:

```
Route
  ↓
Controller
  ↓
Service
  ↓
Prisma
  ↓
PostgreSQL
```

### Route

Responsible for:

- URL
- HTTP method
- middleware
- authorization
- validation

### Controller

Responsible for:

- reading request
- calling service
- returning response

### Service

Responsible for:

- business logic
- database operations
- transaction handling
- business rules

---

# 36. Example Product Module

```
modules/
└── product/
    ├── product.controller.ts
    ├── product.service.ts
    ├── product.routes.ts
    ├── product.validation.ts
    └── product.types.ts
```

### product.routes.ts

```
POST   /
GET    /
GET    /:id
PATCH  /:id
DELETE /:id
```

### product.controller.ts

```
createProduct()
getProducts()
getProductById()
updateProduct()
deleteProduct()
```

### product.service.ts

```
createProduct()
getProducts()
getProductById()
updateProduct()
deleteProduct()
```

The controller should remain thin.

Business logic belongs inside the service.

---

# 37. API Design Standard

Every API will follow this structure:

```
HTTP Method
Endpoint
Authentication
Authorization
Request
Response
Possible Errors
```

---

# 38. Standard API Response

### Success

```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {}
}
```

### Error

```json
{
  "success": false,
  "message": "Product not found",
  "errors": []
}
```

---

# 39. Authentication APIs

## 39.1 Register

```
POST /api/v1/auth/register
```

### Access

Public

### Request

```tsx
{
  name: string;
  email: string;
  phone: string;
  password: string;
  role: "BUYER" | "FARMER";
  locationId: string;
}
```

### Response

```tsx
{
  success: true;
  message: string;
  data: {
    user: {
      id: string;
      name: string;
      email: string;
      role: "BUYER" | "FARMER";
    };
  };
}
```

### Errors

```
400 Invalid input
409 Email already exists
409 Phone already exists
```

---

# 40. Login

```
POST /api/v1/auth/login
```

### Access

Public

### Request

```tsx
{
  email: string;
  password: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Login successful";
  data: {
    accessToken: string;
    refreshToken: string;
    user: {
      id: string;
      name: string;
      email: string;
      role: "BUYER" | "FARMER" | "ADMIN";
    };
  };
}
```

### Errors

```
401 Invalid credentials
403 Account suspended
```

---

# 41. Refresh Token

```
POST /api/v1/auth/refresh
```

### Request

```tsx
{
  refreshToken: string;
}
```

### Response

```tsx
{
  success: true;
  data: {
    accessToken: string;
  };
}
```

---

# 42. Logout

```
POST /api/v1/auth/logout
```

### Access

Authenticated user

### Request

```tsx
{
  refreshToken: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Logged out successfully";
}
```

---

# 43. Current User

```
GET /api/v1/auth/me
```

### Access

Authenticated user

### Request

No body.

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    name: string;
    email: string;
    phone: string;
    role: "BUYER" | "FARMER" | "ADMIN";
    location: {
      id: string;
      district: string;
      upazila: string;
      area: string;
    };
  };
}
```

---

# 44. User APIs

## Get My Profile

```
GET /api/v1/users/me
```

**Access:** Authenticated users

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    name: string;
    email: string;
    phone: string;
    role: string;
    location: Location;
  };
}
```

---

## Update My Profile

```
PATCH /api/v1/users/me
```

### Request

```tsx
{
  name?: string;
  phone?: string;
  locationId?: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Profile updated successfully";
  data: User;
}
```

---

# 45. Farmer APIs

## Get Farmer Profile

```
GET /api/v1/farmers/:farmerId
```

### Access

Public

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    name: string;
    farmName: string;
    farmDescription: string;
    verificationStatus: string;
    location: Location;
    rating: number;
    totalReviews: number;
  };
}
```

---

# 46. Update Farmer Profile

```
PATCH /api/v1/farmers/me
```

### Access

FARMER

### Request

```tsx
{
  farmName?: string;
  farmDescription?: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Farmer profile updated successfully";
  data: FarmerProfile;
}
```

---

# 47. Farmer Dashboard

```
GET /api/v1/farmers/me/dashboard
```

### Access

FARMER

### Response

```tsx
{
  success: true;
  data: {
    totalProducts: number;
    activeProducts: number;
    pendingOrders: number;
    completedOrders: number;
    totalSales: number;
    averageRating: number;
  };
}
```

---

# 48. Product APIs

## Create Product

```
POST /api/v1/products
```

### Access

FARMER + VERIFIED

### Request

```tsx
{
  name: string;
  description?: string;
  categoryId: string;
  locationId: string;
  price: number;
  unit: "KG" | "PIECE" | "LITER" | "DOZEN";
  quantity: number;
  harvestDate?: string;
  availableUntil?: string;
  images?: string[];
}
```

### Response

```tsx
{
  success: true;
  message: "Product created successfully";
  data: Product;
}
```

### Errors

```
400 Invalid product data
403 Farmer is not verified
404 Category not found
404 Location not found
```

---

# 49. Get Products

```
GET /api/v1/products
```

### Access

Public

### Query Parameters

```tsx
{
  search?: string;
  categoryId?: string;
  district?: string;
  upazila?: string;
  area?: string;
  minPrice?: number;
  maxPrice?: number;
  farmerId?: string;
  status?: string;
  page?: number;
  limit?: number;
}
```

### Response

```tsx
{
  success: true;
  data: Product[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

---

# 50. Get Product Details

```
GET /api/v1/products/:productId
```

### Access

Public

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    name: string;
    description: string;
    price: number;
    unit: string;
    availableQuantity: number;
    harvestDate: string;
    images: ProductImage[];
    category: Category;
    location: Location;
    farmer: {
      id: string;
      name: string;
      farmName: string;
      rating: number;
      verificationStatus: string;
    };
  };
}
```

---

# 51. Update Product

```
PATCH /api/v1/products/:productId
```

### Access

Product owner FARMER

### Request

All fields optional:

```tsx
{
  name?: string;
  description?: string;
  price?: number;
  quantity?: number;
  availableUntil?: string;
  status?: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Product updated successfully";
  data: Product;
}
```

---

# 52. Delete Product

```
DELETE /api/v1/products/:productId
```

### Access

Product owner FARMER / ADMIN

### Response

```tsx
{
  success: true;
  message: "Product deleted successfully";
}
```

---

# 53. Category APIs

## Get Categories

```
GET /api/v1/categories
```

### Access

Public

### Response

```tsx
{
  success: true;
  data: Category[];
}
```

---

## Create Category

```
POST /api/v1/categories
```

### Access

ADMIN

### Request

```tsx
{
  name: string;
  description?: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Category created successfully";
  data: Category;
}
```

---

# 54. Cart APIs

## Get Cart

```
GET /api/v1/cart
```

### Access

BUYER

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    items: {
      id: string;
      product: Product;
      quantity: number;
      subtotal: number;
    }[];
    totalAmount: number;
  };
}
```

---

## Add to Cart

```
POST /api/v1/cart/items
```

### Request

```tsx
{
  productId: string;
  quantity: number;
}
```

### Response

```tsx
{
  success: true;
  message: "Product added to cart";
  data: Cart;
}
```

---

## Update Cart Item

```
PATCH /api/v1/cart/items/:itemId
```

### Request

```tsx
{
  quantity: number;
}
```

### Response

```tsx
{
  success: true;
  data: Cart;
}
```

---

## Remove Cart Item

```
DELETE /api/v1/cart/items/:itemId
```

### Response

```tsx
{
  success: true;
  message: "Item removed from cart";
}
```

---

# 55. Order APIs

## Create Order

```
POST /api/v1/orders
```

### Access

BUYER

### Request

```tsx
{
  deliveryAddress: string;
  deliveryPhone: string;
  deliveryNote?: string;
  paymentMethod: "COD";
}
```

The backend will use the authenticated buyer's cart.

### Response

```tsx
{
  success: true;
  message: "Order placed successfully";
  data: {
    orderId: string;
    totalAmount: number;
    status: "PENDING";
    items: OrderItem[];
  };
}
```

---

# 56. Get My Orders

```
GET /api/v1/orders/me
```

### Access

BUYER

### Query

```tsx
{
  status?: string;
  page?: number;
  limit?: number;
}
```

### Response

```tsx
{
  success: true;
  data: Order[];
  pagination: Pagination;
}
```

---

# 57. Get Order Details

```
GET /api/v1/orders/:orderId
```

### Access

- Order owner BUYER
- Related FARMER
- ADMIN

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    status: string;
    totalAmount: number;
    deliveryAddress: string;
    items: OrderItem[];
    buyer: BuyerSummary;
  };
}
```

---

# 58. Cancel Order

```
PATCH /api/v1/orders/:orderId/cancel
```

### Access

BUYER

### Response

```tsx
{
  success: true;
  message: "Order cancelled successfully";
  data: Order;
}
```

The service must verify that the order is still cancellable.

---

# 59. Farmer Orders

```
GET /api/v1/farmers/me/orders
```

### Access

FARMER

### Response

```tsx
{
  success: true;
  data: Order[];
  pagination: Pagination;
}
```

Only orders containing that farmer's products should be returned.

---

# 60. Update Order Status

```
PATCH /api/v1/orders/:orderId/status
```

### Access

FARMER / ADMIN

### Request

```tsx
{
  status:
    | "CONFIRMED"
    | "REJECTED"
    | "PROCESSING"
    | "READY"
    | "SHIPPED"
    | "DELIVERED";
}
```

### Response

```tsx
{
  success: true;
  message: "Order status updated successfully";
  data: Order;
}
```

The service must validate allowed status transitions.

---

# 61. Market Price APIs

## Get Market Prices

```
GET /api/v1/market-prices
```

### Access

Public

### Query

```tsx
{
  categoryId?: string;
  district?: string;
  upazila?: string;
  date?: string;
}
```

### Response

```tsx
{
  success: true;
  data: {
    product: string;
    location: Location;
    averagePrice: number;
    minimumPrice: number;
    maximumPrice: number;
    unit: string;
    recordedAt: string;
  }[];
}
```

---

# 62. Create Market Price

```
POST /api/v1/market-prices
```

### Access

ADMIN

### Request

```tsx
{
  categoryId: string;
  locationId: string;
  price: number;
  unit: string;
  source: string;
  recordedAt: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Market price created successfully";
  data: MarketPrice;
}
```

---

# 63. Reviews

## Create Review

```
POST /api/v1/reviews
```

### Access

BUYER

### Request

```tsx
{
  orderId: string;
  productId: string;
  rating: number;
  comment?: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Review submitted successfully";
  data: Review;
}
```

### Validation

The backend must verify:

```
Order belongs to buyer
        +
Order is DELIVERED
        +
Product exists in order
        +
Buyer has not already reviewed it
```

---

# 64. Get Product Reviews

```
GET /api/v1/products/:productId/reviews
```

### Access

Public

### Response

```tsx
{
  success: true;
  data: Review[];
}
```

---

# 65. Wishlist APIs

## Add Wishlist

```
POST /api/v1/wishlist/:productId
```

### Access

BUYER

### Response

```tsx
{
  success: true;
  message: "Product added to wishlist";
}
```

## Remove Wishlist

```
DELETE /api/v1/wishlist/:productId
```

### Access

BUYER

## Get Wishlist

```
GET /api/v1/wishlist
```

### Response

```tsx
{
  success: true;
  data: Product[];
}
```

---

# 66. AI APIs

## Start/Send AI Conversation

```
POST /api/v1/ai/chat
```

### Access

Authenticated users

### Request

```tsx
{
  conversationId?: string;
  message: string;
}
```

### Response

```tsx
{
  success: true;
  data: {
    conversationId: string;
    message: string;
    sources?: {
      type: "PRODUCT" | "MARKET_PRICE" | "ORDER";
      id: string;
    }[];
  };
}
```

---

# 67. AI Conversation History

```
GET /api/v1/ai/conversations
```

### Access

Authenticated user

### Response

```tsx
{
  success: true;
  data: AIConversation[];
}
```

---

# 68. Get AI Conversation

```
GET /api/v1/ai/conversations/:conversationId
```

### Access

Conversation owner

### Response

```tsx
{
  success: true;
  data: {
    id: string;
    title: string;
    messages: AIMessage[];
  };
}
```

---

# 69. Delete AI Conversation

```
DELETE /api/v1/ai/conversations/:conversationId
```

### Access

Conversation owner

### Response

```tsx
{
  success: true;
  message: "Conversation deleted successfully";
}
```

---

# 70. Notification APIs

## Get Notifications

```
GET /api/v1/notifications
```

### Access

Authenticated

### Response

```tsx
{
  success: true;
  data: Notification[];
}
```

---

## Mark Notification as Read

```
PATCH /api/v1/notifications/:notificationId/read
```

### Access

Notification owner

### Response

```tsx
{
  success: true;
  message: "Notification marked as read";
}
```

---

# 71. Report APIs

## Create Report

```
POST /api/v1/reports
```

### Access

Authenticated

### Request

```tsx
{
  reportedUserId?: string;
  productId?: string;
  reason:
    | "FAKE_PRODUCT"
    | "WRONG_PRICE"
    | "MISLEADING_INFORMATION"
    | "FRAUD"
    | "ABUSIVE_BEHAVIOR"
    | "OTHER";
  description?: string;
}
```

### Response

```tsx
{
  success: true;
  message: "Report submitted successfully";
  data: Report;
}
```

---

# 72. Admin APIs

## Admin Dashboard

```
GET /api/v1/admin/dashboard
```

### Access

ADMIN

### Response

```tsx
{
  success: true;
  data: {
    totalUsers: number;
    totalFarmers: number;
    verifiedFarmers: number;
    totalBuyers: number;
    totalProducts: number;
    activeProducts: number;
    totalOrders: number;
    totalSales: number;
    pendingReports: number;
  };
}
```

---

# 73. Admin User Management

```
GET /api/v1/admin/users
```

### Query

```tsx
{
  role?: string;
  status?: string;
  page?: number;
  limit?: number;
}
```

---

## Update User Status

```
PATCH /api/v1/admin/users/:userId/status
```

### Request

```tsx
{
  status: "ACTIVE" | "SUSPENDED";
}
```

---

# 74. Admin Farmer Verification

## Get Pending Farmers

```
GET /api/v1/admin/farmers/pending
```

### Access

ADMIN

### Response

```tsx
{
  success: true;
  data: FarmerProfile[];
}
```

---

## Verify Farmer

```
PATCH /api/v1/admin/farmers/:farmerId/verify
```

### Response

```tsx
{
  success: true;
  message: "Farmer verified successfully";
  data: FarmerProfile;
}
```

---

## Reject Farmer

```
PATCH /api/v1/admin/farmers/:farmerId/reject
```

### Request

```tsx
{
  reason?: string;
}
```

---

# 75. Admin Product Moderation

## Get Products

```
GET /api/v1/admin/products
```

### Access

ADMIN

---

## Approve Product

```
PATCH /api/v1/admin/products/:productId/approve
```

### Response

```tsx
{
  success: true;
  message: "Product approved successfully";
}
```

---

## Reject Product

```
PATCH /api/v1/admin/products/:productId/reject
```

### Request

```tsx
{
  reason?: string;
}
```

---

# 76. Admin Reports

## Get Reports

```
GET /api/v1/admin/reports
```

### Access

ADMIN

### Query

```tsx
{
  status?: "PENDING" | "REVIEWED" | "RESOLVED" | "DISMISSED";
  page?: number;
  limit?: number;
}
```

---

## Update Report

```
PATCH /api/v1/admin/reports/:reportId
```

### Request

```tsx
{
  status: "REVIEWED" | "RESOLVED" | "DISMISSED";
}
```

---

# 77. API Access Matrix

This is something I strongly recommend including in your project documentation.

| Module | Guest | Buyer | Farmer | Admin |
| --- | --- | --- | --- | --- |
| Register | ✅ | — | — | — |
| Login | ✅ | — | — | — |
| Browse Products | ✅ | ✅ | ✅ | ✅ |
| Search Products | ✅ | ✅ | ✅ | ✅ |
| View Market Prices | ✅ | ✅ | ✅ | ✅ |
| View Farmer | ✅ | ✅ | ✅ | ✅ |
| Create Product | ❌ | ❌ | ✅ | Optional |
| Update Own Product | ❌ | ❌ | ✅ | ✅ |
| Create Order | ❌ | ✅ | ❌ | ❌ |
| View Own Orders | ❌ | ✅ | — | — |
| View Farmer Orders | ❌ | ❌ | ✅ | ✅ |
| Update Order | ❌ | ❌ | ✅ | ✅ |
| Review | ❌ | ✅ | ❌ | — |
| Wishlist | ❌ | ✅ | ❌ | — |
| AI Assistant | Limited | ✅ | ✅ | Optional |
| Create Market Price | ❌ | ❌ | ❌ | ✅ |
| Verify Farmer | ❌ | ❌ | ❌ | ✅ |
| Manage Categories | ❌ | ❌ | ❌ | ✅ |
| Manage Users | ❌ | ❌ | ❌ | ✅ |
| Moderate Products | ❌ | ❌ | ❌ | ✅ |
| Manage Reports | ❌ | ❌ | ❌ | ✅ |

---

# 78. API Error Handling

The backend should use consistent HTTP status codes.

| Status | Meaning |
| --- | --- |
| 200 | Successful request |
| 201 | Resource created |
| 400 | Bad request |
| 401 | Unauthenticated |
| 403 | Unauthorized |
| 404 | Resource not found |
| 409 | Conflict |
| 422 | Validation error |
| 429 | Too many requests |
| 500 | Internal server error |

---

# 79. Important Business Rules

These rules should be implemented inside **services**, not controllers.

### Product

- Only verified farmers can create active listings.
- Farmer can only update/delete own products.
- Sold-out products cannot be ordered.
- Expired products cannot be ordered.
- Quantity cannot be negative.
- Price cannot be negative.

### Cart

- Quantity must be greater than zero.
- Quantity cannot exceed available quantity.

### Order

- Buyer can only cancel eligible orders.
- Farmer can only update orders containing their products.
- Order status transitions must be validated.
- Product quantity must be reduced atomically during order creation.

### Review

- Only buyers can review.
- Buyer must have purchased the product.
- Order must be delivered.
- One review per purchase/product.

### Admin

- Only admin can verify farmers.
- Only admin can manage market-price records.
- Only admin can suspend users.

---

# 80. Transaction Requirement

Order creation is one of the places where Prisma transactions should be used.

For example:

```
Create Order
      +
Create Order Items
      +
Decrease Product Quantity
      +
Clear Cart
      +
Create Notification
```

These operations should succeed together.

If one fails, the transaction should roll back.

Conceptually:

```
BEGIN TRANSACTION

Create Order
Create OrderItems
Update Product Quantity
Clear Cart

COMMIT
```

If something fails:

```
ROLLBACK
```

---

# 81. Backend Module List

The final backend modules should be:

```
modules/

├── auth
├── user
├── farmer
├── buyer
├── product
├── category
├── location
├── cart
├── order
├── review
├── wishlist
├── market-price
├── notification
├── ai
├── report
└── admin
```

---

# 82. Standard Module Structure

Every major module should follow:

```
product/
│
├── product.controller.ts
├── product.service.ts
├── product.routes.ts
├── product.validation.ts
├── product.types.ts
└── product.constants.ts
```

Not every module necessarily needs every file.

For example:

```
location/
├── location.controller.ts
├── location.service.ts
├── location.routes.ts
└── location.types.ts
```

---

# 83. Controller Responsibility

Example:

```tsx
createProduct(req, res, next)
```

Controller should:

1. Read request
2. Get authenticated user
3. Call service
4. Return response
5. Pass errors to error middleware

It should **not** contain complex business logic.

Bad:

```
controller
 ├── validate farmer
 ├── query database
 ├── calculate price
 ├── update quantity
 └── create product
```

Better:

```
Controller
    ↓
Product Service
    ↓
Prisma
```

---

# 84. Service Responsibility

Example:

```
ProductService.createProduct()
```

should handle:

```
Check farmer
Check verification
Check category
Check location
Validate business rules
Create product
Create images
Return product
```

This is where the actual business logic belongs.

---

# 85. Validation

Use Zod schemas.

Example concept:

```
createProductSchema
updateProductSchema
productQuerySchema
```

The validation middleware will validate:

```
req.body
req.params
req.query
```

before the controller is executed.

---

# 86. Testing Requirements

Each important module should have:

### Unit Tests

For services/business logic.

Example:

```
ProductService
OrderService
AuthService
```

### Integration/API Tests

Using:

```
Jest + Supertest
```

Examples:

```
POST /auth/register
POST /auth/login
POST /products
GET /products
POST /orders
```

### Important test cases

You should test:

- Unauthorized request
- Wrong role
- Invalid input
- Non-existing product
- Insufficient quantity
- Unauthorized product update
- Invalid order transition
- Duplicate review
- Non-purchased product review

---

# 87. API Documentation

Use Swagger/OpenAPI /Postman

Every endpoint should document:

```
Endpoint
HTTP Method
Description
Authentication
Role
Parameters
Request Body
Success Response
Error Responses
```

Then  backend can expose:

```
/api-docs
```

---

# 88. MVP Scope

Version : v1

### Must Have

- Authentication
- Buyer/Farmer roles
- Farmer verification
- Product CRUD
- Product search
- Location filtering
- Cart
- Orders
- Farmer order management
- Market prices
- Reviews
- AI Assistant
- Admin dashboard

### Phase 2

- Wishlist
- Notifications
- Advanced location search
- Price charts
- Product recommendations
- Reporting system

### Future

- Online payments
- Delivery partner integration
- GPS distance
- Real-time market price integration
- AI price prediction
- Weather integration
- Voice-based AI assistant
- Mobile application

---

# 89. Final Architecture

The complete system will look like this:

```
                         ┌──────────────────┐
                         │     Next.js      │
                         │    Frontend      │
                         └────────┬─────────┘
                                  │
                              REST API
                                  │
                         ┌────────▼─────────┐
                         │     Express      │
                         │     Backend      │
                         └────────┬─────────┘
                                  │
                   ┌──────────────┼──────────────┐
                   │              │              │
                   ▼              ▼              ▼
               Controllers     Services      Middleware
                                  │
                                  ▼
                             Prisma ORM
                                  │
                                  ▼
                            PostgreSQL
                                  │
                   ┌──────────────┼──────────────┐
                   │              │              │
                   ▼              ▼              ▼
               Products        Orders       Market Data
                                  │
                                  │
                                  ▼
                             AI Service
                                  │
                                  ▼
                              LLM API
```

## Recommended development order

Divide it by modules:

```
Phase 1
├── Database schema
├── Prisma setup
└── Authentication

Phase 2
├── User/Farmer
├── Location
├── Category
└── Product

Phase 3
├── Cart
├── Order
└── Inventory

Phase 4
├── Market Price
├── Review
└── Notification

Phase 5
├── AI Assistant
└── Admin

Phase 6
├── Next.js integration
├── Testing
├── Swagger
└── Deployment
```