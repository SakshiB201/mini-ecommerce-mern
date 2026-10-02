# System Architecture & Technical Design

This document details the architectural design, security mechanisms, data flows, and subsystem interactions of the **Mini E-Commerce MERN Stack Platform**.

---

## 🏛️ High-Level System Architecture

The platform follows a decoupled, three-tier client-server architecture:

```mermaid
graph TD
    subgraph ClientLayer ["Client Layer (React + Vite + Tailwind)"]
        UI[Public Storefront Pages & Admin Dashboard]
        State[React Context / Hooks (Auth, Cart)]
        AxiosClient[Axios HTTP Client + Interceptors]
        UI --> State
        State --> AxiosClient
    end

    subgraph APILayer ["Backend Server (Node.js + Express)"]
        Router[Express Route Handlers]
        AuthMW[Auth Middleware (JWT Verify)]
        AdminMW[Admin Guard Middleware]
        Controllers[Controller Business Logic]
        ErrorHandler[Centralized Error Handling]
        
        Router --> AuthMW
        AuthMW --> AdminMW
        AdminMW --> Controllers
        Controllers --> ErrorHandler
    end

    subgraph DataLayer ["Persistence Layer (MongoDB + Mongoose)"]
        MUser[(User Collection)]
        MCat[(Category Collection)]
        MProd[(Product Collection)]
        MOrder[(Order Collection)]
    end

    AxiosClient -- "REST Requests (JSON + Bearer Token)" --> Router
    Controllers -- "Mongoose Queries & Transactions" --> MUser
    Controllers --> MCat
    Controllers --> MProd
    Controllers --> MOrder
```

---

## 🔐 Authentication & Authorization Lifecycle

Authentication is stateless and implemented using **JSON Web Tokens (JWT)** and **bcryptjs**.

### Token Generation & Structure
- **Payload**:
  ```json
  {
    "id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "role": "customer",
    "email": "customer@example.com",
    "iat": 1727856000,
    "exp": 1728460800
  }
  ```
- **Lifespan**: Typically 7 days.
- **Client Storage**: Token is stored in `localStorage` upon successful login/register.
- **Client Transmission**: Attached to every outgoing Axios request using the `Authorization` header:
  ```http
  Authorization: Bearer <jwt_token>
  ```

### Auth & Role Middleware Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as User / Admin Browser
    participant Client as React Axios Interceptor
    participant Route as Express Route
    participant Protect as protect Middleware
    participant AdminGuard as adminOnly Middleware
    participant Controller as Controller Action
    participant DB as MongoDB

    User->>Client: Triggers protected action
    Client->>Route: HTTP Request + Bearer <token>
    Route->>Protect: Passes request to protect middleware
    
    alt Token Missing or Invalid
        Protect-->>User: 401 Unauthorized (Invalid or expired token)
    else Token Valid
        Protect->>DB: User.findById(decoded.id).select('-password')
        DB-->>Protect: User Document
        Protect->>Route: req.user = user
        
        opt Protected Admin Route
            Route->>AdminGuard: Checks req.user.role === 'admin'
            alt Not Admin
                AdminGuard-->>User: 403 Forbidden (Admin privileges required)
            else Is Admin
                AdminGuard->>Controller: Passes control
            end
        end
        
        Controller->>DB: Executes business logic
        DB-->>Controller: Results
        Controller-->>User: 200 OK / 201 Created + Response JSON
    end
```

---

## 📦 Order Placement & Inventory Safety Flow

A critical business requirement is **Never Trust Client Prices** and **Prevent Overselling**. The order placement flow strictly adheres to server-side canonical data:

```mermaid
sequenceDiagram
    autonumber
    actor Customer as Customer (Browser)
    participant Cart as Cart Context
    participant OrderAPI as POST /api/orders
    participant OrderCtrl as Order Controller
    participant DB_Prod as Product Collection (MongoDB)
    participant DB_Order as Order Collection (MongoDB)

    Customer->>Cart: Clicks "Place Order" (COD)
    Cart->>OrderAPI: Sends { items: [{ productId, quantity }], shippingAddress }
    
    Note over OrderCtrl: Step 1: Validate Authentication (req.user)
    Note over OrderCtrl: Step 2: Validate shipping address fields
    
    loop For each item in cart
        OrderCtrl->>DB_Prod: Find product by productId
        DB_Prod-->>OrderCtrl: Product details (live price, current stock)
        
        alt Product Not Found
            OrderCtrl-->>Customer: 404 Error (Product not found)
        else Requested Quantity > Available Stock
            OrderCtrl-->>Customer: 400 Error (Insufficient stock for product)
        else Stock OK
            Note over OrderCtrl: Add to orderItems: name, qty, price: product.price
            Note over OrderCtrl: totalAmount += product.price * qty
        end
    end

    Note over OrderCtrl: Step 3: Create Order document in MongoDB
    OrderCtrl->>DB_Order: Order.create({ user, products, totalAmount, shippingAddress, status: 'Pending' })
    DB_Order-->>OrderCtrl: Saved Order Document

    Note over OrderCtrl: Step 4: Atomically decrement stock
    loop For each item
        OrderCtrl->>DB_Prod: Product.findByIdAndUpdate(productId, { $inc: { stock: -quantity } })
    end

    OrderCtrl-->>Customer: 201 Created (Order confirmed)
    Customer->>Cart: Clear cart items
    Customer->>Customer: Redirect to "My Orders" page
```

---

## 🌐 State Management Design (Frontend)

The frontend separates server synchronization from local UI state:

1. **`AuthContext`**:
   - `user`: Holds current logged-in user details (`{ id, name, email, role }`).
   - `token`: JWT string.
   - `isAuthenticated`: Boolean derived from presence of valid token.
   - `isAdmin`: Boolean derived from `user.role === 'admin'`.
   - Actions: `login(email, password)`, `register(formData)`, `logout()`.

2. **`CartContext`**:
   - `cartItems`: Array of cart objects `[{ product: { _id, name, price, image, stock }, quantity }]`.
   - `totalPrice`: Derived memoized sum `items.reduce(...)`.
   - `totalItems`: Derived count of all items in cart.
   - Actions: `addToCart(product, quantity)`, `removeFromCart(productId)`, `updateQuantity(productId, quantity)`, `clearCart()`.
   - **Persistence**: Synced with `localStorage.getItem('cart')` to survive page reloads.
   - **Boundary Enforcement**: Cannot increment beyond `product.stock`.

---

## 🛡️ Centralized Error Handling & Validation

### Express Error Pipeline
All asynchronous controller functions are wrapped in an `asyncHandler` utility to forward unhandled errors to the global error middleware:

```javascript
// Centralized Error Response Format
{
  "success": false,
  "message": "Specific human-readable error description",
  "errors": ["Optional list of field-level validation errors"]
}
```

### HTTP Status Code Conventions
- `200 OK`: Request succeeded.
- `201 Created`: Resource successfully created (User registered, Product/Category added, Order created).
- `400 Bad Request`: Input validation failed, password mismatch, or insufficient stock.
- `401 Unauthorized`: Missing or malformed JWT token.
- `403 Forbidden`: Authenticated user lacks admin permissions.
- `404 Not Found`: Requested resource ID does not exist.
- `500 Internal Server Error`: Unhandled server exception.
