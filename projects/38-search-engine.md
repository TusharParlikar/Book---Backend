# Project 38: Search Engine API

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 5-6 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md)

---

## 📝 Description

Build a full-text search engine with relevance scoring and autocomplete.

**What You'll Learn:**
- Full-text search
- Relevance scoring
- Search indexing
- Autocomplete
- Search filters

---

## 🎯 Project Goals

- ✅ Full-text search across multiple collections
- ✅ Relevance-based ranking
- ✅ Autocomplete suggestions
- ✅ Search filters and facets
- ✅ React search UI

---

## 🗺️ Roadmap

### Step 1: Search Index
**Todo:**
- [ ] Create text indexes on searchable fields
- [ ] MongoDB: `db.collection.createIndex({ title: 'text', description: 'text' })`
- [ ] Index: products, posts, users

### Step 2: Basic Search
**Todo:**
- [ ] `GET /api/search?q=laptop`
- [ ] Search across all collections
- [ ] Return combined results
- [ ] Highlight matches

### Step 3: Relevance Scoring
**Todo:**
- [ ] Title match > Description match
- [ ] Exact match > Partial match
- [ ] Recent > Old
- [ ] Popular > Unpopular

**Score Formula:**
```
score = textScore + recencyBoost + popularityBoost
```

### Step 4: Autocomplete
**Todo:**
- [ ] `GET /api/search/autocomplete?q=lap`
- [ ] Return suggestions as user types
- [ ] Use regex or text index
- [ ] Return top 5

### Step 5: Filters
**Todo:**
- [ ] Filter by category
- [ ] Filter by price range
- [ ] Filter by date range
- [ ] Combine filters

### Step 6: React UI
**Todo:**
- [ ] Search bar with autocomplete dropdown
- [ ] Results page with tabs (Products, Posts, Users)
- [ ] Filter sidebar
- [ ] Highlighted search terms

---

**Next:** [Project 39: Email Marketing API](39-email-marketing.md)
