# Project 9: Bookmark Manager API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 2-3 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build an API to save and organize bookmarks with categories, tags, and screenshots.

**What You'll Learn:**
- URL metadata extraction
- Favicon fetching
- Screenshot capture (optional)
- Collections/categories
- Import/export bookmarks

---

## 🎯 Project Goals

- ✅ Save URLs with metadata
- ✅ Organize in collections
- ✅ Tag bookmarks
- ✅ Search by title/URL/tags
- ✅ React bookmark manager UI

---

## 🗺️ Roadmap

### Step 1: Bookmark Model
**Todo:**
- [ ] Create bookmark structure:
  - id, url, title, description
  - favicon, screenshot
  - collectionId, tags
  - createdAt

---

### Step 2: URL Metadata Extraction
**Todo:**
- [ ] When user saves URL, fetch its metadata
- [ ] Extract: title, description, favicon
- [ ] Use package: `node-fetch` + HTML parsing
- [ ] Handle errors if URL unreachable

**Hint:** Fetch HTML, parse `<title>`, `<meta description>`, `<link rel="icon">`

---

### Step 3: Collections System
**Todo:**
- [ ] Collections (like folders)
- [ ] CRUD for collections
- [ ] Assign bookmarks to collections
- [ ] Default "Uncategorized" collection

---

### Step 4: Search & Filter
**Todo:**
- [ ] Search by title, URL, description
- [ ] Filter by collection
- [ ] Filter by tags
- [ ] Sort by: date added, title

---

### Step 5: Import/Export
**Todo:**
- [ ] Export bookmarks to JSON
- [ ] Export to HTML (browser format)
- [ ] Import from JSON
- [ ] Import from HTML

---

### Step 6: React UI
**Todo:**
- [ ] Add bookmark form (URL input)
- [ ] Display bookmarks grid/list
- [ ] Collections sidebar
- [ ] Search bar
- [ ] Tag filtering
- [ ] Quick add browser extension (optional)

---

## 💡 Hints & Tips

### Fetch Favicon:
```
https://www.google.com/s2/favicons?domain=example.com
```

### Metadata Extraction:
Use `cheerio` package to parse HTML

---

**Next:** [Project 10: Calculator API](10-calculator-api.md)
