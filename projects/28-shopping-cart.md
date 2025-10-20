# Project 28: Shopping Cart System

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 5-6 hours  
**Related Chapter:** [B10 - E-Commerce Backend](../chapters/B10_ECOMMERCE_BACKEND.md)

---

## 📝 Description

Build a complete shopping cart with quantity management, pricing calculations, and persistence.

**What You'll Learn:**
- Cart state management
- Inventory validation
- Price calculations
- Cart persistence
- Checkout preparation

---

## 🎯 Project Goals

- ✅ Add/remove/update cart items
- ✅ Calculate totals with tax and shipping
- ✅ Stock validation
- ✅ Cart persistence (logged in + guest)
- ✅ React cart UI

---

## 🗺️ Roadmap

### Step 1: Cart Model

**For Logged Users:**
- [ ] userId, items array
- [ ] Each item: { productId, quantity, price }
- [ ] totalAmount, updatedAt

**For Guest Users:**
- [ ] Use session ID or localStorage
- [ ] Convert to user cart on login

---

### Step 2: Add to Cart
**Todo:**
- [ ] `POST /api/cart/add`
  - Body: { productId, quantity }
  - Check product exists
  - Check stock availability
  - If item exists in cart, increase quantity
  - Else add new item
  - Recalculate total
  - Return updated cart

---

### Step 3: Update Quantity
**Todo:**
- [ ] `PUT /api/cart/items/:productId`
  - Body: { quantity }
  - Validate: quantity > 0 and ≤ stock
  - Update item quantity
  - Recalculate total
  - Return updated cart

---

### Step 4: Remove from Cart
**Todo:**
- [ ] `DELETE /api/cart/items/:productId`
  - Remove item from cart
  - Recalculate total
  - Return updated cart

---

### Step 5: Get Cart
**Todo:**
- [ ] `GET /api/cart`
  - Return full cart with populated product details
  - Include: name, image, price, quantity, subtotal
  - Calculate: subtotal, tax, shipping, total

---

### Step 6: Price Calculations
**Todo:**
- [ ] Subtotal: sum of (price * quantity) for all items
- [ ] Tax: subtotal * tax rate (e.g., 10%)
- [ ] Shipping: 
  - Free if subtotal > $50
  - Else $5 flat rate
- [ ] Total: subtotal + tax + shipping

---

### Step 7: Stock Validation
**Todo:**
- [ ] Before adding/updating, check product stock
- [ ] If requested quantity > available stock, return error
- [ ] Decrement stock on order placement (not on add to cart)

---

### Step 8: Merge Guest Cart on Login
**Todo:**
- [ ] When user logs in, check localStorage cart
- [ ] Merge with user's existing cart
- [ ] Handle duplicates (add quantities)
- [ ] Clear localStorage cart

---

### Step 9: Cart Expiry
**Todo:**
- [ ] Clear abandoned carts after 30 days
- [ ] Or reserve items for 15 minutes during checkout

---

### Step 10: React Frontend
**Todo:**
- [ ] Add to cart button (product page)
- [ ] Cart icon with item count badge
- [ ] Cart page:
  - List all items
  - Quantity selectors (+ / -)
  - Remove button
  - Price breakdown
  - Proceed to checkout button

---

## 💡 Hints & Tips

### Check Stock:
```javascript
if (product.stock < quantity) {
  return res.status(400).json({ error: 'Insufficient stock' });
}
```

### Calculate Total:
```javascript
const subtotal = cart.items.reduce((sum, item) => 
  sum + (item.price * item.quantity), 0
);
const tax = subtotal * 0.10; // 10% tax
const shipping = subtotal > 50 ? 0 : 5;
const total = subtotal + tax + shipping;
```

### Update Existing Item:
```javascript
const existingItem = cart.items.find(i => i.productId === productId);
if (existingItem) {
  existingItem.quantity += quantity;
} else {
  cart.items.push({ productId, quantity, price });
}
```

---

## 📚 Concepts

- Cart management
- Inventory validation
- Price calculations
- Session handling
- State persistence

---

**Next:** [Project 29: Order Management System](29-order-management.md)
