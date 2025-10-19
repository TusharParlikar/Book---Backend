# B04: MongoDB Basics 🗄️

> **Learn NoSQL Databases: Store, Query, and Manage Data**

**Chapter Level:** Beginner  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B03  
**What You'll Build:** Todo app with MongoDB persistence

---

## 📚 Core Concepts

### What is MongoDB?

**MongoDB** = **NoSQL database that stores data as documents (JSON-like)**

Compared to traditional SQL:
- **SQL:** Structured tables with rows/columns
- **MongoDB:** Flexible documents in collections

### Setup

```bash
# Install MongoDB locally
# OR use MongoDB Atlas (cloud): https://www.mongodb.com/cloud/atlas

# Install MongoDB driver for Node.js
npm install mongodb mongoose
```

### Basic Operations

#### Connect to MongoDB

```javascript
// ============================================================
// FILE: config/database.js
// PURPOSE: Connect to MongoDB
// ============================================================

const mongoose = require('mongoose');

// CONNECTION STRING
// Format: mongodb+srv://username:password@cluster.mongodb.net/database
// Local: mongodb://localhost:27017/database-name
const MONGO_URI = process.env.MONGO_URI || 'mongodb://localhost:27017/todo-app';

// CONNECT
mongoose.connect(MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
  .then(() => {
    console.log('✅ Connected to MongoDB');
  })
  .catch((error) => {
    console.error('❌ MongoDB connection error:', error);
  });

module.exports = mongoose;
```

#### Schemas (Define Structure)

```javascript
// ============================================================
// FILE: models/todoSchema.js
// PURPOSE: Define todo document structure
// ============================================================

const mongoose = require('mongoose');

// SCHEMA: Define what fields a todo has
const todoSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: [true, 'Title is required'],
      minlength: [3, 'Title must be at least 3 characters'],
      maxlength: [200, 'Title cannot exceed 200 characters'],
      trim: true
    },
    description: {
      type: String,
      default: ''
    },
    priority: {
      type: String,
      enum: ['low', 'medium', 'high'],
      default: 'medium'
    },
    completed: {
      type: Boolean,
      default: false
    },
    tags: {
      type: [String],
      default: []
    }
  },
  {
    timestamps: true  // Auto-adds createdAt and updatedAt
  }
);

// MODEL: Create model from schema
const Todo = mongoose.model('Todo', todoSchema);

module.exports = Todo;
```

#### CRUD Operations

```javascript
// ============================================================
// MONGODB CRUD OPERATIONS
// ============================================================

// 1. CREATE (INSERT)
const newTodo = await Todo.create({
  title: 'Learn MongoDB',
  description: 'Master NoSQL databases',
  priority: 'high'
});

// 2. READ (FIND)
const allTodos = await Todo.find();
const oneTodo = await Todo.findById(id);
const filtered = await Todo.find({ priority: 'high' });

// 3. UPDATE
const updated = await Todo.findByIdAndUpdate(
  id,
  { title: 'New title', completed: true },
  { new: true }  // Return updated document
);

// 4. DELETE
const deleted = await Todo.findByIdAndDelete(id);
```

---

## Complete MongoDB Example

```javascript
// ============================================================
// FILE: models/Todo.js
// ============================================================

const mongoose = require('mongoose');

const todoSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    minlength: 3
  },
  description: String,
  priority: {
    type: String,
    enum: ['low', 'medium', 'high'],
    default: 'medium'
  },
  completed: {
    type: Boolean,
    default: false
  }
}, { timestamps: true });

module.exports = mongoose.model('Todo', todoSchema);


// ============================================================
// FILE: controllers/todoController.js (Updated with MongoDB)
// ============================================================

const Todo = require('../models/Todo');

module.exports = {
  // GET all todos
  getAllTodos: async (req, res) => {
    try {
      const todos = await Todo.find();
      res.json({ success: true, data: todos });
    } catch (error) {
      res.status(500).json({ success: false, error: error.message });
    }
  },

  // GET single todo
  getTodoById: async (req, res) => {
    try {
      const todo = await Todo.findById(req.params.id);
      if (!todo) {
        return res.status(404).json({ success: false, message: 'Not found' });
      }
      res.json({ success: true, data: todo });
    } catch (error) {
      res.status(500).json({ success: false, error: error.message });
    }
  },

  // CREATE todo
  createTodo: async (req, res) => {
    try {
      const todo = await Todo.create(req.body);
      res.status(201).json({ success: true, data: todo });
    } catch (error) {
      res.status(400).json({ success: false, error: error.message });
    }
  },

  // UPDATE todo
  updateTodo: async (req, res) => {
    try {
      const todo = await Todo.findByIdAndUpdate(
        req.params.id,
        req.body,
        { new: true }
      );
      if (!todo) {
        return res.status(404).json({ success: false });
      }
      res.json({ success: true, data: todo });
    } catch (error) {
      res.status(400).json({ success: false, error: error.message });
    }
  },

  // DELETE todo
  deleteTodo: async (req, res) => {
    try {
      const todo = await Todo.findByIdAndDelete(req.params.id);
      if (!todo) {
        return res.status(404).json({ success: false });
      }
      res.json({ success: true, message: 'Deleted', data: todo });
    } catch (error) {
      res.status(500).json({ success: false, error: error.message });
    }
  }
};


// ============================================================
// FILE: server.js
// ============================================================

const express = require('express');
const mongoose = require('mongoose');
const app = express();

// MIDDLEWARE
app.use(express.json());

// CONNECT TO MONGODB
mongoose.connect('mongodb://localhost:27017/todo-app')
  .then(() => console.log('✅ MongoDB connected'))
  .catch(err => console.error('❌ MongoDB error:', err));

// ROUTES
const todoRoutes = require('./routes/todoRoutes');
app.use('/api/todos', todoRoutes);

// START
app.listen(5000, () => console.log('✅ Server on port 5000'));
```

---

## Interview Questions

**Q1: What's the difference between SQL and NoSQL?**

SQL: Rigid schema, tables, rows/columns, relational  
NoSQL: Flexible, documents, JSON-like, non-relational

**Q2: When would you use MongoDB?**

- Flexible data structure
- Rapid development
- Big data/scalability
- Real-time applications

---

| Previous | Index | Next |
|----------|-------|------|
| [B03: MVC Architecture](B03_MVC_ARCHITECTURE.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B05: Mongoose Schemas](B05_MONGOOSE_SCHEMAS.md) |
