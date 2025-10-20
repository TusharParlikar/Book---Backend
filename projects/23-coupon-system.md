# Project 23: Coupon System API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md), [B10 - E-Commerce Backend](../chapters/B10_ECOMMERCE_BACKEND.md)

---

## 📝 Description

Build a discount coupon system with validation, usage limits, and expiry.

**What You'll Learn:**
- Coupon code generation
- Validation logic
- Usage tracking
- Discount calculations
- Expiry management

---

## 🎯 Project Goals

- ✅ Generate unique coupon codes
- ✅ Apply discounts (percentage/fixed)
- ✅ Usage limits and expiry
- ✅ Validation checks
- ✅ React coupon UI

---

## 🗺️ Roadmap

### Step 1: Coupon Model
**Todo:**
- [ ] code (unique string)
- [ ] discountType (percentage/fixed)
- [ ] discountValue
- [ ] minOrderValue (minimum cart amount)
- [ ] maxDiscount (for percentage coupons)
- [ ] usageLimit, usedCount
- [ ] expiryDate
- [ ] isActive

---

### Step 2: Coupon Generation
**Todo:**
- [ ] `POST /api/coupons` - Create coupon
- [ ] Generate random code (e.g., SAVE20, WINTER2025)
- [ ] Or accept custom code
- [ ] Validate uniqueness

**Code Generation:**
```javascript
function generateCode() {
  return 'SAVE' + Math.random().toString(36).substr(2, 8).toUpperCase();
}
```

---

### Step 3: Validation
**Todo:**
- [ ] `POST /api/coupons/validate`
  - Body: { code, cartValue }
- [ ] Check: exists, active, not expired
- [ ] Check: cartValue >= minOrderValue
- [ ] Check: usageLimit not exceeded
- [ ] Return: valid (boolean), discount amount

**Validation Steps:**
```
1. Find coupon by code
2. Check isActive
3. Check expiryDate > now
4. Check usedCount < usageLimit
5. Check cartValue >= minOrderValue
6. Calculate discount
7. Return result
```

---

### Step 4: Apply Coupon
**Todo:**
- [ ] `POST /api/coupons/apply`
  - Body: { code, cartValue, userId }
- [ ] Validate coupon
- [ ] Calculate final discount
- [ ] Increment usedCount
- [ ] Log usage (who, when, how much saved)

---

### Step 5: Discount Calculation
**Todo:**
- [ ] **Percentage:** 
  - discount = cartValue * (percentage / 100)
  - Apply maxDiscount cap
- [ ] **Fixed:** 
  - discount = fixed amount
  - Ensure discount ≤ cartValue

**Example:**
```
Cart: $100
Coupon: 20% off, max discount $15
Calculation: $100 * 0.20 = $20
Applied: $15 (capped)
Final: $100 - $15 = $85
```

---

### Step 6: Usage Tracking
**Todo:**
- [ ] Create Usage model:
  - couponId, userId, orderValue
  - discountApplied, usedAt
- [ ] Track all coupon applications
- [ ] `GET /api/coupons/:id/usage` - Usage history

---

### Step 7: Management Routes
**Todo:**
- [ ] `GET /api/coupons` - All coupons (admin)
- [ ] `PUT /api/coupons/:id` - Update coupon
- [ ] `DELETE /api/coupons/:id` - Delete
- [ ] `PATCH /api/coupons/:id/toggle` - Activate/deactivate

---

### Step 8: React Frontend
**Todo:**
- [ ] Coupon input field (cart page)
- [ ] Apply button
- [ ] Show discount applied
- [ ] Display final amount
- [ ] Admin: coupon management dashboard
- [ ] Create coupon form
- [ ] List all coupons with stats

---

## 💡 Hints & Tips

### Check Expiry:
```javascript
if (new Date(coupon.expiryDate) < new Date()) {
  return { valid: false, error: 'Coupon expired' };
}
```

### Calculate Discount:
```javascript
if (coupon.discountType === 'percentage') {
  let discount = cartValue * (coupon.discountValue / 100);
  discount = Math.min(discount, coupon.maxDiscount);
  return discount;
} else {
  return Math.min(coupon.discountValue, cartValue);
}
```

### Prevent Double Usage:
Check if user already used this coupon (if one-per-user rule)

---

## 📚 Concepts

- Coupon validation
- Discount calculations
- Usage tracking
- Code generation
- Business logic

---

**Next:** [Project 24: Password Reset System](24-password-reset-system.md)
