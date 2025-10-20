# Project 49: Caching Strategy with Redis

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md)

---

## 📝 Description

Implement multi-level caching to dramatically improve API performance.

**What You'll Learn:**
- Redis caching
- Cache invalidation
- TTL strategies
- Cache-aside pattern
- Performance optimization

---

## 🎯 Project Goals

- ✅ Redis cache layer
- ✅ Cache frequently accessed data
- ✅ Cache invalidation strategies
- ✅ CDN caching headers
- ✅ React data caching

---

## 🗺️ Roadmap

### Step 1: Setup Redis Cache
**Todo:**
- [ ] Install Redis
- [ ] Create cache wrapper functions
- [ ] `get(key)`, `set(key, value, ttl)`, `del(key)`

### Step 2: Cache-Aside Pattern
**Todo:**
- [ ] Check cache first
- [ ] If miss, query database
- [ ] Store in cache
- [ ] Return data

**Implementation:**
```javascript
async function getUser(id) {
  // Check cache
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  
  // Cache miss - query DB
  const user = await User.findById(id);
  
  // Store in cache (TTL: 1 hour)
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  
  return user;
}
```

### Step 3: Cache Invalidation
**Todo:**
- [ ] Invalidate on update/delete
- [ ] Pattern-based invalidation
- [ ] TTL-based expiry

**Strategies:**
```javascript
// On user update
await redis.del(`user:${userId}`);

// Delete multiple keys
await redis.del(`user:${userId}`, `user:${userId}:posts`);

// Pattern delete (all user caches)
const keys = await redis.keys('user:*');
if (keys.length) await redis.del(...keys);
```

### Step 4: Cache Popular Data
**Todo:**
- [ ] Homepage data (cache 5 min)
- [ ] Product listings (cache 1 hour)
- [ ] User profiles (cache 10 min)
- [ ] Static content (cache 1 day)

### Step 5: HTTP Cache Headers
**Todo:**
- [ ] Set Cache-Control headers
- [ ] ETag for conditional requests
- [ ] Last-Modified headers

```javascript
res.set('Cache-Control', 'public, max-age=3600');
res.set('ETag', generateETag(data));
```

### Step 6: React Query Cache
**Todo:**
- [ ] Use React Query for client-side caching
- [ ] Stale-while-revalidate strategy
- [ ] Optimistic updates

---

## 💡 Hints & Tips

### Cache Key Naming:
```
user:123
user:123:posts
product:category:electronics
```

### TTL Guidelines:
- Frequently updated: 1-5 min
- Moderate updates: 10-30 min
- Rarely updated: 1+ hour
- Static: 24 hours

---

**Next:** [Project 50: Microservices Communication](50-microservices.md)
