# Team Task Allocation & GitHub Issues

This document tracks task assignments and GitHub issue mappings for the 4 collaborators working on the **Mini E-Commerce MERN Stack Platform**.

---

## 👥 Team Members & Issue Mapping

| Collaborator | GitHub User | Issue | Module & Responsibilities |
| :--- | :--- | :---: | :--- |
| **Sakshi** | [`@SakshiB201`](https://github.com/SakshiB201) | [#1](https://github.com/SakshiB201/mini-ecommerce-mern/issues/1) | **Backend Architecture, DB Connection, JWT Auth & Security** |
| **Tanmay** | [`@TanmayT134`](https://github.com/TanmayT134) | [#2](https://github.com/SakshiB201/mini-ecommerce-mern/issues/2) | **Category & Product APIs, Catalog Search/Filter & Admin UI** |
| **Aaryan** | [`@aryansathate-design`](https://github.com/aryansathate-design) | [#3](https://github.com/SakshiB201/mini-ecommerce-mern/issues/3) | **Frontend Architecture, Storefront UI/UX, Catalog & Product Views** |
| **Aditya** | *(Pending invite)* | [#4](https://github.com/SakshiB201/mini-ecommerce-mern/issues/4) | **Cart Context, Cash on Delivery Checkout, Order APIs & Stock Lifecycle** |

---

## 📋 Detailed Responsibilities Breakdown

### 1. Sakshi ([#1](https://github.com/SakshiB201/mini-ecommerce-mern/issues/1))
**Module: Backend Core Architecture, Database Schemas, Authentication & Security**
- [ ] Initialize Express backend server in `/server` with CORS, JSON body parser, and error handling middleware.
- [ ] Implement MongoDB connection utility in `server/src/config/db.js` using Mongoose.
- [ ] Create `User` Mongoose schema in `server/src/models/User.js`:
  - Fields: `name`, `email` (unique, regex validated), `password` (hashed, excluded in query projections), `role` (`customer` | `admin`).
  - Pre-save hook using `bcryptjs` (salt rounds: 10).
  - Instance method `matchPassword(enteredPassword)`.
- [ ] Create JWT helper in `server/src/utils/generateToken.js` with expiration.
- [ ] Create Auth & Admin Middlewares in `server/src/middlewares/authMiddleware.js`:
  - `protect`: Verifies Bearer JWT token from `Authorization` header and attaches `req.user`.
  - `adminOnly`: Restricts route access to users with `role === 'admin'`.
- [ ] Implement Authentication REST API endpoints in `server/src/controllers/authController.js` and `server/src/routes/authRoutes.js`:
  - `POST /api/auth/register` (Fields: Name, Email, Password, Confirm Password validation).
  - `POST /api/auth/login` (Customer & Admin login returning user payload + JWT).
- [ ] Create seed script `server/src/utils/seedAdmin.js` to initialize default platform admin credentials.

---

### 2. Tanmay ([#2](https://github.com/SakshiB201/mini-ecommerce-mern/issues/2))
**Module: Catalog Management, Category & Product APIs & Admin Dashboard Views**
- [ ] Create `Category` Mongoose schema in `server/src/models/Category.js`:
  - Fields: `name` (unique, trimmed), `description`, timestamps.
- [ ] Create `Product` Mongoose schema in `server/src/models/Product.js`:
  - Fields: `name`, `description`, `price` (> 0), `image`, `category` (ObjectId ref to Category), `stock` (>= 0), timestamps.
  - Text search index on `name` and index on `category`.
- [ ] Implement Category REST APIs in `server/src/controllers/categoryController.js` and `server/src/routes/categoryRoutes.js`:
  - `GET /api/categories` (Public).
  - `POST /api/categories` (Admin).
  - `PUT /api/categories/:id` (Admin).
  - `DELETE /api/categories/:id` (Admin).
- [ ] Implement Product REST APIs in `server/src/controllers/productController.js` and `server/src/routes/productRoutes.js`:
  - `GET /api/products` supporting query filters `?category=...&search=...` (Public).
  - `GET /api/products/:id` (Public).
  - `POST /api/products` (Admin).
  - `PUT /api/products/:id` (Admin).
  - `DELETE /api/products/:id` (Admin).
- [ ] Build Admin Frontend Management UI in `/client`:
  - `AdminLayout.jsx` with collapsible `AdminSidebar.jsx` (Dashboard, Categories, Products, Orders, Logout).
  - `Categories.jsx` (`/admin/categories`): Responsive table, Add/Edit modal, Delete confirmation dialog.
  - `Products.jsx` (`/admin/products`): Responsive table showing thumbnail, name, category, price, stock; Add/Edit modal; Delete confirmation dialog.

---

### 3. Aaryan ([#3](https://github.com/SakshiB201/mini-ecommerce-mern/issues/3))
**Module: Frontend Architecture, Storefront UI/UX, Product Browsing & Search**
- [ ] Scaffold `/client` using Vite + React with Tailwind CSS and install dependencies (`react-router-dom`, `axios`, `react-hot-toast`, `lucide-react`).
- [ ] Set up Axios HTTP client in `client/src/services/api.js` with request interceptor for JWT `Authorization: Bearer <token>`.
- [ ] Implement `AuthContext` in `client/src/context/AuthContext.jsx`:
  - `login`, `register`, `logout` actions.
  - Persist credentials & token in `localStorage`.
- [ ] Build Storefront Layout & Navigation:
  - `MainLayout.jsx` with sticky responsive `Navbar.jsx` (brand logo, search shortcut, cart icon with dynamic badge count, auth dropdown).
  - Mobile hamburger slide-over menu and responsive `Footer.jsx`.
- [ ] Build Authentication Pages:
  - `Login.jsx`: Clean credentials form with loading state and error handling.
  - `Register.jsx`: Registration form with client-side password confirmation validation.
- [ ] Build Public Storefront Pages:
  - `Home.jsx`: Minimalist hero banner, category highlight shortcuts, trending products section.
  - `Products.jsx`: Responsive product catalog, instant search input, interactive category filter tabs (`All | Electronics | Fashion | Shoes`).
  - `ProductCard.jsx`: Thumbnail with hover zoom, title, category tag, formatted price, stock badge, and direct "Add to Cart" action.
  - `ProductDetails.jsx`: Detailed product view with stock counter, quantity stepper, and "Add to Cart" integration.
- [ ] Implement UI/UX feedback: Loading skeleton loaders, empty state components, and toast notifications.

---

### 4. Aditya ([#4](https://github.com/SakshiB201/mini-ecommerce-mern/issues/4))
**Module: Shopping Cart, Checkout (COD), Order Processing & Inventory Lifecycle**
- [ ] Implement `CartContext` in `client/src/context/CartContext.jsx`:
  - Actions: `addToCart`, `updateQuantity`, `removeFromCart`, `clearCart`.
  - Enforce inventory boundaries: Prevent quantity from exceeding `product.stock`.
  - Compute live subtotal and total items; persist in `localStorage`.
- [ ] Build Shopping Cart UI in `client/src/pages/Cart.jsx`:
  - Item listing with quantity increment/decrement buttons, removal action, subtotal summary card, and "Proceed to Checkout" CTA.
  - Empty cart state with redirect to `/products`.
- [ ] Build Checkout Page in `client/src/pages/Checkout.jsx`:
  - Protected route requiring customer login.
  - Shipping address form with validation: Full Name, Phone, Address, City, Pincode.
  - Dedicated **Cash on Delivery (COD)** payment selection.
  - Order submission handling with cart clearance and navigation to `/my-orders`.
- [ ] Create `Order` Mongoose schema in `server/src/models/Order.js`:
  - `user` (ref User), `products` array (`product`, `name`, `quantity`, `price`), `totalAmount`, `shippingAddress`, `status` enum (`Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`), timestamps.
- [ ] Implement Orders REST APIs in `server/src/controllers/orderController.js` and `server/src/routes/orderRoutes.js`:
  - `POST /api/orders`: Secure server-side pricing lookup (never trust client prices), stock validation, atomic stock decrement (`$inc: { stock: -quantity }`), and order document creation.
  - `GET /api/orders/my-orders`: Retrieve authenticated customer's order history.
  - `GET /api/admin/orders`: Admin retrieval of all customer orders across the platform.
  - `PATCH /api/admin/orders/:id/status`: Admin status transitions (`Pending` ➔ `Confirmed` ➔ `Shipped` ➔ `Delivered` ➔ `Cancelled`).
- [ ] Build Order Views:
  - `client/src/pages/MyOrders.jsx`: Card-based order history with real-time status badges and item summaries.
  - `client/src/pages/admin/Orders.jsx`: Admin table for inspecting orders and updating fulfillment status.
