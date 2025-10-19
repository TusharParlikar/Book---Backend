# Chapter 11: Advanced Data Modeling - E-commerce

## 11.1 User Model (with Avatar and Tokens)

```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true
  },
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  avatar: {
    type: String,
    default: null
  },
  coverImage: {
    type: String,
    default: null
  },
  watchHistory: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Product'
  }],
  refreshToken: {
    type: String,
    select: false
  }
}, { timestamps: true });

const User = mongoose.model('User', userSchema);
module.exports = User;
```

## 11.2 Category Model

```javascript
const categorySchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true
  },
  description: String,
  icon: String
}, { timestamps: true });

const Category = mongoose.model('Category', categorySchema);
module.exports = Category;
```

## 11.3 Product Model (with References)

```javascript
const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  description: String,
  price: {
    type: Number,
    required: true,
    min: 0
  },
  stock: {
    type: Number,
    default: 0
  },
  images: [String],
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category'
  },
  owner: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
}, { timestamps: true });

const Product = mongoose.model('Product', productSchema);
module.exports = Product;
```

## 11.4 Order Model (with Enums and Embedded Schema)

```javascript
const orderSchema = new mongoose.Schema({
  customer: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  items: [{
    product: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Product'
    },
    quantity: Number,
    price: Number
  }],
  totalPrice: Number,
  shippingAddress: {
    street: String,
    city: String,
    country: String,
    zipCode: String
  },
  status: {
    type: String,
    enum: ['pending', 'processing', 'shipped', 'delivered', 'cancelled'],
    default: 'pending'
  }
}, { timestamps: true });

const Order = mongoose.model('Order', orderSchema);
module.exports = Order;
```

---

## 🎯 Next: Chapter 12 - Hospital Management Models

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 10: Basic Data Models](./10_BASIC_DATA_MODELS.md) | [📖 Course Index](../INDEX.md) | [Chapter 12: Hospital System Models →](./12_HOSPITAL_MODELS.md) |
