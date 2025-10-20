# Project 27: Product Recommendation Engine 🎯

> Build an intelligent recommendation system that suggests products based on user behavior

**Difficulty:** 🟠 Advanced  
**Time:** 6-8 hours  
**Chapter:** B10 - E-Commerce Backend  
**React Integration:** ✅ Required

---

## 🎯 Learning Objectives

- Implement recommendation algorithms
- Track user behavior (clicks, views, purchases)
- Calculate product similarity scores
- Build collaborative filtering logic
- Optimize database queries for recommendations
- Create personalized user experiences

---

## 📋 What You'll Build

### Core Features
- ✅ Track user product views and clicks
- ✅ Track user purchase history
- ✅ Calculate product similarity (category, price, tags)
- ✅ Recommend "Similar Products"
- ✅ Recommend "Customers Also Bought"
- ✅ Recommend "Based on Your Interests"
- ✅ Display personalized homepage recommendations

### Technical Stack
- **Backend:** Node.js, Express, MongoDB
- **Frontend:** React with product cards
- **Algorithms:** Collaborative filtering, Content-based filtering

---

## 🗺️ Implementation Roadmap

### Phase 1: Data Modeling (1 hour)

#### Todo 1.1: Design Product Schema
Create a Product model with fields for recommendations:
- [ ] Basic info (name, price, description, image)
- [ ] Category and subcategory
- [ ] Tags array (for similarity matching)
- [ ] Price range tier (budget, mid, premium)
- [ ] View count (for popularity)
- [ ] Purchase count (for trending)

**Hint:** Add fields you'll query frequently as indexes for performance

#### Todo 1.2: Design User Behavior Schema
Create models to track user interactions:
- [ ] **ProductView model:** userId, productId, timestamp
- [ ] **ProductClick model:** userId, productId, timestamp
- [ ] **Purchase model:** userId, products array, timestamp

**Hint:** Think about how you'll query "all products user has viewed" efficiently

#### Todo 1.3: Design User Preference Schema
Store calculated user preferences:
- [ ] **UserPreference model:** 
  - userId
  - favoriteCategories array (with counts)
  - priceRangePreference (budget/mid/premium)
  - frequentTags array

**Concept Reference:** [MongoDB Schema Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)

---

### Phase 2: Track User Behavior (2 hours)

#### Todo 2.1: Track Product Views
Create endpoint to record when user views a product:
- [ ] POST `/api/analytics/view/:productId`
- [ ] Save: userId (from auth), productId, timestamp
- [ ] Increment product's total view count
- [ ] Update product's popularity score

**React Integration:**
```javascript
// In React ProductDetails component
useEffect(() => {
  // When product loads, track view
  trackProductView(productId);
}, [productId]);
```

#### Todo 2.2: Track Product Clicks
Record clicks on product cards:
- [ ] POST `/api/analytics/click/:productId`
- [ ] Differentiate between view (opened page) and click (browsed from list)

**React Integration:**
```javascript
// In React ProductCard component
<div onClick={() => {
  trackProductClick(productId);
  navigate(`/product/${productId}`);
}}>
```

#### Todo 2.3: Track Purchases
When order is completed:
- [ ] In your order creation endpoint, also save to Purchase model
- [ ] Record each product in the order
- [ ] Increment each product's purchase count

**Hint:** You can call this after successful payment or order confirmation

---

### Phase 3: Calculate Product Similarity (2 hours)

#### Todo 3.1: Category-Based Similarity
Algorithm: Products in same category are similar
- [ ] Create function `findSimilarByCategory(productId)`
- [ ] Find the product's category
- [ ] Query products with same category
- [ ] Exclude the current product
- [ ] Sort by popularity (view count)
- [ ] Return top 10

**Hint:** Use MongoDB's `find()` with category filter and limit

#### Todo 3.2: Tag-Based Similarity
Algorithm: Products with matching tags are similar
- [ ] Create function `findSimilarByTags(productId)`
- [ ] Get the product's tags array
- [ ] Find products that have ANY matching tags
- [ ] Calculate similarity score (# of matching tags)
- [ ] Sort by similarity score then popularity
- [ ] Return top 10

**Hint:** MongoDB `$in` operator for array matching

**Concept Reference:** [Content-Based Filtering](https://developers.google.com/machine-learning/recommendation/content-based/basics)

#### Todo 3.3: Price-Based Similarity
Algorithm: Products in similar price range
- [ ] Create function `findSimilarByPrice(productId)`
- [ ] Get product price
- [ ] Find products within ±20% price range
- [ ] Sort by category match first, then price proximity

**Algorithm Example:**
```
Product Price: $100
Range: $80 - $120
Query: price >= 80 AND price <= 120
```

---

### Phase 4: Collaborative Filtering (2 hours)

#### Todo 4.1: "Customers Also Bought" Algorithm
Logic: If user bought Product A, show what others bought with Product A

**Steps:**
- [ ] Find all purchases containing the current product
- [ ] Get all other products from those purchases
- [ ] Count frequency of each product
- [ ] Sort by frequency (most co-purchased first)
- [ ] Return top 10

**Pseudocode:**
```
1. Find Purchase where products contains productId
2. Extract all products from these purchases
3. Count occurrences of each product
4. Sort by count descending
5. Exclude current product
6. Return top 10
```

**Hint:** Use MongoDB aggregation pipeline with `$unwind` and `$group`

**Concept Reference:** [Collaborative Filtering Explained](https://towardsdatascience.com/introduction-to-recommender-systems-6c66cf15ada)

#### Todo 4.2: "Customers Also Viewed" Algorithm
Logic: Users who viewed Product A also viewed Products B, C, D

**Steps:**
- [ ] Find all users who viewed current product
- [ ] Get all products those users viewed
- [ ] Count frequency
- [ ] Return top products

**Hint:** Similar to "also bought" but using ProductView model

#### Todo 4.3: User-Based Recommendations
Find similar users and recommend what they liked:
- [ ] Find users with similar viewing history
- [ ] Get products they viewed/bought
- [ ] Exclude products current user already saw
- [ ] Return personalized recommendations

**Algorithm:**
```
1. Get current user's viewed products
2. Find users who viewed same products
3. Get products THOSE users viewed
4. Exclude already viewed by current user
5. Sort by popularity among similar users
```

---

### Phase 5: Build Recommendation API (1 hour)

#### Todo 5.1: Similar Products Endpoint
- [ ] GET `/api/recommendations/similar/:productId`
- [ ] Combine results from category, tags, and price algorithms
- [ ] Merge and deduplicate results
- [ ] Sort by combined score
- [ ] Return top 12 products

**Response Structure:**
```json
{
  "success": true,
  "data": {
    "products": [...],
    "algorithm": "content-based",
    "total": 12
  }
}
```

#### Todo 5.2: Personalized Homepage Endpoint
- [ ] GET `/api/recommendations/for-you` (authenticated)
- [ ] Get user's favorite categories from UserPreference
- [ ] Get user's price range preference
- [ ] Find products matching preferences
- [ ] Mix trending + personalized
- [ ] Return 20-30 products

**Logic:**
```
50% from user's favorite categories
30% trending products
20% new arrivals
```

#### Todo 5.3: Also Bought/Viewed Endpoints
- [ ] GET `/api/recommendations/also-bought/:productId`
- [ ] GET `/api/recommendations/also-viewed/:productId`

---

### Phase 6: Update User Preferences (1 hour)

#### Todo 6.1: Calculate Category Preferences
When user views/buys products:
- [ ] Increment count for that product's category
- [ ] Store in UserPreference model
- [ ] Update favorite categories list

**Example:**
```
User views phone → Electronics: +1
User views laptop → Electronics: +1
User views shoes → Fashion: +1

Preferences: Electronics (2), Fashion (1)
```

#### Todo 6.2: Calculate Price Preference
Analyze user's purchase history:
- [ ] Calculate average price of purchased items
- [ ] Categorize as budget (<$50), mid ($50-$200), premium (>$200)
- [ ] Save to UserPreference

#### Todo 6.3: Schedule Preference Updates
- [ ] Create background job (or API endpoint)
- [ ] Run after each purchase
- [ ] Or batch process daily

**Hint:** You can use simple setInterval or cron jobs for this

---

### Phase 7: React Frontend Integration (1-2 hours)

#### Todo 7.1: Product Details - Similar Products
In ProductDetails page:
- [ ] Fetch similar products when product loads
- [ ] Display in "Similar Products" section
- [ ] Show product cards in horizontal scroll
- [ ] Track clicks on recommendations

**React Component Structure:**
```jsx
<ProductDetails>
  <ProductInfo />
  <SimilarProducts productId={productId} />
</ProductDetails>
```

#### Todo 7.2: Product Details - Also Bought
- [ ] Fetch "Customers Also Bought"
- [ ] Display in separate section
- [ ] Add "Frequently Bought Together" badge

#### Todo 7.3: Homepage - Personalized Feed
- [ ] Fetch personalized recommendations
- [ ] Display multiple sections:
  - "Recommended for You"
  - "Based on Your Recent Views"
  - "Trending in [Favorite Category]"
- [ ] Implement lazy loading (show more on scroll)

#### Todo 7.4: Track Everything!
Add tracking to all interactions:
- [ ] Product card click → track click
- [ ] Product page load → track view
- [ ] Add to cart → track intent
- [ ] Purchase → track conversion

**React Hooks Example:**
```jsx
const useProductTracking = () => {
  const trackView = (productId) => {
    // API call to track view
  };
  
  const trackClick = (productId) => {
    // API call to track click
  };
  
  return { trackView, trackClick };
};
```

---

## 🧠 Algorithm Deep Dive

### Algorithm 1: Jaccard Similarity (Tag Matching)

**Purpose:** Calculate how similar two products are based on tags

**Formula:**
```
Similarity = (Common Tags) / (Total Unique Tags)

Product A tags: [red, cotton, summer, dress]
Product B tags: [blue, cotton, summer, shirt]

Common: [cotton, summer] = 2
Total: [red, cotton, summer, dress, blue, shirt] = 6

Similarity = 2/6 = 0.33 (33% similar)
```

**Implementation Hint:**
- [ ] Get both products' tags
- [ ] Find intersection (common tags)
- [ ] Find union (all unique tags)
- [ ] Divide intersection length by union length

---

### Algorithm 2: Popularity Score

**Purpose:** Rank products by engagement

**Formula:**
```
Score = (Views * 1) + (Clicks * 2) + (Purchases * 10)

Product A: 100 views, 20 clicks, 5 purchases
Score = (100 * 1) + (20 * 2) + (5 * 10) = 190

Product B: 50 views, 15 clicks, 10 purchases
Score = (50 * 1) + (15 * 2) + (10 * 10) = 180
```

**Why weights?**
- Purchases most valuable (10x)
- Clicks show interest (2x)
- Views show exposure (1x)

---

### Algorithm 3: Time Decay

**Purpose:** Favor recent activity over old

**Formula:**
```
Weight = 1 / (Days Since Activity + 1)

View from today: 1 / (0 + 1) = 1.0
View from 7 days ago: 1 / (7 + 1) = 0.125
View from 30 days ago: 1 / (30 + 1) = 0.032
```

**Use Case:** Recent views matter more for preferences

---

## 🎨 React UI Components to Build

### Component 1: Recommendation Section
```jsx
<RecommendationSection
  title="Similar Products"
  products={similarProducts}
  onProductClick={handleClick}
/>
```
**Features:**
- [ ] Horizontal scroll
- [ ] Product cards with image, price, rating
- [ ] Loading skeleton
- [ ] "View All" button

### Component 2: Product Grid
```jsx
<ProductGrid
  products={recommendations}
  layout="grid"  // or "list"
  trackClicks={true}
/>
```

### Component 3: Personalized Homepage
```jsx
<PersonalizedHome>
  <Section title="Recommended for You" />
  <Section title="Trending Now" />
  <Section title="Based on Your Interests" />
</PersonalizedHome>
```

---

## 📊 Testing Your Recommendations

### Test Scenario 1: New User (Cold Start)
- [ ] User with no history
- [ ] Should see: Trending products, Popular in all categories
- [ ] NOT personalized (no data yet)

### Test Scenario 2: Active User
- [ ] User viewed 10 products in Electronics
- [ ] Should see: More Electronics recommendations
- [ ] Categorized properly

### Test Scenario 3: Purchase Made
- [ ] User bought product A
- [ ] View product A page
- [ ] Should see: "Customers also bought" with relevant items

### Test Scenario 4: Multi-Category User
- [ ] User interested in both Electronics and Fashion
- [ ] Should see: Mix of both categories
- [ ] Balanced recommendations

---

## 🚀 Enhancements

### Level 1: Basic Improvements
- [ ] Add "Viewed Recently" section (user's history)
- [ ] Add "New Arrivals" section
- [ ] Add "On Sale" recommendations
- [ ] Cache recommendations (expires every hour)

### Level 2: Advanced Algorithms
- [ ] Implement matrix factorization
- [ ] Add seasonality (summer products in summer)
- [ ] Add trending score (viral products)
- [ ] Implement A/B testing (test different algorithms)

### Level 3: Machine Learning Integration
- [ ] Use TensorFlow.js for predictions
- [ ] Train model on historical data
- [ ] Implement deep learning recommendations
- [ ] Real-time model updates

### Level 4: Real-Time Updates
- [ ] WebSocket for live recommendations
- [ ] Update recommendations as user browses
- [ ] Real-time popularity updates
- [ ] Collaborative sessions (friends' activities)

---

## 📚 Concepts to Learn

### Required Reading
- [ ] [Recommendation Systems Overview](https://developers.google.com/machine-learning/recommendation)
- [ ] [Collaborative Filtering](https://towardsdatascience.com/introduction-to-recommender-systems-6c66cf15ada)
- [ ] [Content-Based Filtering](https://realpython.com/build-recommendation-engine-collaborative-filtering/)
- [ ] [MongoDB Aggregation](https://docs.mongodb.com/manual/aggregation/)

### Related Chapters
- [B04: MongoDB Basics](../chapters/B04_MONGODB_BASICS.md) - Queries
- [B06: Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md) - Complex queries
- [B10: E-Commerce Backend](../chapters/B10_ECOMMERCE_BACKEND.md) - Product system

### Additional Resources
- [Netflix Recommendation System](https://netflixtechblog.com/netflix-recommendations-beyond-the-5-stars-part-1-55838468f429)
- [Amazon's Recommendation Algorithm](https://www.amazon.science/the-history-of-amazons-recommendation-algorithm)
- [YouTube's Recommendation System](https://research.google/pubs/pub45530/)

---

## 🐛 Common Challenges

### Challenge 1: Cold Start Problem
**Issue:** New users/products have no data  
**Solutions:**
- [ ] Show popular products to new users
- [ ] Ask users for preferences on signup
- [ ] Use product metadata (category, price) initially
- [ ] Gradually personalize as data accumulates

### Challenge 2: Scalability
**Issue:** Calculating recommendations for millions of products is slow  
**Solutions:**
- [ ] Pre-calculate recommendations (batch job)
- [ ] Cache results
- [ ] Use indexes on frequently queried fields
- [ ] Implement pagination

### Challenge 3: Diversity
**Issue:** Recommendations too similar (filter bubble)  
**Solutions:**
- [ ] Mix different algorithms
- [ ] Add random exploration (10% random products)
- [ ] Ensure category diversity
- [ ] Implement "Because you viewed [X]" explanations

### Challenge 4: Sparsity
**Issue:** Not enough user interaction data  
**Solutions:**
- [ ] Combine multiple signals (views + clicks + purchases)
- [ ] Use implicit feedback (time spent)
- [ ] Leverage product metadata
- [ ] Implement hybrid approach

---

## ✅ Completion Checklist

### Backend
- [ ] Product view tracking working
- [ ] Product click tracking working
- [ ] Purchase tracking working
- [ ] Similar products endpoint returns results
- [ ] Also bought endpoint returns results
- [ ] Personalized recommendations endpoint working
- [ ] User preferences are calculated and updated
- [ ] All endpoints properly authenticated

### Frontend
- [ ] Product page shows similar products
- [ ] Product page shows "also bought"
- [ ] Homepage shows personalized recommendations
- [ ] All clicks are tracked
- [ ] All views are tracked
- [ ] Loading states implemented
- [ ] Error handling in place

### Algorithms
- [ ] Category-based similarity works
- [ ] Tag-based similarity works
- [ ] Price-based similarity works
- [ ] Collaborative filtering works
- [ ] Popularity scoring implemented
- [ ] Results are relevant and useful

---

## 🎯 Success Criteria

You've mastered this project when:
- ✅ Recommendations feel personalized and relevant
- ✅ Users discover new products they're interested in
- ✅ Similar products are actually similar
- ✅ "Also bought" makes logical sense
- ✅ System handles new users gracefully
- ✅ Performance is acceptable (< 500ms response time)
- ✅ You understand the algorithms, not just copied code

---

## 🏆 Real-World Impact

This project teaches you:
- **E-Commerce:** Amazon, eBay, Shopify use these techniques
- **Streaming:** Netflix, Spotify, YouTube recommendations
- **Social Media:** Facebook, Instagram, TikTok feed algorithms
- **Data Science:** Foundation for ML/AI career
- **Product Management:** Understanding user behavior

---

## 📈 Analytics to Track

Monitor your recommendation system:
- [ ] Click-through rate (CTR) on recommendations
- [ ] Conversion rate from recommendations
- [ ] Time to purchase after viewing recommendations
- [ ] User engagement with recommended products
- [ ] Algorithm performance comparison

**Metric Goals:**
- CTR > 5% (good recommendation relevance)
- Conversion rate > 2% (recommendations lead to sales)
- User satisfaction (survey or A/B test)

---

**Next Project:** [Project 28: Shopping Cart System](28-shopping-cart.md)

**This is where algorithms meet real business value! 🎯**
