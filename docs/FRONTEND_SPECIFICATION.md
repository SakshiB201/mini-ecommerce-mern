# Frontend Specification & UI/UX Design

This document details the frontend architecture, page layouts, component hierarchy, design system, and state management for the **React + Vite + Tailwind CSS** client.

---

## 🎨 Design System & Theme Foundations

### Color Palette (Tailwind CSS)
- **Primary Accent**: Slate / Indigo (`indigo-600` primary buttons, `indigo-700` hover, `indigo-50` active background highlights)
- **Neutral Dark**: `slate-900` for primary typography, `slate-800` for cards and headings
- **Neutral Muted**: `slate-500` for secondary text and descriptions
- **Neutral Light**: `slate-50` for page background, `white` for cards and modals
- **Border / Divider**: `slate-200`
- **Status Indicators**:
  - `Pending`: Amber (`bg-amber-100 text-amber-800 border-amber-200`)
  - `Confirmed`: Blue (`bg-blue-100 text-blue-800 border-blue-200`)
  - `Shipped`: Purple (`bg-purple-100 text-purple-800 border-purple-200`)
  - `Delivered`: Emerald (`bg-emerald-100 text-emerald-800 border-emerald-200`)
  - `Cancelled`: Rose (`bg-rose-100 text-rose-800 border-rose-200`)

### Responsive Breakpoints
- **Mobile** (`< 640px`): Single-column cards, full-width inputs, hamburger slide-over menu.
- **Tablet** (`640px - 1024px`): 2-column product grid, top navigation with search bar, compact table views.
- **Desktop** (`> 1024px`): 3 to 4-column product grid, permanent admin sidebar, sticky order summary card.

---

## 🧭 Page Routes & View Hierarchy

### Public & Customer Routes (`MainLayout.jsx`)

```text
/                      ➔ Home.jsx (Hero banner, featured categories & top products)
/products              ➔ Products.jsx (Search input, category pills, responsive grid)
/products/:id          ➔ ProductDetails.jsx (Product image, specs, stock counter, Add to Cart)
/cart                  ➔ Cart.jsx (Item rows, quantity stepper, subtotal, Checkout button)
/checkout              ➔ Checkout.jsx (Protected: COD shipping form & order summary)
/my-orders             ➔ MyOrders.jsx (Protected: List of customer orders & real-time statuses)
/login                 ➔ Login.jsx (Email & password credentials form)
/register              ➔ Register.jsx (Name, email, password & confirm password form)
```

### Admin Routes (`AdminLayout.jsx` with `AdminRoute` Guard)

```text
/admin                 ➔ admin/Dashboard.jsx (Quick statistics & shortcut links)
/admin/categories      ➔ admin/Categories.jsx (Category table, Add/Edit modal, Delete dialog)
/admin/products        ➔ admin/Products.jsx (Product table, Add/Edit modal, Delete dialog)
/admin/orders          ➔ admin/Orders.jsx (Customer orders table, Status change dropdown)
```

---

## 🖼️ Page-by-Page Specifications

### 1. Home Page (`Home.jsx`)
- **Hero Section**: Minimalist banner with call-to-action ("Explore Collection" pointing to `/products`).
- **Category Highlights**: Interactive circular or card shortcuts for featured categories (`Electronics`, `Fashion`, `Shoes`).
- **Trending Products**: Top 4–8 products rendered with `ProductCard`.

---

### 2. Products Catalog (`Products.jsx`)
- **Search Header**:
  - Full-width responsive search bar with instant debounced input.
  - Category Filter Pill Tabs:
    ```text
    [ All ]   [ Electronics ]   [ Fashion ]   [ Shoes ]
    ```
    Active pill highlighted in solid `indigo-600` with white text.
- **Product Grid**:
  - Responsive layout: `grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6`.
  - Cards contain:
    - Product thumbnail with subtle zoom on hover.
    - Category badge.
    - Product title (clamped to 2 lines).
    - Price formatted in local currency (e.g., `$199.99`).
    - Stock badge (`In Stock` or `Out of Stock`).
    - Primary "Add to Cart" button (disabled if `stock === 0`).
- **Empty State**: Friendly illustration and message when search or filter matches zero products.

---

### 3. Product Details (`ProductDetails.jsx`)
- **Two-Column Layout** (on desktop):
  - Left column: High-resolution product image with rounded corners.
  - Right column:
    - Category breadcrumb.
    - Full product name and description.
    - Prominent price.
    - Available stock indicator (e.g. `15 units available`).
    - Quantity selector (`-` and `+` buttons capped between 1 and available stock).
    - "Add to Cart" button with instant toast confirmation.

---

### 4. Shopping Cart (`Cart.jsx`)
- **Cart Item Rows**:
  - Thumbnail, Product Name, Unit Price, and Item Subtotal.
  - Interactive Quantity Stepper (`-` decrement, `+` increment capped by available stock).
  - Trash can icon to remove item.
- **Cart Summary Card**:
  - Items Subtotal.
  - Shipping fee (Displayed as `FREE (Cash on Delivery)`).
  - Total Payable Amount.
  - "Proceed to Checkout" primary action button.
- **Empty State**: Icon with "Your cart is empty" and a button returning to `/products`.

---

### 5. Checkout Page (`Checkout.jsx`)
- **Requirement**: Customer authentication required (redirects to `/login` if unauthenticated).
- **Shipping Address Form**:
  - Name (Full Name)
  - Phone Number (valid contact number)
  - Address (Street, Apartment / House No.)
  - City
  - Pincode (Postal code)
- **Payment Method**:
  - Radio card locked to **Cash on Delivery (COD)**.
- **Order Placement**:
  - Button labeled **"Place Order (Cash on Delivery)"**.
  - On click:
    - Frontend validates form inputs.
    - Calls `POST /api/orders`.
    - On success: Empties `CartContext`, triggers success toast, and navigates to `/my-orders`.

---

### 6. My Orders (`MyOrders.jsx`)
- **Card-Based Order History**:
  - Header: Order ID, Placement Date/Time, and Status Badge (`Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`).
  - Item List: Thumbnails, product names, quantities, and purchased prices.
  - Shipping Details: Recipient name, contact phone, and delivery address.
  - Order Total: Highlighted total amount payable upon delivery.

---

### 7. Admin Dashboard & Sub-Views (`admin/*`)

#### Admin Sidebar (`components/admin/AdminSidebar.jsx`)
- Sticky navigation links:
  - 📊 Dashboard Overview
  - 🏷️ Categories
  - 📦 Products
  - 🛒 Orders
  - 🚪 Logout

#### Category Management (`admin/Categories.jsx`)
- "Add Category" button triggering creation modal.
- Responsive table: Name, Description, Created Date, Actions (Edit, Delete).
- Confirmation dialog modal for Delete actions.

#### Product Management (`admin/Products.jsx`)
- "Add Product" button triggering creation modal with fields:
  - Name, Description, Price, Image URL, Category selector, Stock count.
- Responsive table with product thumbnail, title, category, price, stock counter, and action buttons.
- Confirmation dialog modal for Delete actions.

#### Order Management (`admin/Orders.jsx`)
- Responsive table displaying: Order ID, Customer Name/Email, Items Count, Total Amount, Date, Status.
- Inline status selector dropdown or modal to transition statuses:
  `Pending` ➔ `Confirmed` ➔ `Shipped` ➔ `Delivered` ➔ `Cancelled`.

---

## 🔔 Feedback & Interactive States

1. **Toast Notifications**: Integrated with `react-hot-toast` for positive (green), warning (amber), and failure (red) feedback.
2. **Loading States**: Tailwind animated spinners (`animate-spin`) or pulse skeleton blocks during data fetching.
3. **Delete Confirmation Dialogs**: Modal intercepting any delete action requiring deliberate confirmation before issuing `DELETE` API calls.
