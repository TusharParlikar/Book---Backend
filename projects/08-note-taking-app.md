# Project 8: Note Taking App API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 2-3 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build a note-taking API with folders, tags, and search. Like a backend for Google Keep or Evernote.

**What You'll Learn:**
- Hierarchical data (folders → notes)
- Tagging system
- Full-text search
- Markdown support
- Favorites/pinned items

---

## 🎯 Project Goals

- ✅ Create, edit, delete notes
- ✅ Organize notes in folders
- ✅ Add tags to notes
- ✅ Search notes by title/content
- ✅ Pin/favorite notes
- ✅ React note editor with rich text

---

## 🗺️ Roadmap

### Step 1: Data Models
**Todo:**
- [ ] Note structure:
  - id, title, content, folderId
  - tags (array), isPinned, color
  - createdAt, updatedAt
- [ ] Folder structure:
  - id, name, color, icon
- [ ] Create in-memory storage

---

### Step 2: Notes CRUD
**Todo:**
- [ ] `GET /api/notes` - All notes
- [ ] `GET /api/notes/:id` - Single note
- [ ] `POST /api/notes` - Create
- [ ] `PUT /api/notes/:id` - Update
- [ ] `DELETE /api/notes/:id` - Delete
- [ ] `PATCH /api/notes/:id/pin` - Toggle pin

---

### Step 3: Folders System
**Todo:**
- [ ] `GET /api/folders` - All folders
- [ ] `POST /api/folders` - Create folder
- [ ] `GET /api/folders/:id/notes` - Notes in folder
- [ ] `PUT /api/folders/:id` - Update folder
- [ ] `DELETE /api/folders/:id` - Delete (move notes to default)

---

### Step 4: Tagging System
**Todo:**
- [ ] Add tags array to notes
- [ ] `GET /api/tags` - Get all unique tags
- [ ] `GET /api/notes/tag/:tag` - Filter by tag
- [ ] Auto-suggest tags based on existing ones

---

### Step 5: Search Functionality
**Todo:**
- [ ] `GET /api/notes/search?q=keyword`
- [ ] Search in title and content
- [ ] Case-insensitive
- [ ] Return matching notes
- [ ] Highlight matches (optional)

**Algorithm:**
```
Loop through notes
Check if keyword exists in title or content
Return matches
```

---

### Step 6: Additional Features
**Todo:**
- [ ] Color coding for notes
- [ ] Markdown support
- [ ] Sort by: date, title, pinned first
- [ ] Archive notes (soft delete)
- [ ] Duplicate note

---

### Step 7: React Frontend
**Todo:**
- [ ] Sidebar with folders
- [ ] Notes list view
- [ ] Note editor (textarea or rich text)
- [ ] Tag input with autocomplete
- [ ] Search bar
- [ ] Color picker
- [ ] Pin button

---

## 💡 Hints & Tips

### Search Implementation:
```javascript
notes.filter(note => 
  note.title.toLowerCase().includes(query.toLowerCase()) ||
  note.content.toLowerCase().includes(query.toLowerCase())
)
```

### Getting Unique Tags:
```javascript
const allTags = notes.flatMap(note => note.tags);
const uniqueTags = [...new Set(allTags)];
```

### Pinned Notes First:
```javascript
notes.sort((a, b) => b.isPinned - a.isPinned);
```

---

## 📚 Concepts

- Nested data structures
- Array filtering
- Text search algorithms
- Tagging systems
- React controlled components

---

**Next:** [Project 9: Bookmark Manager](09-bookmark-manager.md)
