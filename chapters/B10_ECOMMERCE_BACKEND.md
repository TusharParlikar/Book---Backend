# B10: E-Commerce Backend 🛍️

> **Real-World System: Products, Orders, Payments**

**Chapter Level:** Intermediate  
**Time Required:** 3-4 hours  
**Prerequisites:** Completed B01-B09  

---

## Data Models

```javascript
// User (from B09)
// ├─ name, email, password, cart, orders

// Product
// ├─ name, price, description, stock, category, image

// Order
// ├─ user (ref), items (array of products), total, status

// Cart Item
// ├─ user (ref), product (ref), quantity

// ============================================================
// FILE: models/Product.js
// ============================================================

const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true, min: 0 },
  description: String,
  image: String,
  category: String,
  stock: { type: Number, default: 0 },
  rating: { type: Number, default: 0 }
}, { timestamps: true });

module.exports = mongoose.model('Product', productSchema);


// ============================================================
// FILE: models/Order.js
// ============================================================

const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  items: [{
    product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
    quantity: Number,
    price: Number
  }],
  total: Number,
  status: { type: String, enum: ['pending', 'processing', 'shipped', 'delivered'], default: 'pending' },
  shippingAddress: String
}, { timestamps: true });

module.exports = mongoose.model('Order', orderSchema);


// ============================================================
// FILE: controllers/productController.js
// ============================================================

const Product = require('../models/Product');

const getAllProducts = async (req, res) => {
  try {
    const { category, minPrice, maxPrice } = req.query;
    let filter = {};
    if (category) filter.category = category;
    if (minPrice || maxPrice) {
      filter.price = {};
      if (minPrice) filter.price.$gte = minPrice;
      if (maxPrice) filter.price.$lte = maxPrice;
    }
    const products = await Product.find(filter);
    res.json({ success: true, data: products });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const createProduct = async (req, res) => {
  try {
    const product = await Product.create(req.body);
    res.status(201).json({ success: true, data: product });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

module.exports = { getAllProducts, createProduct };


// ============================================================
// FILE: controllers/orderController.js
// ============================================================

const Order = require('../models/Order');
const Product = require('../models/Product');

const createOrder = async (req, res) => {
  try {
    const { items, shippingAddress } = req.body;
    
    // Calculate total
    let total = 0;
    const orderItems = [];
    
    for (const item of items) {
      const product = await Product.findById(item.productId);
      if (!product || product.stock < item.quantity) {
        return res.status(400).json({ success: false, message: 'Stock unavailable' });
      }
      orderItems.push({
        product: product._id,
        quantity: item.quantity,
        price: product.price
      });
      total += product.price * item.quantity;
      
      // Update stock
      product.stock -= item.quantity;
      await product.save();
    }
    
    const order = await Order.create({
      user: req.user.userId,
      items: orderItems,
      total,
      shippingAddress
    });
    
    res.status(201).json({ success: true, data: order });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const getUserOrders = async (req, res) => {
  try {
    const orders = await Order.find({ user: req.user.userId }).populate('items.product');
    res.json({ success: true, data: orders });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

module.exports = { createOrder, getUserOrders };
```

---

| Previous | Index | Next |
|----------|-------|------|
| [B09: Complete Auth Flow](B09_COMPLETE_AUTH_FLOW.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B11: File Uploads](B11_FILE_UPLOADS.md) |
