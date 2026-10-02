# REST API Specification

This document provides the complete API contract for the **Mini E-Commerce MERN Platform**.

- **Base URL**: `http://localhost:5000/api`
- **Default Headers**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <jwt_token>` *(required for protected endpoints)*

---

## 📑 Endpoints Summary

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| **POST** | `/api/auth/register` | Public | Register new customer account |
| **POST** | `/api/auth/login` | Public | Login customer or admin |
| **GET** | `/api/categories` | Public | List all categories |
| **POST** | `/api/categories` | Admin | Create a new category |
| **PUT** | `/api/categories/:id` | Admin | Update existing category |
| **DELETE** | `/api/categories/:id` | Admin | Delete category |
| **GET** | `/api/products` | Public | List products (with category & search filters) |
| **GET** | `/api/products/:id` | Public | Get single product by ID |
| **POST** | `/api/products` | Admin | Create a new product |
| **PUT** | `/api/products/:id` | Admin | Update existing product |
| **DELETE** | `/api/products/:id` | Admin | Delete product |
| **POST** | `/api/orders` | Customer | Place new order with COD and stock validation |
| **GET** | `/api/orders/my-orders`| Customer | Get authenticated user's order history |
| **GET** | `/api/admin/orders` | Admin | Get all platform orders |
| **PATCH**| `/api/admin/orders/:id/status` | Admin | Update order status |

---

## 1. Authentication Endpoints

### 1.1 Register Customer
- **Endpoint**: `POST /api/auth/register`
- **Access**: Public
- **Request Body**:
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "securepassword123",
  "confirmPassword": "securepassword123"
}
```
- **Success Response (201 Created)**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "_id": "64f1a2b3c4d5e6f7a8b9c001",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "role": "customer",
      "createdAt": "2026-10-02T10:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```
- **Error Responses**:
  - `400 Bad Request`: Passwords do not match, missing fields, or password length < 6.
  - `400 Bad Request`: Email already in use (`"User already exists with this email"`).

---

### 1.2 Login User (Customer or Admin)
- **Endpoint**: `POST /api/auth/login`
- **Access**: Public
- **Request Body**:
```json
{
  "email": "admin@example.com",
  "password": "adminpassword123"
}
```
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "_id": "64f1a2b3c4d5e6f7a8b9c002",
      "name": "Platform Admin",
      "email": "admin@example.com",
      "role": "admin"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```
- **Error Responses**:
  - `401 Unauthorized`: Invalid email or password credentials.

---

## 2. Category Endpoints

### 2.1 Get All Categories
- **Endpoint**: `GET /api/categories`
- **Access**: Public
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "_id": "64f1a2b3c4d5e6f7a8b9c010",
      "name": "Electronics",
      "description": "Smartphones, laptops, and gadgets",
      "createdAt": "2026-10-02T08:30:00.000Z"
    },
    {
      "_id": "64f1a2b3c4d5e6f7a8b9c011",
      "name": "Fashion",
      "description": "Apparel, clothing, and accessories",
      "createdAt": "2026-10-02T08:35:00.000Z"
    }
  ]
}
```

---

### 2.2 Create Category
- **Endpoint**: `POST /api/categories`
- **Access**: Admin (Requires Bearer Token)
- **Request Body**:
```json
{
  "name": "Shoes",
  "description": "Footwear, sneakers, and running shoes"
}
```
- **Success Response (201 Created)**:
```json
{
  "success": true,
  "message": "Category created successfully",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c012",
    "name": "Shoes",
    "description": "Footwear, sneakers, and running shoes",
    "createdAt": "2026-10-02T09:00:00.000Z"
  }
}
```

---

### 2.3 Update Category
- **Endpoint**: `PUT /api/categories/:id`
- **Access**: Admin
- **Request Body**:
```json
{
  "name": "Footwear & Shoes",
  "description": "Updated category description"
}
```
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "message": "Category updated successfully",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c012",
    "name": "Footwear & Shoes",
    "description": "Updated category description",
    "updatedAt": "2026-10-02T09:10:00.000Z"
  }
}
```

---

### 2.4 Delete Category
- **Endpoint**: `DELETE /api/categories/:id`
- **Access**: Admin
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "message": "Category deleted successfully"
}
```

---

## 3. Product Endpoints

### 3.1 Get All Products (with Filtering & Search)
- **Endpoint**: `GET /api/products`
- **Access**: Public
- **Query Parameters**:
  - `category` *(optional)*: Category ID or slug (e.g. `?category=electronics`)
  - `search` *(optional)*: Case-insensitive search keyword (e.g. `?search=phone`)
- **Example Request**:
  `GET /api/products?category=64f1a2b3c4d5e6f7a8b9c010&search=wireless`
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "_id": "64f1a2b3c4d5e6f7a8b9c020",
      "name": "Wireless Noise Cancelling Headphones",
      "description": "Premium over-ear wireless headphones with active noise cancellation.",
      "price": 199.99,
      "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e",
      "category": {
        "_id": "64f1a2b3c4d5e6f7a8b9c010",
        "name": "Electronics"
      },
      "stock": 15,
      "createdAt": "2026-10-02T09:15:00.000Z"
    }
  ]
}
```

---

### 3.2 Get Single Product by ID
- **Endpoint**: `GET /api/products/:id`
- **Access**: Public
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c020",
    "name": "Wireless Noise Cancelling Headphones",
    "description": "Premium over-ear wireless headphones with active noise cancellation.",
    "price": 199.99,
    "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e",
    "category": {
      "_id": "64f1a2b3c4d5e6f7a8b9c010",
      "name": "Electronics"
    },
    "stock": 15,
    "createdAt": "2026-10-02T09:15:00.000Z"
  }
}
```

---

### 3.3 Create Product
- **Endpoint**: `POST /api/products`
- **Access**: Admin
- **Request Body**:
```json
{
  "name": "Classic Denim Jacket",
  "description": "100% cotton vintage style denim jacket.",
  "price": 79.99,
  "image": "https://images.unsplash.com/photo-1576995853123-5a10305d93c0",
  "category": "64f1a2b3c4d5e6f7a8b9c011",
  "stock": 25
}
```
- **Success Response (201 Created)**:
```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c021",
    "name": "Classic Denim Jacket",
    "description": "100% cotton vintage style denim jacket.",
    "price": 79.99,
    "image": "https://images.unsplash.com/photo-1576995853123-5a10305d93c0",
    "category": "64f1a2b3c4d5e6f7a8b9c011",
    "stock": 25,
    "createdAt": "2026-10-02T09:20:00.000Z"
  }
}
```

---

### 3.4 Update Product
- **Endpoint**: `PUT /api/products/:id`
- **Access**: Admin
- **Request Body**:
```json
{
  "price": 69.99,
  "stock": 30
}
```
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c021",
    "name": "Classic Denim Jacket",
    "price": 69.99,
    "stock": 30
  }
}
```

---

### 3.5 Delete Product
- **Endpoint**: `DELETE /api/products/:id`
- **Access**: Admin
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

---

## 4. Order Endpoints

### 4.1 Place Order (Cash on Delivery)
- **Endpoint**: `POST /api/orders`
- **Access**: Customer (Authenticated)
- **Validation**:
  - The server extracts items from payload, queries MongoDB for canonical prices, verifies stock availability, deducts stock atomically, and creates the Order document.
- **Request Body**:
```json
{
  "items": [
    {
      "product": "64f1a2b3c4d5e6f7a8b9c020",
      "quantity": 2
    },
    {
      "product": "64f1a2b3c4d5e6f7a8b9c021",
      "quantity": 1
    }
  ],
  "shippingAddress": {
    "name": "Jane Doe",
    "phone": "+1 555-0199",
    "address": "456 Market St, Suite 200",
    "city": "San Francisco",
    "pincode": "94103"
  }
}
```
- **Success Response (201 Created)**:
```json
{
  "success": true,
  "message": "Order placed successfully via Cash on Delivery",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c030",
    "user": "64f1a2b3c4d5e6f7a8b9c001",
    "products": [
      {
        "product": "64f1a2b3c4d5e6f7a8b9c020",
        "name": "Wireless Noise Cancelling Headphones",
        "quantity": 2,
        "price": 199.99
      },
      {
        "product": "64f1a2b3c4d5e6f7a8b9c021",
        "name": "Classic Denim Jacket",
        "quantity": 1,
        "price": 69.99
      }
    ],
    "totalAmount": 469.97,
    "shippingAddress": {
      "name": "Jane Doe",
      "phone": "+1 555-0199",
      "address": "456 Market St, Suite 200",
      "city": "San Francisco",
      "pincode": "94103"
    },
    "status": "Pending",
    "createdAt": "2026-10-02T09:30:00.000Z"
  }
}
```
- **Error Response (400 Bad Request)**:
```json
{
  "success": false,
  "message": "Product 'Wireless Noise Cancelling Headphones' has only 1 items left in stock"
}
```

---

### 4.2 Get Customer Order History
- **Endpoint**: `GET /api/orders/my-orders`
- **Access**: Customer (Authenticated)
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "_id": "64f1a2b3c4d5e6f7a8b9c030",
      "products": [
        {
          "product": "64f1a2b3c4d5e6f7a8b9c020",
          "name": "Wireless Noise Cancelling Headphones",
          "quantity": 2,
          "price": 199.99
        }
      ],
      "totalAmount": 469.97,
      "shippingAddress": {
        "name": "Jane Doe",
        "phone": "+1 555-0199",
        "address": "456 Market St, Suite 200",
        "city": "San Francisco",
        "pincode": "94103"
      },
      "status": "Pending",
      "createdAt": "2026-10-02T09:30:00.000Z"
    }
  ]
}
```

---

### 4.3 Get All Orders (Admin View)
- **Endpoint**: `GET /api/admin/orders`
- **Access**: Admin
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "count": 1,
  "data": [
    {
      "_id": "64f1a2b3c4d5e6f7a8b9c030",
      "user": {
        "_id": "64f1a2b3c4d5e6f7a8b9c001",
        "name": "Jane Doe",
        "email": "jane@example.com"
      },
      "products": [ ... ],
      "totalAmount": 469.97,
      "shippingAddress": { ... },
      "status": "Pending",
      "createdAt": "2026-10-02T09:30:00.000Z"
    }
  ]
}
```

---

### 4.4 Update Order Status
- **Endpoint**: `PATCH /api/admin/orders/:id/status`
- **Access**: Admin
- **Allowed Status Values**:
  - `Pending`
  - `Confirmed`
  - `Shipped`
  - `Delivered`
  - `Cancelled`
- **Request Body**:
```json
{
  "status": "Shipped"
}
```
- **Success Response (200 OK)**:
```json
{
  "success": true,
  "message": "Order status updated to Shipped",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c030",
    "status": "Shipped",
    "updatedAt": "2026-10-02T10:15:00.000Z"
  }
}
```
