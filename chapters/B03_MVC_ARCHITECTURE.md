# B03: MVC Architecture 🏗️

> **Organize Your Code Like a Pro: Models, Views, Controllers**

**Chapter Level:** Beginner  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B02  
**What You'll Build:** Refactor Todo API into professional MVC structure

---

## 📚 Table of Contents

1. [What is MVC?](#what-is-mvc)
2. [MVC Pattern Explained](#mvc-pattern-explained)
3. [Folder Structure](#folder-structure)
4. [Models Layer](#models-layer)
5. [Controllers Layer](#controllers-layer)
6. [Routes Layer](#routes-layer)
7. [Refactoring Todo App](#refactoring-todo-app)
8. [Best Practices](#best-practices)
9. [Project Task](#project-task)
10. [Interview Questions](#interview-questions)

---

## What is MVC?

### MVC = Model-View-Controller

**MVC** is an **architecture pattern** that organizes code into 3 separate, responsible layers:

```
MVC PATTERN
│
├─ MODEL (Data)
│  └─ Database, data validation, business logic
│
├─ VIEW (Presentation)
│  └─ User interface, how data is displayed
│
└─ CONTROLLER (Logic)
   └─ Handles requests, processes data, sends responses
```

### Why MVC?

```
❌ NO MVC (Spaghetti Code):
- Routes do everything
- Mixed concerns
- Hard to maintain
- Hard to test
- Code duplicated

✅ WITH MVC:
- Clear separation
- Easy to maintain
- Easy to test
- Reusable code
- Professional structure
```

---

## MVC Pattern Explained

### The Flow

```
USER REQUEST
    ↓
ROUTER (Express route)
    ↓
CONTROLLER (Process request)
    ├─ Get data from MODEL
    ├─ Process/validate
    ├─ Call functions
    │
MODEL (Database/Data)
├─ Get data
├─ Validate
├─ Save data
    │
CONTROLLER (Prepare response)
    ├─ Format data
    ├─ Create response
    │
VIEW (Frontend - React)
├─ Receive data
├─ Display to user
    │
USER SEES RESULT
```

### Real-World Analogy

```
RESTAURANT MVC:

MODEL (Kitchen/Storage)
├─ Ingredients stored here
├─ Recipe validation
├─ How to prepare food

CONTROLLER (Waiter)
├─ Takes customer order
├─ Tells kitchen what to make
├─ Gets food from kitchen
├─ Formats plate nicely
├─ Brings to customer

VIEW (Dining Room)
├─ Where customer eats
├─ Nice presentation
├─ Customer sees the dish

ROUTER (Front Desk)
├─ Customer arrives
├─ Directs to waiter (controller)
```

---

## Folder Structure

### Professional MVC Structure

```
my-app/
│
├─ server.js                    # Main entry point
│
├─ config/
│  ├─ database.js              # Database connection config
│  └─ constants.js             # App constants
│
├─ models/
│  ├─ Todo.js                  # Todo data model
│  ├─ User.js                  # User data model
│  └─ Post.js                  # Post data model
│
├─ controllers/
│  ├─ todoController.js        # Todo business logic
│  ├─ userController.js        # User business logic
│  └─ postController.js        # Post business logic
│
├─ routes/
│  ├─ todoRoutes.js            # Todo routes
│  ├─ userRoutes.js            # User routes
│  └─ postRoutes.js            # Post routes
│
├─ middleware/
│  ├─ errorHandler.js          # Error handling middleware
│  ├─ validation.js            # Validation middleware
│  └─ auth.js                  # Authentication middleware
│
├─ utils/
│  ├─ validators.js            # Validation helper functions
│  ├─ helpers.js               # General utilities
│  └─ errors.js                # Custom error classes
│
├─ package.json
├─ .env
├─ .env.example
├─ .gitignore
└─ README.md
```

### Benefit of Structure

- ✅ **Easy to find code** - Know exactly where things are
- ✅ **Easy to maintain** - Change one place only
- ✅ **Easy to test** - Each layer isolated
- ✅ **Scalable** - Add new features without breaking existing
- ✅ **Professional** - Industry standard

---

## Models Layer

### What is a Model?

**Model** = **Represents data and defines how it's stored/validated**

Think: "Shape of data", "Database schema", "Data rules"

### Creating Models

```javascript
// ============================================================
// FILE: models/Todo.js
// PURPOSE: Define Todo data structure and validation
// ============================================================

// EXAMPLE: Simple in-memory model
class Todo {
  // Constructor: Create new todo
  constructor(id, title, description = '', priority = 'medium') {
    this.id = id;
    this.title = title;
    this.description = description;
    this.priority = priority;
    this.completed = false;
    this.createdAt = new Date();
    this.updatedAt = new Date();
  }

  // VALIDATION: Check if todo data is valid
  static validate(data) {
    const errors = [];

    // Validate title
    if (!data.title) {
      errors.push('Title is required');
    } else if (typeof data.title !== 'string') {
      errors.push('Title must be text');
    } else if (data.title.trim().length < 3) {
      errors.push('Title must be at least 3 characters');
    }

    // Validate priority
    if (data.priority) {
      const validPriorities = ['low', 'medium', 'high'];
      if (!validPriorities.includes(data.priority)) {
        errors.push('Priority must be low, medium, or high');
      }
    }

    return errors;
  }

  // CONVERT: Convert to JSON for API response
  toJSON() {
    return {
      id: this.id,
      title: this.title,
      description: this.description,
      priority: this.priority,
      completed: this.completed,
      createdAt: this.createdAt,
      updatedAt: this.updatedAt
    };
  }
}

// EXPORT: Make available to controllers
module.exports = Todo;
```

### In-Memory Database

For now, we'll store data in memory (later use real database):

```javascript
// ============================================================
// FILE: models/todoDatabase.js
// PURPOSE: Store todos in memory (simulate database)
// ============================================================

const Todo = require('./Todo');

class TodoDatabase {
  constructor() {
    // Array to store todos
    this.todos = [];
    // Counter for IDs
    this.nextId = 1;
  }

  // CREATE: Add new todo
  create(title, description, priority) {
    const todo = new Todo(
      String(this.nextId++),
      title,
      description,
      priority
    );
    this.todos.push(todo);
    return todo;
  }

  // READ: Get all todos
  getAll() {
    return this.todos;
  }

  // READ: Get single todo by ID
  getById(id) {
    return this.todos.find(todo => todo.id === id);
  }

  // UPDATE: Update todo
  update(id, data) {
    const todo = this.getById(id);
    if (!todo) return null;

    // Update fields if provided
    if (data.title) todo.title = data.title;
    if (data.description !== undefined) todo.description = data.description;
    if (data.priority) todo.priority = data.priority;
    if (data.completed !== undefined) todo.completed = data.completed;

    todo.updatedAt = new Date();
    return todo;
  }

  // DELETE: Remove todo
  delete(id) {
    const index = this.todos.findIndex(todo => todo.id === id);
    if (index === -1) return null;

    const deleted = this.todos[index];
    this.todos.splice(index, 1);
    return deleted;
  }
}

// EXPORT: Create and export single instance
module.exports = new TodoDatabase();
```

---

## Controllers Layer

### What is a Controller?

**Controller** = **Handles business logic and orchestrates MODEL and RESPONSE**

Think: "Takes request → processes with MODEL → sends response"

### Creating Controllers

```javascript
// ============================================================
// FILE: controllers/todoController.js
// PURPOSE: Handle all todo operations
// ============================================================

const Todo = require('../models/Todo');
const todoDb = require('../models/todoDatabase');

// ============================================================
// CONTROLLER FUNCTION: Get all todos
// ============================================================
const getAllTodos = (req, res) => {
  try {
    // STEP 1: Get filter from query parameters
    const { status, limit = 10, sort = 'newest' } = req.query;

    // STEP 2: Get todos from MODEL
    let todos = todoDb.getAll();

    // STEP 3: FILTER: Apply status filter
    if (status === 'completed') {
      todos = todos.filter(todo => todo.completed === true);
    } else if (status === 'pending') {
      todos = todos.filter(todo => todo.completed === false);
    }

    // STEP 4: SORT: Apply sorting
    if (sort === 'oldest') {
      todos.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
    } else {
      todos.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
    }

    // STEP 5: LIMIT: Apply limit
    todos = todos.slice(0, parseInt(limit));

    // STEP 6: Send response
    res.json({
      success: true,
      message: 'Todos retrieved',
      data: todos.map(todo => todo.toJSON()),
      total: todos.length
    });

  } catch (error) {
    // ERROR HANDLING
    console.error('Error in getAllTodos:', error);
    res.status(500).json({
      success: false,
      message: 'Server error',
      error: error.message
    });
  }
};

// ============================================================
// CONTROLLER FUNCTION: Get single todo
// ============================================================
const getTodoById = (req, res) => {
  try {
    const { id } = req.params;

    // Get from MODEL
    const todo = todoDb.getById(id);

    if (!todo) {
      return res.status(404).json({
        success: false,
        message: 'Todo not found',
        id: id
      });
    }

    res.json({
      success: true,
      data: todo.toJSON()
    });

  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error',
      error: error.message
    });
  }
};

// ============================================================
// CONTROLLER FUNCTION: Create todo
// ============================================================
const createTodo = (req, res) => {
  try {
    const { title, description, priority } = req.body;

    // STEP 1: VALIDATE: Check data using MODEL
    const errors = Todo.validate({ title, priority });

    if (errors.length > 0) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: errors
      });
    }

    // STEP 2: CREATE: Use MODEL to create
    const newTodo = todoDb.create(title, description, priority);

    // STEP 3: RESPONSE: Send back
    res.status(201).json({
      success: true,
      message: 'Todo created',
      data: newTodo.toJSON()
    });

  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error',
      error: error.message
    });
  }
};

// ============================================================
// CONTROLLER FUNCTION: Update todo
// ============================================================
const updateTodo = (req, res) => {
  try {
    const { id } = req.params;
    const updateData = req.body;

    // VALIDATE
    if (updateData.title) {
      const errors = Todo.validate({ title: updateData.title });
      if (errors.length > 0) {
        return res.status(400).json({
          success: false,
          errors: errors
        });
      }
    }

    // UPDATE using MODEL
    const updatedTodo = todoDb.update(id, updateData);

    if (!updatedTodo) {
      return res.status(404).json({
        success: false,
        message: 'Todo not found'
      });
    }

    res.json({
      success: true,
      message: 'Todo updated',
      data: updatedTodo.toJSON()
    });

  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error'
    });
  }
};

// ============================================================
// CONTROLLER FUNCTION: Delete todo
// ============================================================
const deleteTodo = (req, res) => {
  try {
    const { id } = req.params;

    // DELETE using MODEL
    const deleted = todoDb.delete(id);

    if (!deleted) {
      return res.status(404).json({
        success: false,
        message: 'Todo not found'
      });
    }

    res.json({
      success: true,
      message: 'Todo deleted',
      data: deleted.toJSON()
    });

  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error'
    });
  }
};

// ============================================================
// EXPORT: All controller functions
// ============================================================
module.exports = {
  getAllTodos,
  getTodoById,
  createTodo,
  updateTodo,
  deleteTodo
};
```

### Controller Structure

Each controller function should:

1. **Extract data** from request
2. **Validate** using model
3. **Call model** to process
4. **Handle errors**
5. **Send response**

---

## Routes Layer

### What is a Route?

**Route** = **Maps HTTP request to controller function**

Think: "Which controller should handle this request?"

### Creating Routes

```javascript
// ============================================================
// FILE: routes/todoRoutes.js
// PURPOSE: Define all todo routes
// ============================================================

const express = require('express');
const router = express.Router();

// IMPORT: Get controller functions
const todoController = require('../controllers/todoController');

// ============================================================
// ROUTES: Map URLs to controller functions
// ============================================================

// GET all todos
// Route: GET /api/todos
// Handler: todoController.getAllTodos
router.get('/', todoController.getAllTodos);

// GET single todo
// Route: GET /api/todos/:id
router.get('/:id', todoController.getTodoById);

// CREATE todo
// Route: POST /api/todos
// Body: { title, description, priority }
router.post('/', todoController.createTodo);

// UPDATE todo
// Route: PUT /api/todos/:id
router.put('/:id', todoController.updateTodo);

// DELETE todo
// Route: DELETE /api/todos/:id
router.delete('/:id', todoController.deleteTodo);

// EXPORT: Router with all routes
module.exports = router;
```

### Using Routes in Server

```javascript
// ============================================================
// FILE: server.js (Main entry point)
// PURPOSE: Set up Express and register routes
// ============================================================

const express = require('express');
const app = express();

// MIDDLEWARE
app.use(express.json());

// IMPORT ROUTES
const todoRoutes = require('./routes/todoRoutes');

// REGISTER ROUTES
// All routes from todoRoutes will be prefixed with /api/todos
app.use('/api/todos', todoRoutes);

// START SERVER
const PORT = 5000;
app.listen(PORT, () => {
  console.log(`✅ Server running on port ${PORT}`);
});
```

---

## Refactoring Todo App

Let's put it all together:

### Complete MVC Structure

**File 1: Model definition**
```javascript
// models/Todo.js
class Todo {
  constructor(id, title, description = '', priority = 'medium') {
    this.id = id;
    this.title = title;
    this.description = description;
    this.priority = priority;
    this.completed = false;
    this.createdAt = new Date();
    this.updatedAt = new Date();
  }

  static validate(data) {
    const errors = [];
    if (!data.title) errors.push('Title required');
    if (data.title && data.title.length < 3) errors.push('Title too short');
    return errors;
  }

  toJSON() {
    return {
      id: this.id,
      title: this.title,
      description: this.description,
      priority: this.priority,
      completed: this.completed,
      createdAt: this.createdAt,
      updatedAt: this.updatedAt
    };
  }
}

module.exports = Todo;
```

**File 2: Database (Model persistence)**
```javascript
// models/todoDatabase.js
const Todo = require('./Todo');

class TodoDatabase {
  constructor() {
    this.todos = [];
    this.nextId = 1;
  }

  create(title, description, priority) {
    const todo = new Todo(String(this.nextId++), title, description, priority);
    this.todos.push(todo);
    return todo;
  }

  getAll() { return this.todos; }
  getById(id) { return this.todos.find(t => t.id === id); }
  update(id, data) { /* ... */ }
  delete(id) { /* ... */ }
}

module.exports = new TodoDatabase();
```

**File 3: Controller (Business logic)**
```javascript
// controllers/todoController.js
const Todo = require('../models/Todo');
const todoDb = require('../models/todoDatabase');

module.exports = {
  getAllTodos: (req, res) => {
    try {
      const todos = todoDb.getAll();
      res.json({ success: true, data: todos.map(t => t.toJSON()) });
    } catch (e) {
      res.status(500).json({ success: false, error: e.message });
    }
  },

  createTodo: (req, res) => {
    try {
      const { title, description, priority } = req.body;
      const errors = Todo.validate({ title, priority });
      if (errors.length > 0) {
        return res.status(400).json({ success: false, errors });
      }
      const todo = todoDb.create(title, description, priority);
      res.status(201).json({ success: true, data: todo.toJSON() });
    } catch (e) {
      res.status(500).json({ success: false, error: e.message });
    }
  },

  getTodoById: (req, res) => {
    const { id } = req.params;
    const todo = todoDb.getById(id);
    if (!todo) return res.status(404).json({ success: false });
    res.json({ success: true, data: todo.toJSON() });
  },

  updateTodo: (req, res) => {
    const { id } = req.params;
    const todo = todoDb.update(id, req.body);
    if (!todo) return res.status(404).json({ success: false });
    res.json({ success: true, data: todo.toJSON() });
  },

  deleteTodo: (req, res) => {
    const { id } = req.params;
    const todo = todoDb.delete(id);
    if (!todo) return res.status(404).json({ success: false });
    res.json({ success: true, data: todo.toJSON() });
  }
};
```

**File 4: Routes (URL mapping)**
```javascript
// routes/todoRoutes.js
const express = require('express');
const controller = require('../controllers/todoController');

const router = express.Router();

router.get('/', controller.getAllTodos);
router.post('/', controller.createTodo);
router.get('/:id', controller.getTodoById);
router.put('/:id', controller.updateTodo);
router.delete('/:id', controller.deleteTodo);

module.exports = router;
```

**File 5: Server setup**
```javascript
// server.js
const express = require('express');
const app = express();

app.use(express.json());

const todoRoutes = require('./routes/todoRoutes');
app.use('/api/todos', todoRoutes);

app.listen(5000, () => {
  console.log('✅ Server running on port 5000');
});
```

### Benefits of This Structure

✅ **Easy to maintain** - Each file has one responsibility  
✅ **Easy to test** - Can test models and controllers separately  
✅ **Easy to scale** - Add new features without breaking existing  
✅ **Professional** - Industry standard structure  
✅ **Reusable** - Can reuse models/controllers in other projects  

---

## Best Practices

### 1. **Thin Routes, Fat Controllers**
```javascript
// ❌ BAD: Logic in routes
app.post('/api/todos', (req, res) => {
  const { title } = req.body;
  if (!title) { /* validation */ }
  // ... more logic
  res.json(todo);
});

// ✅ GOOD: Logic in controllers
app.post('/api/todos', todoController.create);
```

### 2. **Use Models for Validation**
```javascript
// ❌ BAD: Validation in controller
const validateTodo = (title) => { /* ... */ };

// ✅ GOOD: Validation in model
class Todo {
  static validate(data) { /* ... */ }
}
```

### 3. **Separate Concerns**
```
Models    = Data & Validation
Routes    = URL mapping
Controllers = Business logic
Views     = Presentation (React)
```

### 4. **Reusable Functions**
```javascript
// ✅ GOOD: Reusable validation
const errors = Todo.validate(data);
// Use in controller, tests, other places
```

---

## Project Task

### 🎯 Refactor Blog API to MVC

**Time:** 3-4 hours

**Requirements:**

1. **Create folder structure**
   - models/, controllers/, routes/

2. **Create Models**
   - Post model with validation
   - PostDatabase for storage

3. **Create Controllers**
   - getAllPosts, getPostById, createPost, updatePost, deletePost

4. **Create Routes**
   - Map all endpoints to controllers

5. **Update server.js**
   - Import and use routes

6. **Frontend integration**
   - React should still work same way
   - Update API calls if URLs changed

---

## Interview Questions

**Q1: Why use MVC pattern?**
<details>
<summary>Answer</summary>

Benefits:
- **Separation of Concerns** - Each layer has one job
- **Maintainability** - Easy to find and fix bugs
- **Scalability** - Add features without breaking existing
- **Testability** - Can test each layer independently
- **Reusability** - Can reuse models/controllers
- **Professional** - Industry standard
</details>

**Q2: What goes in Model vs Controller?**
<details>
<summary>Answer</summary>

- **Model:** Data structure, validation, storage operations
- **Controller:** Request handling, orchestrating model operations, formatting responses
</details>

---

## Key Takeaways

✅ **MVC** = Model-View-Controller architecture  
✅ **Model** = Data and validation  
✅ **Controller** = Business logic  
✅ **Routes** = URL to controller mapping  
✅ **Separation** = Each layer has one responsibility  
✅ **Professional** = Industry standard structure  

---

| Previous | Index | Next |
|----------|-------|------|
| [B02: Express & Routing](B02_EXPRESS_AND_ROUTING.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B04: MongoDB Basics](B04_MONGODB_BASICS.md) |

---

**Author:** Tushar Parlikar  
**Last Updated:** October 20, 2025
