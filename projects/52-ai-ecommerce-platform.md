# Project 52: AI-Powered E-Commerce Platform ⭐⭐⭐

**Build a Production-Grade Amazon/Flipkart Clone with AI & ML**

---

## 🎯 Project Overview

**Difficulty:** Expert ⭐⭐⭐⭐⭐  
**Time Required:** 45-50 hours  
**Skills Used:** B01-B20 (All chapters)

Build a complete e-commerce platform with AI recommendations, smart search, dynamic pricing, and real-time analytics.

---

## 💡 What You'll Build

### Core Features:

1. **Product Catalog**
   - Categories and subcategories
   - Advanced filters (price, brand, rating)
   - Elasticsearch integration for fast search
   - Product variations (size, color)

2. **AI Recommendations**
   - Collaborative filtering (users who bought this also bought...)
   - Content-based filtering (similar products)
   - Hybrid recommendation system
   - Personalized homepage

3. **Smart Search**
   - NLP-powered search with typo tolerance
   - Autocomplete suggestions
   - Search ranking with relevance scoring
   - Voice search integration

4. **Dynamic Pricing**
   - ML model predicts optimal prices
   - Competitor price tracking
   - Demand-based pricing
   - Flash sales and discounts

5. **Inventory Management**
   - Real-time stock tracking
   - Auto-reorder with ML prediction
   - Warehouse management
   - Low stock alerts

6. **Payment Gateway**
   - Razorpay integration
   - Multiple payment methods (Cards, UPI, Wallets, COD)
   - EMI options
   - Refund management

7. **Order Tracking**
   - Real-time order updates with Socket.io
   - Shipment tracking
   - Delivery notifications
   - Order history

8. **Review Sentiment Analysis**
   - AI analyzes review sentiment (positive/negative)
   - Auto-flag spam reviews
   - Sentiment-based product ranking
   - Review summarization

9. **AI Chatbot**
   - Customer support chatbot
   - Product recommendations via chat
   - Order tracking through chat
   - FAQs automation

10. **Admin Analytics**
    - Real-time sales dashboard
    - Revenue charts
    - Best-selling products
    - Customer insights

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────┐
│           React SPA (TypeScript)                │
│  - Product Catalog  - Cart  - Checkout          │
│  - User Dashboard   - Admin Panel               │
└────────────────┬────────────────────────────────┘
                 │
    ┌────────────▼────────────────────┐
    │   Node.js API Gateway           │
    │   - Rate Limiting               │
    │   - Authentication (JWT)        │
    │   - Request Validation          │
    └────┬───────┬──────────┬─────────┘
         │       │          │
    ┌────▼───┐ ┌▼──────┐ ┌▼────────┐
    │MongoDB │ │Redis  │ │Socket.io│
    │        │ │Cache  │ │Real-time│
    └────────┘ └───────┘ └─────────┘
         │
    ┌────▼──────────────────────────┐
    │   Python ML Service           │
    │   - Recommendations           │
    │   - Price Prediction          │
    │   - Sentiment Analysis        │
    └───────────────────────────────┘
         │
    ┌────▼──────────────────────────┐
    │   Elasticsearch               │
    │   - Product Search            │
    │   - Full-text indexing        │
    └───────────────────────────────┘
```

---

## 📋 Step-by-Step Roadmap

### Phase 1: Product Catalog (10 hours)

**Database Schema:**

```javascript
// Product Model
{
  _id: ObjectId,
  name: String,
  description: String,
  category: ObjectId,
  subcategory: ObjectId,
  brand: String,
  price: Number,
  mrp: Number, // Maximum Retail Price
  discount: Number, // Percentage
  stock: Number,
  images: [String], // Cloudinary URLs
  variations: [{
    size: String,
    color: String,
    sku: String,
    stock: Number,
    priceModifier: Number
  }],
  specifications: {
    weight: String,
    dimensions: String,
    material: String,
    // Dynamic fields
  },
  tags: [String],
  rating: {
    average: Number,
    count: Number
  },
  reviews: [{
    user: ObjectId,
    rating: Number,
    text: String,
    sentiment: String, // positive/negative
    helpful: Number,
    createdAt: Date
  }],
  createdAt: Date,
  updatedAt: Date
}
```

**Tasks:**
1. Create product CRUD APIs
2. Implement category hierarchy
3. Add product variations
4. Upload images to Cloudinary
5. Build React product listing page
6. Implement filters and sorting

---

### Phase 2: Search with Elasticsearch (8 hours)

**Setup Elasticsearch:**

```bash
npm install @elastic/elasticsearch
```

**Index Products:**

```javascript
const { Client } = require('@elastic/elasticsearch');
const client = new Client({ node: 'http://localhost:9200' });

// Index all products
async function indexProducts() {
  const products = await Product.find();
  
  const body = products.flatMap(product => [
    { index: { _index: 'products', _id: product._id } },
    {
      name: product.name,
      description: product.description,
      category: product.category.name,
      brand: product.brand,
      price: product.price,
      tags: product.tags
    }
  ]);
  
  await client.bulk({ body });
}

// Search products
router.get('/products/search', async (req, res) => {
  const { q } = req.query;
  
  const { body } = await client.search({
    index: 'products',
    body: {
      query: {
        multi_match: {
          query: q,
          fields: ['name^3', 'description', 'brand^2', 'tags'],
          fuzziness: 'AUTO' // Typo tolerance
        }
      }
    }
  });
  
  const productIds = body.hits.hits.map(hit => hit._id);
  const products = await Product.find({ _id: { $in: productIds } });
  
  res.json({ products });
});
```

**Tasks:**
1. Set up Elasticsearch
2. Index all products
3. Implement fuzzy search
4. Add autocomplete
5. Build search UI with React

---

### Phase 3: AI Recommendations (10 hours)

**Collaborative Filtering (Python):**

```python
# ml-service/recommender.py
from sklearn.metrics.pairwise import cosine_similarity
import pandas as pd
import numpy as np

@app.route('/recommend/collaborative', methods=['POST'])
def collaborative_filtering():
    user_id = request.json['user_id']
    
    # Load user-product matrix (userId x productId x rating)
    ratings_df = pd.read_csv('user_ratings.csv')
    matrix = ratings_df.pivot_table(
        index='user_id',
        columns='product_id',
        values='rating'
    ).fillna(0)
    
    # Calculate user similarity
    user_similarity = cosine_similarity(matrix)
    user_idx = matrix.index.get_loc(user_id)
    
    # Find similar users
    similar_users = user_similarity[user_idx].argsort()[::-1][1:11]
    
    # Recommend products liked by similar users
    recommendations = []
    for similar_user_idx in similar_users:
        similar_user_id = matrix.index[similar_user_idx]
        liked_products = matrix.loc[similar_user_id]
        liked_products = liked_products[liked_products > 3.5]
        
        for product_id, rating in liked_products.items():
            if matrix.loc[user_id, product_id] == 0:  # Not yet rated
                recommendations.append({
                    'product_id': int(product_id),
                    'score': float(rating)
                })
    
    # Sort and return top 10
    recommendations.sort(key=lambda x: x['score'], reverse=True)
    return jsonify({'recommendations': recommendations[:10]})
```

**Node.js Integration:**

```javascript
router.get('/products/recommendations', auth, async (req, res) => {
  // Call Python ML service
  const response = await axios.post('http://localhost:5001/recommend/collaborative', {
    user_id: req.user._id.toString()
  });
  
  const productIds = response.data.recommendations.map(r => r.product_id);
  const products = await Product.find({ _id: { $in: productIds } });
  
  res.json({ recommendations: products });
});
```

**Tasks:**
1. Build collaborative filtering model
2. Build content-based filtering
3. Create hybrid recommendation system
4. Integrate with Node.js backend
5. Display recommendations on homepage

---

### Phase 4: Razorpay Payment Integration (6 hours)

Follow **Chapter B16** for complete implementation.

**Tasks:**
1. Create Razorpay orders
2. Verify payment signatures
3. Handle webhooks
4. Implement refunds
5. Build checkout UI with React

---

### Phase 5: Real-Time Features (8 hours)

**Order Tracking with Socket.io:**

```javascript
// When order status updates
router.put('/orders/:id/status', auth, isAdmin, async (req, res) => {
  const { status } = req.body;
  const order = await Order.findByIdAndUpdate(
    req.params.id,
    { status },
    { new: true }
  );
  
  // Emit real-time update to customer
  io.to(`user-${order.user}`).emit('order-update', {
    orderId: order._id,
    status: order.status,
    message: `Your order is now ${status}`
  });
  
  res.json({ order });
});
```

**Tasks:**
1. Set up Socket.io
2. Real-time order updates
3. Live inventory updates
4. Real-time admin notifications

---

### Phase 6: Sentiment Analysis (5 hours)

**Python Service:**

```python
from transformers import pipeline

sentiment_analyzer = pipeline("sentiment-analysis")

@app.route('/analyze/sentiment', methods=['POST'])
def analyze_sentiment():
    text = request.json['text']
    result = sentiment_analyzer(text)[0]
    
    return jsonify({
        'sentiment': result['label'],  # POSITIVE/NEGATIVE
        'confidence': result['score']
    })
```

**Auto-analyze reviews:**

```javascript
router.post('/products/:id/reviews', auth, async (req, res) => {
  const { rating, text } = req.body;
  
  // Analyze sentiment
  const sentimentResponse = await axios.post('http://localhost:5001/analyze/sentiment', {
    text
  });
  
  const review = {
    user: req.user._id,
    rating,
    text,
    sentiment: sentimentResponse.data.sentiment.toLowerCase(),
    confidence: sentimentResponse.data.confidence
  };
  
  const product = await Product.findById(req.params.id);
  product.reviews.push(review);
  
  // Update average rating
  product.rating.average = product.reviews.reduce((sum, r) => sum + r.rating, 0) / product.reviews.length;
  product.rating.count = product.reviews.length;
  
  await product.save();
  
  res.json({ review });
});
```

---

## 🎨 React Frontend Features

### Product Listing Page
- Grid/List view toggle
- Filter sidebar (price, brand, rating, category)
- Sort options (price, rating, newest)
- Pagination/Infinite scroll
- Quick view modal

### Product Detail Page
- Image gallery with zoom
- Add to cart/wishlist
- Product variants selector
- Reviews and ratings
- Similar products (AI recommendations)
- Q&A section

### Shopping Cart
- Quantity adjustment
- Apply coupon codes
- Calculate shipping
- Move to wishlist
- Real-time price updates

### Checkout
- Address management
- Payment method selection
- Razorpay integration
- Order summary
- Apply gift cards

### User Dashboard
- Order history
- Track orders (real-time)
- Wishlist
- Saved addresses
- Reviews written

### Admin Dashboard
- Sales analytics (charts)
- Product management
- Order management
- Customer insights
- Inventory alerts

---

## 🚀 Deployment

**Docker Compose:**

```yaml
version: '3.8'

services:
  api:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - MONGODB_URI=mongodb://mongo:27017/ecommerce
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongo
      - redis
      - elasticsearch
      
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
      
  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db
      
  redis:
    image: redis:alpine
    
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
      
  ml-service:
    build: ./ml-service
    ports:
      - "5001:5001"

volumes:
  mongo-data:
```

---

## 📚 Technologies Used

- **Backend**: Node.js, Express, MongoDB, Redis
- **Frontend**: React, TypeScript, Redux Toolkit, TailwindCSS
- **AI/ML**: Python, Scikit-learn, TensorFlow, Transformers
- **Search**: Elasticsearch
- **Payments**: Razorpay
- **Real-time**: Socket.io
- **Storage**: Cloudinary
- **DevOps**: Docker, GitHub Actions

---

## 🎓 Learning Outcomes

✅ Built production-grade e-commerce platform  
✅ Implemented AI/ML recommendations  
✅ Integrated payment gateway  
✅ Used Elasticsearch for search  
✅ Real-time features with WebSockets  
✅ Sentiment analysis with NLP  
✅ Complete full-stack application  

**Portfolio Impact:** This project alone can land you senior developer roles!

---

## ✅ Deliverables

- [ ] Product catalog with 100+ products
- [ ] AI-powered recommendation system
- [ ] Smart search with Elasticsearch
- [ ] Razorpay payment integration
- [ ] Real-time order tracking
- [ ] Sentiment analysis for reviews
- [ ] Admin analytics dashboard
- [ ] AI chatbot for customer support
- [ ] Docker deployment
- [ ] 80%+ test coverage

---

**Related Chapters:** B01-B20 (All chapters), B16 (Payments), B17 (AI), B18 (ML)

**Made with ❤️ by Tushar Parlikar**

[← Back to Projects](README.md) | [Next: Social Media Platform →](53-social-media-platform.md)
