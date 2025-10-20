# Project 45: API Rate Limiting with Redis

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Implement production-grade rate limiting using Redis for distributed systems.

**What You'll Learn:**
- Redis integration
- Distributed rate limiting
- Token bucket with Redis
- API throttling
- Redis caching

---

## 🎯 Project Goals

- ✅ Rate limit with Redis
- ✅ Multiple limit tiers (free, premium)
- ✅ Distributed system support
- ✅ Real-time limit tracking
- ✅ React usage dashboard

---

## 🗺️ Roadmap

### Step 1: Setup Redis
**Todo:**
- [ ] Install Redis
- [ ] Install `redis` npm package
- [ ] Connect to Redis server
- [ ] Test connection

### Step 2: Sliding Window Algorithm
**Todo:**
- [ ] Use Redis sorted sets
- [ ] Store timestamps of requests
- [ ] Remove old requests outside window
- [ ] Count remaining requests

**Redis Implementation:**
```javascript
const key = `ratelimit:${userId}`;
const now = Date.now();
const windowStart = now - windowMs;

// Remove old entries
await redis.zremrangebyscore(key, 0, windowStart);

// Count current requests
const count = await redis.zcard(key);

if (count >= limit) {
  // Rate limit exceeded
}

// Add current request
await redis.zadd(key, now, now);
await redis.expire(key, Math.ceil(windowMs / 1000));
```

### Step 3: Different Limit Tiers
**Todo:**
- [ ] Free tier: 100 requests/hour
- [ ] Premium: 1000 requests/hour
- [ ] Enterprise: unlimited
- [ ] Check user's tier from database

### Step 4: Rate Limit Headers
**Todo:**
- [ ] X-RateLimit-Limit
- [ ] X-RateLimit-Remaining
- [ ] X-RateLimit-Reset
- [ ] Retry-After (if 429)

### Step 5: React Dashboard
**Todo:**
- [ ] Current usage display
- [ ] Usage graph (last 24 hours)
- [ ] Upgrade prompt if near limit

---

**Next:** [Project 46: GraphQL API](46-graphql-api.md)
