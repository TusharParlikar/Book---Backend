# Project 29: Order Management System

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 5-6 hours  
**Related Chapter:** [B10 - E-Commerce Backend](../chapters/B10_ECOMMERCE_BACKEND.md)

---

## 📝 Description

Build a complete order processing system with status tracking and order history.

**What You'll Learn:**
- Order lifecycle management
- Status workflows
- Payment integration basics
- Order tracking
- Invoice generation

---

## 🎯 Project Goals

- ✅ Create orders from cart
- ✅ Order status workflow
- ✅ Order history and details
- ✅ Cancel/return orders
- ✅ React order tracking UI

---

## 🗺️ Roadmap

### Step 1: Order Model
**Todo:**
- [ ] Order structure:
  - orderNumber (unique)
  - userId, items (from cart)
  - totalAmount, status
  - shippingAddress, billingAddress
  - paymentMethod, paymentStatus
  - createdAt, updatedAt
  - trackingNumber (optional)

---

### Step 2: Create Order (Checkout)
**Todo:**
- [ ] `POST /api/orders`
  - Body: { shippingAddress, paymentMethod }
  - Get user's cart
  - Validate: cart not empty, items in stock
  - Generate unique order number
  - Create order with status: 'pending'
  - Decrement product stock
  - Clear cart
  - Send order confirmation email
  - Return order details

**Order Number Generation:**
```javascript
const orderNumber = 'ORD' + Date.now() + Math.random().toString(36).substr(2, 5).toUpperCase();
```

---

### Step 3: Order Status Workflow
**Todo:**
- [ ] Status flow:
  - pending → processing → shipped → delivered
  - Can cancel before shipping
  - Can return after delivery (within 30 days)
- [ ] `PATCH /api/orders/:id/status`
  - Body: { status }
  - Validate status transition
  - Update order
  - Send notification email

**Valid Transitions:**
```
pending → processing ✅
pending → cancelled ✅
processing → shipped ✅
shipped → delivered ✅
delivered → returned (within 30 days) ✅
```

---

### Step 4: Get Order History
**Todo:**
- [ ] `GET /api/orders` - User's all orders
- [ ] `GET /api/orders/:id` - Single order details
- [ ] Sort by: newest first
- [ ] Include: items with product details

---

### Step 5: Cancel Order
**Todo:**
- [ ] `POST /api/orders/:id/cancel`
- [ ] Check: status is pending or processing
- [ ] Restore product stock
- [ ] Update status to 'cancelled'
- [ ] Process refund (if paid)
- [ ] Send cancellation email

---

### Step 6: Return Order
**Todo:**
- [ ] `POST /api/orders/:id/return`
  - Body: { reason, items }
- [ ] Check: status is delivered
- [ ] Check: within 30 days of delivery
- [ ] Create return request
- [ ] Update status to 'return_requested'
- [ ] Admin approval flow

---

### Step 7: Invoice Generation
**Todo:**
- [ ] `GET /api/orders/:id/invoice`
- [ ] Generate PDF invoice
- [ ] Include: order details, items, totals
- [ ] Return PDF or download link

**Use:** `pdfkit` or `puppeteer`

---

### Step 8: Order Tracking
**Todo:**
- [ ] `GET /api/orders/:id/track`
- [ ] Return timeline:
  - Order placed (date)
  - Processing (date)
  - Shipped (date, tracking number)
  - Out for delivery (date)
  - Delivered (date)

---

### Step 9: Admin Orders Management
**Todo:**
- [ ] `GET /api/admin/orders` - All orders
- [ ] Filter by: status, date, customer
- [ ] Update order status
- [ ] View order details
- [ ] Export orders to CSV

---

### Step 10: React Frontend
**Todo:**
- [ ] Checkout page (create order)
- [ ] Order confirmation page
- [ ] Order history page
- [ ] Order details page with tracking
- [ ] Cancel button (if applicable)
- [ ] Return request form
- [ ] Invoice download button

---

## 💡 Hints & Tips

### Stock Decrement:
```javascript
for (const item of cart.items) {
  const product = await Product.findById(item.productId);
  product.stock -= item.quantity;
  await product.save();
}
```

### Validate Status Transition:
```javascript
const validTransitions = {
  pending: ['processing', 'cancelled'],
  processing: ['shipped', 'cancelled'],
  shipped: ['delivered'],
  delivered: ['returned']
};

if (!validTransitions[currentStatus].includes(newStatus)) {
  return res.status(400).json({ error: 'Invalid status transition' });
}
```

### Check Return Window:
```javascript
const deliveryDate = new Date(order.deliveredAt);
const now = new Date();
const daysSinceDelivery = (now - deliveryDate) / (1000 * 60 * 60 * 24);

if (daysSinceDelivery > 30) {
  return res.status(400).json({ error: 'Return window expired' });
}
```

---

## 📚 Concepts

- Order lifecycle
- Status workflows
- Stock management
- Transaction safety
- Invoice generation
- Order tracking

---

**Next:** [Project 30: Inventory Management API](30-inventory-management.md)
