# Project 22: Rate Limiter API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3 hours  
**Related Chapter:** [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Build a rate limiting system to prevent API abuse using different algorithms.

**What You'll Learn:**
- Rate limiting algorithms
- Token bucket algorithm
- Sliding window
- IP-based limiting
- User-based limiting

---

## 🎯 Project Goals

- ✅ Implement multiple rate limit strategies
- ✅ IP-based and user-based limits
- ✅ Different limits for different endpoints
- ✅ Rate limit headers
- ✅ React dashboard showing limits

---

## 🗺️ Roadmap

### Step 1: Fixed Window Algorithm
**Todo:**
- [ ] Track requests per time window
- [ ] Example: 100 requests per hour
- [ ] Reset counter every hour
- [ ] Return 429 if exceeded

**Implementation:**
```
Store: { IP: { count, windowStart } }
On request:
  If current time > windowStart + 1 hour: reset count
  Increment count
  If count > limit: reject
```

---

### Step 2: Sliding Window Algorithm
**Todo:**
- [ ] More accurate than fixed window
- [ ] Track timestamps of all requests
- [ ] Remove old requests outside window
- [ ] Count requests in last N minutes

**Implementation:**
```
Store: { IP: [timestamp1, timestamp2, ...] }
On request:
  Remove timestamps older than window
  Add current timestamp
  If length > limit: reject
```

---

### Step 3: Token Bucket Algorithm
**Todo:**
- [ ] Bucket has tokens, refills over time
- [ ] Each request consumes 1 token
- [ ] If no tokens, reject request
- [ ] Tokens refill at constant rate

**Implementation:**
```
Store: { IP: { tokens, lastRefill } }
On request:
  Calculate tokens to add based on time elapsed
  Add tokens (max = bucket capacity)
  If tokens >= 1: allow request, consume token
  Else: reject
```

---

### Step 4: Endpoint-Specific Limits
**Todo:**
- [ ] Different limits for different routes
- [ ] `/api/login` - 5 per 15 min (strict)
- [ ] `/api/posts` - 100 per hour (normal)
- [ ] `/api/search` - 1000 per hour (loose)

---

### Step 5: User vs IP Limits
**Todo:**
- [ ] Authenticated users: limit by user ID
- [ ] Anonymous: limit by IP
- [ ] Premium users: higher limits

---

### Step 6: Rate Limit Headers
**Todo:**
- [ ] Add response headers:
  - `X-RateLimit-Limit` - Total allowed
  - `X-RateLimit-Remaining` - Remaining in window
  - `X-RateLimit-Reset` - When limit resets
  - `Retry-After` - Seconds to wait (if 429)

---

### Step 7: React Dashboard
**Todo:**
- [ ] Display current usage
- [ ] Show limits for each endpoint
- [ ] Remaining requests
- [ ] Time until reset
- [ ] Usage graphs

---

## 💡 Hints & Tips

### Fixed Window:
```javascript
const now = Date.now();
const windowStart = Math.floor(now / WINDOW_MS) * WINDOW_MS;
```

### Sliding Window:
```javascript
const cutoff = Date.now() - WINDOW_MS;
requests = requests.filter(t => t > cutoff);
```

### Token Bucket:
```javascript
const elapsed = now - lastRefill;
const tokensToAdd = (elapsed / 1000) * REFILL_RATE;
tokens = Math.min(tokens + tokensToAdd, CAPACITY);
```

---

## 📚 Concepts

- Rate limiting algorithms
- Token bucket
- Sliding window
- API security
- Request throttling

---

**Next:** [Project 23: Coupon System API](23-coupon-system.md)
