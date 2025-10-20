# Project 46: Pagination & Infinite Scroll

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md)

---

## 📝 Description

Implement efficient pagination strategies for large datasets.

**What You'll Learn:**
- Offset pagination
- Cursor-based pagination
- Infinite scroll
- Performance optimization
- React infinite scroll

---

## 🎯 Project Goals

- ✅ Offset pagination
- ✅ Cursor-based pagination
- ✅ Infinite scroll API
- ✅ Load more functionality
- ✅ React infinite scroll UI

---

## 🗺️ Roadmap

### Step 1: Offset Pagination
**Todo:**
- [ ] `GET /api/posts?page=1&limit=20`
- [ ] Calculate skip: `(page - 1) * limit`
- [ ] Return: items, totalPages, currentPage

### Step 2: Cursor Pagination
**Todo:**
- [ ] Use last item's ID as cursor
- [ ] `GET /api/posts?cursor=lastId&limit=20`
- [ ] Query: `_id > cursor`
- [ ] More efficient for large datasets

### Step 3: React Infinite Scroll
**Todo:**
- [ ] Load first page on mount
- [ ] Detect when user scrolls to bottom
- [ ] Load next page
- [ ] Append to existing list
- [ ] Show loading spinner

---

**Next:** [Project 47: API Documentation Generator](47-api-docs.md)
