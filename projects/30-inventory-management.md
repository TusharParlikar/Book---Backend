# Project 30: Inventory Management API

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B10 - E-Commerce Backend](../chapters/B10_ECOMMERCE_BACKEND.md)

---

## 📝 Description

Build a warehouse inventory system with stock tracking, low stock alerts, and reorder management.

**What You'll Learn:**
- Inventory tracking
- Stock level monitoring
- Reorder point calculations
- Warehouse management
- Stock movement logs

---

## 🎯 Project Goals

- ✅ Track product stock levels
- ✅ Low stock alerts
- ✅ Stock movement history
- ✅ Reorder management
- ✅ React inventory dashboard

---

## 🗺️ Roadmap

### Step 1: Inventory Model
**Todo:**
- [ ] Product Inventory:
  - productId, sku (stock keeping unit)
  - currentStock, reservedStock
  - reorderPoint, reorderQuantity
  - warehouseLocation
  - lastRestocked

---

### Step 2: Stock Operations
**Todo:**
- [ ] `POST /api/inventory/restock`
  - Body: { productId, quantity, source }
  - Increase currentStock
  - Log movement
- [ ] `POST /api/inventory/reserve`
  - Reserve stock for pending orders
  - Increase reservedStock
  - Decrease currentStock
- [ ] `POST /api/inventory/release`
  - Release reserved stock (order cancelled)

---

### Step 3: Stock Levels
**Todo:**
- [ ] Available Stock = currentStock - reservedStock
- [ ] `GET /api/inventory/:productId` - Get stock info
- [ ] Calculate:
  - Available for sale
  - Reserved for orders
  - Total in warehouse

---

### Step 4: Low Stock Alerts
**Todo:**
- [ ] `GET /api/inventory/low-stock`
- [ ] Return products where: currentStock ≤ reorderPoint
- [ ] Auto-generate reorder suggestions

**Reorder Logic:**
```
if (currentStock <= reorderPoint) {
  suggestedOrder = reorderQuantity;
  alert = true;
}
```

---

### Step 5: Stock Movement Log
**Todo:**
- [ ] Movement Model:
  - productId, type (in/out), quantity
  - reason (sale, return, restock, damaged)
  - timestamp, userId
- [ ] `POST /api/inventory/movement` - Log movement
- [ ] `GET /api/inventory/:productId/movements` - History

---

### Step 6: Bulk Updates
**Todo:**
- [ ] `POST /api/inventory/bulk-update`
  - Body: array of { productId, quantity }
  - Update multiple products at once
  - Used for CSV imports

---

### Step 7: Stock Audit
**Todo:**
- [ ] `GET /api/inventory/audit`
- [ ] Compare: expected stock vs actual
- [ ] Identify discrepancies
- [ ] Generate audit report

---

### Step 8: Reorder Management
**Todo:**
- [ ] `GET /api/inventory/reorder-suggestions`
- [ ] Based on: reorder point, sales velocity
- [ ] `POST /api/inventory/create-purchase-order`
  - Generate PO for supplier
  - Track: ordered, received

---

### Step 9: Stock Alerts
**Todo:**
- [ ] Email notifications:
  - Low stock alert
  - Out of stock alert
  - Restock needed
- [ ] Admin dashboard alerts

---

### Step 10: React Dashboard
**Todo:**
- [ ] Inventory overview (total products, total value)
- [ ] Low stock alerts (red badge)
- [ ] Stock levels table
- [ ] Restock form
- [ ] Stock movement history
- [ ] Reorder suggestions list

---

## 💡 Hints & Tips

### Reserve Stock (On Order):
```javascript
product.reservedStock += orderQuantity;
product.currentStock -= orderQuantity;
```

### Release Stock (Order Cancelled):
```javascript
product.currentStock += orderQuantity;
product.reservedStock -= orderQuantity;
```

### Low Stock Check:
```javascript
const lowStockProducts = products.filter(p => 
  (p.currentStock - p.reservedStock) <= p.reorderPoint
);
```

### Calculate Total Inventory Value:
```javascript
const totalValue = products.reduce((sum, p) => 
  sum + (p.currentStock * p.price), 0
);
```

---

## 📚 Concepts

- Inventory management
- Stock reservation
- Movement tracking
- Reorder automation
- Audit systems

---

**Next:** [Project 31: Social Media Backend](31-social-media-backend.md)
