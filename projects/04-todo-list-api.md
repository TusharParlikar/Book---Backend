# Project 4: Todo List API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 2-3 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build a complete CRUD API for a todo list. This is the classic project that teaches all HTTP methods: GET, POST, PUT, DELETE.

**What You'll Learn:**
- All CRUD operations
- Request body handling
- Data validation
- Status codes
- Array manipulation

---

## 🎯 Project Goals

- ✅ Create, Read, Update, Delete todos
- ✅ Mark todos as complete/incomplete
- ✅ Filter by status
- ✅ Input validation
- ✅ React todo app with full functionality

---

## 🗺️ Roadmap

### Step 1: Setup & Data Structure
**Todo:**
- [ ] Initialize Express project
- [ ] Install express, cors, uuid (for unique IDs)
- [ ] Create data structure for todos:
  - id (unique string)
  - title (string, required)
  - description (string, optional)
  - completed (boolean, default false)
  - createdAt (date timestamp)
  - updatedAt (date timestamp)
- [ ] Store todos in array (in-memory for now)

**Hint:** Use `uuid` package for unique IDs: `npm install uuid`

---

### Step 2: Implement GET Routes
**Todo:**
- [ ] `GET /api/todos` - Get all todos
- [ ] `GET /api/todos/:id` - Get single todo
- [ ] `GET /api/todos?completed=true` - Filter completed
- [ ] `GET /api/todos?completed=false` - Filter pending
- [ ] Return 404 if todo not found

**Hint:** Use `array.find()` for single todo, `array.filter()` for filtering

---

### Step 3: Implement POST Route
**Todo:**
- [ ] `POST /api/todos` - Create new todo
- [ ] Accept JSON body with title and description
- [ ] Validate: title is required and not empty
- [ ] Generate unique ID
- [ ] Set createdAt and updatedAt timestamps
- [ ] Set completed to false by default
- [ ] Return created todo with 201 status

**Validation Rules:**
- Title: required, min 3 characters, max 100 characters
- Description: optional, max 500 characters

**Hint:** Access request body with `req.body` (needs `express.json()` middleware)

---

### Step 4: Implement PUT Route
**Todo:**
- [ ] `PUT /api/todos/:id` - Update entire todo
- [ ] `PATCH /api/todos/:id` - Partial update
- [ ] `PATCH /api/todos/:id/toggle` - Toggle completed status
- [ ] Find todo by ID
- [ ] Update fields
- [ ] Update updatedAt timestamp
- [ ] Return updated todo

**Hint:** PUT replaces entire object, PATCH updates specific fields

---

### Step 5: Implement DELETE Route
**Todo:**
- [ ] `DELETE /api/todos/:id` - Delete todo
- [ ] Find todo by ID
- [ ] Remove from array
- [ ] Return success message
- [ ] Return 404 if not found

**Hint:** Use `array.filter()` to remove item: `todos = todos.filter(t => t.id !== id)`

---

### Step 6: Add Advanced Features
**Todo:**
- [ ] Sort todos (newest first, oldest first)
- [ ] Pagination (page & limit query params)
- [ ] Search in title/description
- [ ] Get statistics (total, completed, pending)
- [ ] Bulk delete completed todos

**Hint for Pagination:**
```
page = req.query.page || 1
limit = req.query.limit || 10
startIndex = (page - 1) * limit
endIndex = page * limit
result = todos.slice(startIndex, endIndex)
```

---

### Step 7: React Frontend
**Todo:**
- [ ] Create todo list component
- [ ] Input field to add new todo
- [ ] Display all todos
- [ ] Checkbox to mark complete
- [ ] Delete button for each todo
- [ ] Edit functionality
- [ ] Filter: All / Active / Completed
- [ ] Show count of pending todos

**React Features:**
- Form submission
- Optimistic UI updates
- Conditional rendering
- Array mapping
- Filter functionality

---

## 💡 Hints & Tips

### Data Validation:
```javascript
if (!title || title.trim().length === 0) {
  return res.status(400).json({ error: 'Title is required' });
}
```

### Generating Unique IDs:
```javascript
const { v4: uuidv4 } = require('uuid');
const id = uuidv4(); // generates unique string
```

### Toggle Complete Status:
```javascript
todo.completed = !todo.completed;
todo.updatedAt = new Date().toISOString();
```

### React DELETE with Confirmation:
```javascript
if (window.confirm('Delete this todo?')) {
  // Make DELETE request
}
```

---

## 🎨 Enhancement Ideas

**Easy:**
- Add priority levels (high, medium, low)
- Add due dates
- Add categories/tags
- Color code by priority

**Medium:**
- Drag and drop to reorder
- Duplicate todo
- Archive completed todos
- Undo delete

**Advanced:**
- Connect to MongoDB (Chapter B04)
- User authentication
- Multiple todo lists
- Shared todos between users
- Real-time updates with Socket.io

---

## ✅ Completion Checklist

**Backend:**
- [ ] GET all todos works
- [ ] GET single todo works
- [ ] POST creates todo correctly
- [ ] PUT/PATCH updates todo
- [ ] DELETE removes todo
- [ ] Validation working
- [ ] Proper status codes
- [ ] Error handling complete

**Frontend:**
- [ ] Can add todos
- [ ] Can view all todos
- [ ] Can mark complete
- [ ] Can delete todos
- [ ] Can edit todos
- [ ] Filter by status works
- [ ] UI is clean and intuitive

**Integration:**
- [ ] Frontend and backend connected
- [ ] All CRUD operations work
- [ ] No CORS errors
- [ ] Loading states implemented
- [ ] Error messages shown

---

## 📚 Concepts You'll Practice

- Full CRUD operations
- HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Request body parsing
- URL parameters
- Query parameters
- Data validation
- Array manipulation
- Status codes
- React forms and state

---

## 🔗 Related Resources

**Chapters:**
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)
- [B03: MVC Architecture](../chapters/B03_MVC_ARCHITECTURE.md)

**External:**
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [UUID Package](https://www.npmjs.com/package/uuid)
- [Array Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

---

**Next:** [Project 5: Blog Post API](05-blog-post-api.md)
