# Chapter 16: Razorpay Payment Integration 💳

**Learn to Accept Real Payments in Your Applications**

---

## 📚 What You'll Learn

- Setting up Razorpay merchant account
- Creating and verifying payment orders
- Handling payment webhooks and callbacks
- Refund management and payment tracking
- Securing payment flows with signatures
- Testing with Razorpay test mode
- Building complete checkout flow

**Time Required:** 6-8 hours  
**Difficulty:** Advanced  
**Prerequisites:** B01-B10 (Auth, Database, E-Commerce basics)

---

## 🎯 Learning Objectives

By the end of this chapter, you will:

✅ Integrate Razorpay SDK in Node.js  
✅ Create secure payment orders  
✅ Verify payment signatures for security  
✅ Handle payment webhooks for automation  
✅ Implement refund logic  
✅ Store payment history in database  
✅ Build checkout page with React  

---

## 💡 Why Razorpay?

Razorpay is India's leading payment gateway that supports:

- **Credit/Debit Cards** (Visa, Mastercard, RuPay)
- **Net Banking** (All major banks)
- **UPI** (Google Pay, PhonePe, Paytm)
- **Wallets** (Paytm, Mobikwik, etc.)
- **EMI Options**
- **International Cards**

**Benefits:**
- ✅ Easy integration with Node.js
- ✅ Automatic settlement to bank account
- ✅ Real-time payment tracking
- ✅ Test mode for development
- ✅ Comprehensive webhooks
- ✅ Refund API support

---

## 🔧 Setup Razorpay Account

### Step 1: Create Account

1. Go to [https://razorpay.com](https://razorpay.com)
2. Sign up for free
3. Complete KYC (for live mode)
4. Get API credentials:
   - **Key ID** (starts with `rzp_test_` or `rzp_live_`)
   - **Key Secret** (keep this secret!)

### Step 2: Install Razorpay SDK

```bash
npm install razorpay
```

<details>
<summary>💡 Expected Output</summary>

```bash
added 1 package, and audited 2 packages in 1s
found 0 vulnerabilities
```
</details>

### Step 3: Environment Variables

Create `.env` file:

```env
RAZORPAY_KEY_ID=rzp_test_YOUR_KEY_ID
RAZORPAY_KEY_SECRET=YOUR_KEY_SECRET
```

---

## 🏗️ Project: E-Commerce Checkout System

We'll build a complete payment system for an e-commerce store.

### Database Schema

**Order Model:**

```javascript
const orderSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  items: [{
    product: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Product',
      required: true
    },
    quantity: {
      type: Number,
      required: true,
      min: 1
    },
    price: {
      type: Number,
      required: true
    }
  }],
  totalAmount: {
    type: Number,
    required: true
  },
  currency: {
    type: String,
    default: 'INR'
  },
  status: {
    type: String,
    enum: ['pending', 'processing', 'paid', 'failed', 'refunded'],
    default: 'pending'
  },
  // Razorpay specific fields
  razorpayOrderId: String,
  razorpayPaymentId: String,
  razorpaySignature: String,
  paymentMethod: String, // card, netbanking, upi, wallet
  paidAt: Date,
  refundId: String,
  refundedAt: Date
}, { timestamps: true });
```

---

## 💳 Creating Payment Orders

### Step 1: Initialize Razorpay

Create `utils/razorpay.js`:

```javascript
const Razorpay = require('razorpay');

const razorpayInstance = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET,
});

module.exports = razorpayInstance;
```

### Step 2: Create Order Endpoint

**Route:** `POST /api/orders/create`

**Controller Logic:**

```javascript
// Step 1: Calculate total from cart
const cart = await Cart.findOne({ user: req.user._id }).populate('items.product');

if (!cart || cart.items.length === 0) {
  return res.status(400).json({ message: 'Cart is empty' });
}

let totalAmount = 0;
cart.items.forEach(item => {
  totalAmount += item.product.price * item.quantity;
});

// Step 2: Create Razorpay order
const options = {
  amount: totalAmount * 100, // Amount in paise (₹100 = 10000 paise)
  currency: 'INR',
  receipt: `receipt_${Date.now()}`,
  notes: {
    userId: req.user._id.toString(),
    cartId: cart._id.toString()
  }
};

const razorpayOrder = await razorpayInstance.orders.create(options);

// Step 3: Save order in database
const order = await Order.create({
  user: req.user._id,
  items: cart.items,
  totalAmount: totalAmount,
  razorpayOrderId: razorpayOrder.id,
  status: 'pending'
});

// Step 4: Send order details to frontend
res.json({
  success: true,
  orderId: order._id,
  razorpayOrderId: razorpayOrder.id,
  amount: razorpayOrder.amount,
  currency: razorpayOrder.currency,
  keyId: process.env.RAZORPAY_KEY_ID // Send to frontend for checkout
});
```

---

## ✅ Verifying Payment

**CRITICAL:** Always verify payment signature on backend to prevent fraud.

### Route: `POST /api/orders/verify`

```javascript
const crypto = require('crypto');

// Receive from frontend after payment
const { razorpayOrderId, razorpayPaymentId, razorpaySignature } = req.body;

// Step 1: Generate signature
const generatedSignature = crypto
  .createHmac('sha256', process.env.RAZORPAY_KEY_SECRET)
  .update(`${razorpayOrderId}|${razorpayPaymentId}`)
  .digest('hex');

// Step 2: Compare signatures
if (generatedSignature !== razorpaySignature) {
  return res.status(400).json({ 
    success: false, 
    message: 'Payment verification failed' 
  });
}

// Step 3: Update order status
const order = await Order.findOneAndUpdate(
  { razorpayOrderId },
  {
    status: 'paid',
    razorpayPaymentId,
    razorpaySignature,
    paidAt: new Date()
  },
  { new: true }
);

// Step 4: Clear user's cart
await Cart.findOneAndUpdate(
  { user: order.user },
  { items: [] }
);

// Step 5: Reduce product stock
for (const item of order.items) {
  await Product.findByIdAndUpdate(item.product, {
    $inc: { stock: -item.quantity }
  });
}

res.json({
  success: true,
  message: 'Payment verified successfully',
  order
});
```

---

## 🔔 Handling Webhooks

Razorpay sends webhooks for events like payment success, failure, refunds.

### Why Webhooks?

- User closes payment page before completion
- Payment succeeds but network fails
- Delayed payment confirmations
- Automatic order updates

### Setup Webhook Endpoint

**Route:** `POST /api/webhooks/razorpay`

```javascript
const validateWebhookSignature = (req, res, next) => {
  const signature = req.headers['x-razorpay-signature'];
  const webhookSecret = process.env.RAZORPAY_WEBHOOK_SECRET;

  const generatedSignature = crypto
    .createHmac('sha256', webhookSecret)
    .update(JSON.stringify(req.body))
    .digest('hex');

  if (generatedSignature !== signature) {
    return res.status(400).json({ message: 'Invalid signature' });
  }

  next();
};

// Webhook handler
router.post('/webhooks/razorpay', 
  express.raw({ type: 'application/json' }), // Important: raw body needed
  validateWebhookSignature,
  async (req, res) => {
    const event = req.body.event;
    const payment = req.body.payload.payment.entity;

    switch (event) {
      case 'payment.captured':
        // Payment successful
        await Order.findOneAndUpdate(
          { razorpayOrderId: payment.order_id },
          { 
            status: 'paid',
            razorpayPaymentId: payment.id,
            paymentMethod: payment.method,
            paidAt: new Date(payment.created_at * 1000)
          }
        );
        break;

      case 'payment.failed':
        // Payment failed
        await Order.findOneAndUpdate(
          { razorpayOrderId: payment.order_id },
          { status: 'failed' }
        );
        break;

      case 'refund.created':
        // Refund initiated
        await Order.findOneAndUpdate(
          { razorpayPaymentId: payment.id },
          { 
            status: 'refunded',
            refundId: req.body.payload.refund.entity.id,
            refundedAt: new Date()
          }
        );
        break;
    }

    res.json({ status: 'ok' });
  }
);
```

**Configure in Razorpay Dashboard:**
1. Go to Settings → Webhooks
2. Add your webhook URL: `https://yourdomain.com/api/webhooks/razorpay`
3. Select events to track
4. Copy webhook secret to `.env`

---

## 💰 Refund Management

### Full Refund

```javascript
// Route: POST /api/orders/:orderId/refund
const order = await Order.findById(req.params.orderId);

if (order.status !== 'paid') {
  return res.status(400).json({ message: 'Order not paid yet' });
}

// Create refund
const refund = await razorpayInstance.payments.refund(
  order.razorpayPaymentId,
  {
    amount: order.totalAmount * 100, // Full refund
    notes: {
      reason: req.body.reason || 'Customer requested'
    }
  }
);

// Update order
order.status = 'refunded';
order.refundId = refund.id;
order.refundedAt = new Date();
await order.save();

// Restore product stock
for (const item of order.items) {
  await Product.findByIdAndUpdate(item.product, {
    $inc: { stock: item.quantity }
  });
}

res.json({ success: true, refund });
```

### Partial Refund

```javascript
// Refund specific amount
const refund = await razorpayInstance.payments.refund(
  order.razorpayPaymentId,
  {
    amount: 5000, // ₹50 in paise
    notes: {
      reason: 'Partial refund for damaged item'
    }
  }
);
```

---

## 🎨 React Frontend Integration

### Step 1: Load Razorpay Script

```jsx
import { useEffect } from 'react';

function CheckoutPage() {
  useEffect(() => {
    const script = document.createElement('script');
    script.src = 'https://checkout.razorpay.com/v1/checkout.js';
    script.async = true;
    document.body.appendChild(script);
  }, []);

  // Rest of component...
}
```

### Step 2: Create Order and Open Checkout

```jsx
const handlePayment = async () => {
  try {
    // Step 1: Create order on backend
    const response = await fetch('/api/orders/create', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      }
    });

    const data = await response.json();

    // Step 2: Configure Razorpay options
    const options = {
      key: data.keyId,
      amount: data.amount,
      currency: data.currency,
      name: 'Your Store Name',
      description: 'Purchase Products',
      order_id: data.razorpayOrderId,
      
      // Callback on successful payment
      handler: async function (response) {
        // Step 3: Verify payment on backend
        const verifyResponse = await fetch('/api/orders/verify', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${token}`
          },
          body: JSON.stringify({
            razorpayOrderId: response.razorpay_order_id,
            razorpayPaymentId: response.razorpay_payment_id,
            razorpaySignature: response.razorpay_signature
          })
        });

        const verifyData = await verifyResponse.json();

        if (verifyData.success) {
          alert('Payment successful!');
          window.location.href = '/order-success';
        } else {
          alert('Payment verification failed');
        }
      },

      // Prefill user details
      prefill: {
        name: user.name,
        email: user.email,
        contact: user.phone
      },

      // Theme customization
      theme: {
        color: '#3399cc'
      }
    };

    // Step 4: Open Razorpay checkout
    const razorpay = new window.Razorpay(options);
    razorpay.open();

  } catch (error) {
    console.error('Payment error:', error);
    alert('Payment failed. Please try again.');
  }
};
```

---

## 🧪 Testing Payments

### Test Mode Credentials

Use test API keys (starting with `rzp_test_`).

### Test Card Details

**Successful Payment:**
- Card Number: `4111 1111 1111 1111`
- CVV: Any 3 digits
- Expiry: Any future date

**Failed Payment:**
- Card Number: `4111 1111 1111 1234`

**Insufficient Funds:**
- Card Number: `5104 0600 0000 0008`

### Test UPI

- UPI ID: `success@razorpay`
- UPI ID (failure): `failure@razorpay`

---

## 📊 Fetching Payment Details

### Get Payment Info

```javascript
// Get single payment
const payment = await razorpayInstance.payments.fetch(paymentId);

console.log(payment.method); // card, netbanking, upi, wallet
console.log(payment.email);
console.log(payment.contact);
console.log(payment.card_id); // Masked card details
```

### List All Payments

```javascript
// Get payments for last 30 days
const payments = await razorpayInstance.payments.all({
  from: Math.floor(Date.now() / 1000) - 30 * 24 * 60 * 60,
  to: Math.floor(Date.now() / 1000),
  count: 100
});
```

---

## 🔒 Security Best Practices

### 1. Never Expose Secret Key

❌ **Wrong:**
```javascript
// Never send secret to frontend
res.json({ keySecret: process.env.RAZORPAY_KEY_SECRET });
```

✅ **Correct:**
```javascript
// Only send key_id to frontend
res.json({ keyId: process.env.RAZORPAY_KEY_ID });
```

### 2. Always Verify Signatures

```javascript
// ALWAYS verify on backend, never trust frontend
if (generatedSignature !== razorpaySignature) {
  throw new Error('Invalid signature');
}
```

### 3. Use HTTPS in Production

Razorpay requires HTTPS for live payments.

### 4. Implement Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const paymentLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Max 5 payment attempts
  message: 'Too many payment attempts. Please try later.'
});

app.use('/api/orders/create', paymentLimiter);
```

---

## 🎯 Complete Checkout Flow

```
1. User adds items to cart
        ↓
2. User clicks "Checkout"
        ↓
3. Frontend calls POST /api/orders/create
        ↓
4. Backend creates Razorpay order + saves in DB
        ↓
5. Backend returns order details to frontend
        ↓
6. Frontend opens Razorpay checkout modal
        ↓
7. User completes payment
        ↓
8. Razorpay calls handler function with payment details
        ↓
9. Frontend sends payment details to POST /api/orders/verify
        ↓
10. Backend verifies signature
        ↓
11. Backend updates order status to 'paid'
        ↓
12. Backend clears cart + reduces stock
        ↓
13. Frontend redirects to success page
        ↓
14. (Optional) Webhook confirms payment asynchronously
```

---

## 🚀 Advanced Features

### 1. Subscription Payments

```javascript
// Create subscription plan
const plan = await razorpayInstance.plans.create({
  period: 'monthly',
  interval: 1,
  item: {
    name: 'Premium Membership',
    amount: 49900, // ₹499
    currency: 'INR'
  }
});

// Create subscription
const subscription = await razorpayInstance.subscriptions.create({
  plan_id: plan.id,
  customer_notify: 1,
  total_count: 12, // 12 months
  start_at: Math.floor(Date.now() / 1000) + 3600 // Start after 1 hour
});
```

### 2. Payment Links

```javascript
// Create payment link
const paymentLink = await razorpayInstance.paymentLink.create({
  amount: 50000,
  currency: 'INR',
  description: 'Invoice #123',
  customer: {
    name: 'John Doe',
    email: 'john@example.com',
    contact: '+919876543210'
  },
  notify: {
    sms: true,
    email: true
  },
  callback_url: 'https://yoursite.com/payment-success',
  callback_method: 'get'
});

// Send link to customer
console.log(paymentLink.short_url);
```

### 3. Transfers (Marketplace)

```javascript
// Split payment between multiple accounts
const transfer = await razorpayInstance.transfers.create({
  account: 'acc_vendorAccountId',
  amount: 10000, // ₹100 to vendor
  currency: 'INR',
  source: paymentId,
  notes: {
    commission: 'Vendor payout'
  }
});
```

---

## 📝 Error Handling

```javascript
try {
  const razorpayOrder = await razorpayInstance.orders.create(options);
} catch (error) {
  if (error.statusCode === 400) {
    // Bad request - invalid parameters
    return res.status(400).json({ message: 'Invalid order details' });
  } else if (error.statusCode === 401) {
    // Unauthorized - invalid API key
    return res.status(500).json({ message: 'Payment gateway error' });
  } else {
    // Other errors
    return res.status(500).json({ message: 'Payment creation failed' });
  }
}
```

**Common Errors:**
- `BAD_REQUEST_ERROR` - Invalid parameters
- `GATEWAY_ERROR` - Payment gateway down
- `SERVER_ERROR` - Internal server error

---

## 🎓 Hands-On Exercise

**Build: SaaS Subscription Platform**

Create a SaaS application with:

1. **Free Trial**: 7-day free trial, no card required
2. **Monthly Subscription**: ₹499/month
3. **Annual Subscription**: ₹4,999/year (17% discount)
4. **Payment Gateway**: Razorpay integration
5. **Features**:
   - User registers → gets 7-day trial
   - After trial ends → prompt to subscribe
   - Create Razorpay subscription
   - Handle subscription webhooks (activated, cancelled, charged)
   - Allow plan upgrades/downgrades
   - Handle failed payments → retry 3 times
   - Cancel subscription → retain data for 30 days
6. **React Dashboard**:
   - Show subscription status
   - Billing history
   - Upgrade/downgrade options
   - Cancel subscription button

**Deliverables:**
- Subscription management backend
- Webhook handlers for all events
- React billing dashboard
- Email notifications on payment events

---

## 📚 Practice Projects

After completing this chapter, build these projects:

- **Project #28:** [Shopping Cart with Razorpay Checkout](../projects/28-shopping-cart.md)
- **Project #29:** [Complete Order Management System](../projects/29-order-management.md)
- **Project #51:** [SaaS Billing Platform](../projects/51-saas-platform.md) ⭐

---

## 🔗 Resources

**Official Documentation:**
- [Razorpay Docs](https://razorpay.com/docs/)
- [Node.js SDK](https://razorpay.com/docs/api/nodejs/)
- [Payment Gateway](https://razorpay.com/docs/payments/)
- [Webhooks](https://razorpay.com/docs/webhooks/)

**Testing:**
- [Test Cards](https://razorpay.com/docs/payments/test-cards/)
- [Test UPI](https://razorpay.com/docs/payments/test-upi/)

---

## ✅ Chapter Checklist

Before moving to the next chapter, ensure you can:

- [ ] Set up Razorpay account and get API keys
- [ ] Create payment orders from backend
- [ ] Verify payment signatures for security
- [ ] Handle payment success and failure
- [ ] Implement webhook handlers
- [ ] Process refunds (full and partial)
- [ ] Integrate Razorpay checkout in React
- [ ] Test payments with test cards
- [ ] Store payment history in database
- [ ] Handle edge cases (network failures, duplicate payments)

---

## 🎯 What's Next?

In the next chapter, we'll learn **AI Chatbot Integration** - how to add intelligent conversational AI to your applications using OpenAI and Google Gemini.

**Coming Up:**
- Streaming chat responses
- Conversation history management
- Function calling with AI
- Building AI-powered customer support

---

**Made with ❤️ by Tushar Parlikar**

[← Previous: CI/CD with GitHub Actions](B15_CICD_GITHUB_ACTIONS.md) | [Next: AI Chatbot Integration →](B17_AI_CHATBOT_INTEGRATION.md)
