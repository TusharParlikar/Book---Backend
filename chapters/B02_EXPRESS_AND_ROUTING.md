# B02: Express & Routing 🛣️

> **Master Express Routes, HTTP Methods, and Building Multi-Endpoint APIs**

**Chapter Level:** Beginner  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01, basic JavaScript  
**What You'll Build:** A complete Todo API with full CRUD operations

---

## 📚 Table of Contents

1. [What is Routing?](#what-is-routing)
2. [HTTP Methods Deep Dive](#http-methods-deep-dive)
3. [Route Parameters](#route-parameters)
4. [Query Parameters](#query-parameters)
5. [Request Body (POST/PUT)](#request-body)
6. [Status Codes & Responses](#status-codes-and-responses)
7. [Building a Todo API](#building-a-todo-api)
8. [React Frontend Integration](#react-frontend-integration)
9. [Error Handling & Validation](#error-handling-validation)
10. [Best Practices](#best-practices)
11. [Project Task](#project-task)
12. [Interview Questions](#interview-questions)

---

## What is Routing?

### Definition

**Routing** = **Matching URLs to handler functions**

When a request comes to your server, Express looks at:
1. **HTTP Method** (GET, POST, PUT, DELETE)
2. **URL Path** (/api/users, /api/users/123, etc.)
3. **Runs the matching handler function**

### Analogy

Imagine a restaurant's phone line:

```
CUSTOMER CALLS (HTTP Request)
├─ Says action: "I want to PLACE an order" (GET/POST/PUT/DELETE)
├─ Says what: "Table for 2 at 7 PM" (URL path)
│
RECEPTIONIST (Express Router)
├─ Checks action type
├─ Checks what they want
├─ Connects to right department (handler function)
│
KITCHEN/MANAGER (Handler function)
├─ Takes the order
├─ Processes it
├─ Sends back confirmation (response)
```

### Basic Routing Syntax

```javascript
// Syntax: app.METHOD(PATH, HANDLER)

app.get('/api/todos', (req, res) => {
  // GET request received
});

app.post('/api/todos', (req, res) => {
  // POST request received
});

app.put('/api/todos/:id', (req, res) => {
  // PUT request received with ID parameter
});

app.delete('/api/todos/:id', (req, res) => {
  // DELETE request received with ID parameter
});
```

---

## HTTP Methods Deep Dive

### The Four Main Methods

#### 1. **GET** - Retrieve Data (Read)
```javascript
// WHAT: Get all items
app.get('/api/todos', (req, res) => {
  // Read from database
  // DON'T modify anything
  // Send back data
});

// WHAT: Get specific item by ID
app.get('/api/todos/:id', (req, res) => {
  // Read ONE item from database
  // Send it back
});

// CHARACTERISTICS:
// - Should NOT modify data
// - Browser can make this request
// - Data in URL (visible)
// - No body/payload
```

#### 2. **POST** - Create New Data
```javascript
// WHAT: Create new item
app.post('/api/todos', (req, res) => {
  // Receive data from request body
  // Validate it
  // Save to database
  // Send back created item with ID
});

// CHARACTERISTICS:
// - Creates new resource
// - Body contains data
// - Server assigns ID/timestamp
// - Returns 201 Created
// - Idempotent: NOT idempotent (calling twice = 2 new items)
```

#### 3. **PUT** - Replace Entire Data
```javascript
// WHAT: Replace entire item
app.put('/api/todos/:id', (req, res) => {
  // Receive new complete data
  // Replace ALL fields of item with ID
  // Send back updated item
});

// CHARACTERISTICS:
// - Replaces complete resource
// - Need to provide ALL fields
// - Body contains complete new data
// - Returns 200 OK
// - Idempotent: YES (calling twice = same result)
```

#### 4. **PATCH** - Partially Update Data
```javascript
// WHAT: Update only some fields
app.patch('/api/todos/:id', (req, res) => {
  // Receive partial data
  // Update only provided fields
  // Keep other fields unchanged
  // Send back updated item
});

// CHARACTERISTICS:
// - Partially updates resource
// - Only send fields to update
// - Body contains only changes
// - Returns 200 OK
// - Idempotent: YES
```

#### 5. **DELETE** - Remove Data
```javascript
// WHAT: Delete item
app.delete('/api/todos/:id', (req, res) => {
  // Find item with ID
  // Remove from database
  // Send back confirmation or updated list
});

// CHARACTERISTICS:
// - Removes resource
// - No body needed
// - Idempotent: YES (deleting twice, second is "already deleted")
```

### Comparison Table

| Method | Purpose | Modifies Data | Body | Idempotent | Status | Example |
|--------|---------|---------------|------|-----------|--------|---------|
| **GET** | Read | ❌ No | ❌ No | ✅ Yes | 200 | `/api/todos` |
| **POST** | Create | ✅ Yes | ✅ Yes | ❌ No | 201 | `/api/todos` (body: new todo) |
| **PUT** | Replace | ✅ Yes | ✅ Yes | ✅ Yes | 200 | `/api/todos/1` (body: full todo) |
| **PATCH** | Update | ✅ Yes | ✅ Yes | ✅ Yes | 200 | `/api/todos/1` (body: {title: "new"}) |
| **DELETE** | Remove | ✅ Yes | ❌ No | ✅ Yes | 204 | `/api/todos/1` |

---

## Route Parameters

### What Are Route Parameters?

**Route parameters** = **Variables in the URL path**

```javascript
// URL pattern with parameter
app.get('/api/todos/:id', (req, res) => {
  // :id is a PARAMETER
  // User might request: /api/todos/123
  // Then req.params.id = '123'
});
```

### Using Route Parameters

```javascript
// ============================================================
// EXAMPLE: Get a specific todo by ID
// ============================================================

app.get('/api/todos/:id', (req, res) => {
  // STEP 1: Get ID from URL
  // URL: /api/todos/123
  // req.params.id = '123'
  const { id } = req.params;
  
  console.log(`Fetching todo with ID: ${id}`);
  
  // STEP 2: Find todo in database
  // (For now, we'll use fake data)
  const todo = {
    id: id,
    title: 'Learn Express Routing',
    completed: false,
    createdAt: new Date()
  };
  
  // STEP 3: Send response
  res.json({
    success: true,
    data: todo
  });
});
```

### Multiple Route Parameters

```javascript
// ============================================================
// PATTERN: Multiple parameters in one URL
// ============================================================

app.get('/api/users/:userId/todos/:todoId', (req, res) => {
  const { userId, todoId } = req.params;
  
  console.log(`User: ${userId}, Todo: ${todoId}`);
  
  // Find specific todo for specific user
  res.json({
    success: true,
    data: {
      userId: userId,
      todoId: todoId,
      title: 'Do homework'
    }
  });
});

// Request: /api/users/5/todos/10
// Response: { userId: '5', todoId: '10', ... }
```

### Wildcard Parameters

```javascript
// ============================================================
// PATTERN: Match everything after /files/
// ============================================================

app.get('/files/:filename', (req, res) => {
  const { filename } = req.params;
  
  // Could be:
  // /files/document.pdf
  // /files/image.png
  // /files/folder/subfolder/file.txt
  
  res.json({ file: filename });
});
```

---

## Query Parameters

### What Are Query Parameters?

**Query parameters** = **Key-value pairs in URL after `?`**

```javascript
// URL: /api/todos?status=completed&limit=10

// Query parameters:
// status = 'completed'
// limit = '10'

app.get('/api/todos', (req, res) => {
  // Access with req.query
  const { status, limit } = req.query;
  
  console.log(status); // 'completed'
  console.log(limit);  // '10'
});
```

### Examples

```
GET /api/users?role=admin
→ req.query.role = 'admin'

GET /api/todos?status=completed&priority=high
→ req.query.status = 'completed'
→ req.query.priority = 'high'

GET /api/posts?page=2&limit=20
→ req.query.page = '2'
→ req.query.limit = '20'
```

### Using Query Parameters

```javascript
// ============================================================
// EXAMPLE: Filter todos by status
// ============================================================

app.get('/api/todos', (req, res) => {
  // Get query parameters
  const { status, limit = 10 } = req.query;
  
  // Default: all todos
  let todos = [
    { id: 1, title: 'Learn Express', status: 'in-progress' },
    { id: 2, title: 'Build API', status: 'completed' },
    { id: 3, title: 'Deploy', status: 'pending' }
  ];
  
  // FILTER: If status parameter provided
  if (status) {
    todos = todos.filter(todo => todo.status === status);
    console.log(`Filtered to status: ${status}`);
  }
  
  // LIMIT: Take only first N items
  todos = todos.slice(0, parseInt(limit));
  
  res.json({
    success: true,
    data: todos,
    total: todos.length
  });
});

// USAGE:
// GET /api/todos
// → Returns all todos, max 10

// GET /api/todos?status=completed
// → Returns only completed todos

// GET /api/todos?status=completed&limit=5
// → Returns max 5 completed todos
```

### Difference: Parameters vs Query

| Type | In URL | Example | Access |
|------|--------|---------|--------|
| **Route Parameter** | Path (required) | `/api/todos/:id` | `req.params.id` |
| **Query Parameter** | After `?` (optional) | `/api/todos?limit=10` | `req.query.limit` |

```javascript
// Route Parameters: /api/todos/:id/:section
app.get('/api/todos/:id/:section', (req, res) => {
  // req.params = { id: '123', section: 'details' }
});
// Request: /api/todos/123/details

// Query Parameters: /api/todos?filter=done&sort=date
app.get('/api/todos', (req, res) => {
  // req.query = { filter: 'done', sort: 'date' }
});
// Request: /api/todos?filter=done&sort=date

// BOTH: /api/todos/:id?details=full
app.get('/api/todos/:id', (req, res) => {
  // req.params = { id: '123' }
  // req.query = { details: 'full' }
});
// Request: /api/todos/123?details=full
```

---

## Request Body

### What is Request Body?

**Request body** = **Data sent with POST/PUT/PATCH requests**

When frontend sends complex data, it goes in the body:

```javascript
// Frontend sends:
axios.post('/api/todos', {
  title: 'Buy groceries',
  description: 'Milk, eggs, bread',
  priority: 'high'
});

// Backend receives in req.body:
{
  title: 'Buy groceries',
  description: 'Milk, eggs, bread',
  priority: 'high'
}
```

### Setup: Parse JSON

```javascript
// Must add this BEFORE your routes!
const express = require('express');
const app = express();

// MIDDLEWARE: Enable JSON parsing
// This converts incoming JSON string to JavaScript object
app.use(express.json());

// Now you can use req.body in routes
```

### Using Request Body

```javascript
// ============================================================
// EXAMPLE: Create new todo with POST
// ============================================================

app.post('/api/todos', (req, res) => {
  // STEP 1: Get data from request body
  const { title, description, priority } = req.body;
  
  console.log('Received todo:', { title, description, priority });
  
  // STEP 2: Validate data exists
  if (!title) {
    return res.status(400).json({
      success: false,
      message: 'Title is required'
    });
  }
  
  // STEP 3: Validate data types/values
  if (typeof title !== 'string') {
    return res.status(400).json({
      success: false,
      message: 'Title must be text'
    });
  }
  
  if (title.length < 3) {
    return res.status(400).json({
      success: false,
      message: 'Title must be at least 3 characters'
    });
  }
  
  // STEP 4: Create todo object
  const newTodo = {
    id: Math.random().toString(36).substr(2, 9), // Random ID
    title: title,
    description: description || '',  // Default to empty string
    priority: priority || 'medium',   // Default to medium
    completed: false,
    createdAt: new Date()
  };
  
  // STEP 5: Save to database (later)
  // For now, just return it
  
  // STEP 6: Send back with 201 Created status
  res.status(201).json({
    success: true,
    message: 'Todo created successfully',
    data: newTodo
  });
});

// ============================================================
// EXAMPLE: Update existing todo with PUT
// ============================================================

app.put('/api/todos/:id', (req, res) => {
  const { id } = req.params;
  const { title, description, priority, completed } = req.body;
  
  // Validate new data
  if (!title) {
    return res.status(400).json({
      success: false,
      message: 'Title is required'
    });
  }
  
  // Updated todo object
  const updatedTodo = {
    id: id,
    title: title,
    description: description || '',
    priority: priority || 'medium',
    completed: completed || false,
    updatedAt: new Date()
  };
  
  res.json({
    success: true,
    message: 'Todo updated successfully',
    data: updatedTodo
  });
});
```

---

## Status Codes & Responses

### HTTP Status Codes

Express lets you set status codes to tell the client what happened:

```javascript
// 2xx: Success
200 OK                    // Request succeeded
201 Created              // New resource created
204 No Content           // Success, no body to return

// 3xx: Redirect
301 Moved Permanently    // Resource moved
304 Not Modified         // Use cached version

// 4xx: Client Error
400 Bad Request          // Invalid data from client
401 Unauthorized         // Need authentication
403 Forbidden            // Authenticated but not allowed
404 Not Found            // Resource doesn't exist
409 Conflict             // Resource already exists

// 5xx: Server Error
500 Internal Server Error // Server error
503 Service Unavailable  // Server temporarily down
```

### Using Status Codes

```javascript
// ============================================================
// EXAMPLE: Complete response with status codes
// ============================================================

app.get('/api/todos/:id', (req, res) => {
  const { id } = req.params;
  
  // Pretend to fetch from database
  const todo = null; // Not found
  
  if (!todo) {
    // Return 404 with error message
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
      id: id
    });
  }
  
  // Return 200 with data (200 is default, no need to specify)
  res.json({
    success: true,
    message: 'Todo retrieved',
    data: todo
  });
});

// ============================================================
// EXAMPLE: All CRUD operations with proper status codes
// ============================================================

app.get('/api/todos', (req, res) => {
  // 200 OK (default)
  res.json({
    success: true,
    data: [/* todos */]
  });
});

app.post('/api/todos', (req, res) => {
  // 201 Created
  res.status(201).json({
    success: true,
    data: { /* new todo */ }
  });
});

app.put('/api/todos/:id', (req, res) => {
  // 200 OK
  res.json({
    success: true,
    data: { /* updated todo */ }
  });
});

app.patch('/api/todos/:id', (req, res) => {
  // 200 OK
  res.json({
    success: true,
    data: { /* partially updated todo */ }
  });
});

app.delete('/api/todos/:id', (req, res) => {
  // 204 No Content (success but no body)
  // OR 200 OK with message
  res.status(204).send();
  
  // Alternative:
  // res.json({
  //   success: true,
  //   message: 'Todo deleted'
  // });
});
```

---

## Building a Todo API

Let's build a **complete Todo API** with all CRUD operations:

```javascript
// ============================================================
// FILE: server.js
// PURPOSE: Complete Todo API with all routes
// ============================================================

const express = require('express');
const app = express();

// MIDDLEWARE: Enable JSON parsing
app.use(express.json());

// ============================================================
// DATA: In-memory storage (later replaced with database)
// ============================================================

let todos = [
  { id: '1', title: 'Learn Express', description: 'Master routing', completed: false, createdAt: new Date() },
  { id: '2', title: 'Build API', description: 'Create REST API', completed: false, createdAt: new Date() }
];

// ============================================================
// HELPER FUNCTION: Generate unique ID
// ============================================================

function generateId() {
  return Math.random().toString(36).substr(2, 9);
}

// ============================================================
// ROUTE 1: GET all todos
// ============================================================

app.get('/api/todos', (req, res) => {
  // STEP 1: Get query parameters for filtering
  const { status, limit = 10, sort = 'newest' } = req.query;
  
  // STEP 2: Filter todos
  let filteredTodos = todos;
  
  if (status === 'completed') {
    filteredTodos = todos.filter(todo => todo.completed === true);
  } else if (status === 'pending') {
    filteredTodos = todos.filter(todo => todo.completed === false);
  }
  
  // STEP 3: Sort todos
  if (sort === 'oldest') {
    filteredTodos.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt));
  } else if (sort === 'newest') {
    filteredTodos.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
  }
  
  // STEP 4: Apply limit
  filteredTodos = filteredTodos.slice(0, parseInt(limit));
  
  // STEP 5: Send response
  res.json({
    success: true,
    message: 'Todos retrieved',
    data: filteredTodos,
    total: filteredTodos.length
  });
});

// ============================================================
// ROUTE 2: GET single todo by ID
// ============================================================

app.get('/api/todos/:id', (req, res) => {
  const { id } = req.params;
  
  // Find todo with this ID
  const todo = todos.find(t => t.id === id);
  
  // Check if found
  if (!todo) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
      id: id
    });
  }
  
  // Return the todo
  res.json({
    success: true,
    message: 'Todo retrieved',
    data: todo
  });
});

// ============================================================
// ROUTE 3: CREATE new todo
// ============================================================

app.post('/api/todos', (req, res) => {
  // STEP 1: Get data from request body
  const { title, description, priority } = req.body;
  
  // STEP 2: Validate required fields
  if (!title) {
    return res.status(400).json({
      success: false,
      message: 'Title is required'
    });
  }
  
  // STEP 3: Validate field values
  if (typeof title !== 'string' || title.trim() === '') {
    return res.status(400).json({
      success: false,
      message: 'Title must be non-empty text'
    });
  }
  
  if (title.length < 3) {
    return res.status(400).json({
      success: false,
      message: 'Title must be at least 3 characters long'
    });
  }
  
  // STEP 4: Create new todo object
  const newTodo = {
    id: generateId(),
    title: title.trim(),
    description: description || '',
    priority: priority || 'medium',
    completed: false,
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  // STEP 5: Add to todos array
  todos.push(newTodo);
  
  // STEP 6: Send response with 201 Created status
  res.status(201).json({
    success: true,
    message: 'Todo created successfully',
    data: newTodo
  });
});

// ============================================================
// ROUTE 4: UPDATE entire todo (PUT)
// ============================================================

app.put('/api/todos/:id', (req, res) => {
  const { id } = req.params;
  const { title, description, priority, completed } = req.body;
  
  // STEP 1: Find the todo
  const todoIndex = todos.findIndex(t => t.id === id);
  
  if (todoIndex === -1) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
      id: id
    });
  }
  
  // STEP 2: Validate new title
  if (!title) {
    return res.status(400).json({
      success: false,
      message: 'Title is required'
    });
  }
  
  // STEP 3: Update entire todo
  todos[todoIndex] = {
    ...todos[todoIndex],
    title: title,
    description: description !== undefined ? description : todos[todoIndex].description,
    priority: priority || todos[todoIndex].priority,
    completed: completed !== undefined ? completed : todos[todoIndex].completed,
    updatedAt: new Date()
  };
  
  // STEP 4: Send updated todo
  res.json({
    success: true,
    message: 'Todo updated successfully',
    data: todos[todoIndex]
  });
});

// ============================================================
// ROUTE 5: PARTIALLY update todo (PATCH)
// ============================================================

app.patch('/api/todos/:id', (req, res) => {
  const { id } = req.params;
  const updates = req.body;  // Only updated fields
  
  // STEP 1: Find the todo
  const todoIndex = todos.findIndex(t => t.id === id);
  
  if (todoIndex === -1) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
      id: id
    });
  }
  
  // STEP 2: Update only provided fields
  // Use spread operator to merge updates
  todos[todoIndex] = {
    ...todos[todoIndex],
    ...updates,
    updatedAt: new Date()
  };
  
  // STEP 3: Send updated todo
  res.json({
    success: true,
    message: 'Todo updated successfully',
    data: todos[todoIndex]
  });
});

// ============================================================
// ROUTE 6: DELETE todo
// ============================================================

app.delete('/api/todos/:id', (req, res) => {
  const { id } = req.params;
  
  // STEP 1: Find todo index
  const todoIndex = todos.findIndex(t => t.id === id);
  
  if (todoIndex === -1) {
    return res.status(404).json({
      success: false,
      message: 'Todo not found',
      id: id
    });
  }
  
  // STEP 2: Remove from array
  const deletedTodo = todos.splice(todoIndex, 1)[0];
  
  // STEP 3: Send confirmation
  res.json({
    success: true,
    message: 'Todo deleted successfully',
    data: deletedTodo
  });
});

// ============================================================
// ERROR HANDLING: 404 for undefined routes
// ============================================================

app.use((req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found',
    path: req.path,
    method: req.method
  });
});

// ============================================================
// START SERVER
// ============================================================

const PORT = 5000;
app.listen(PORT, () => {
  console.log(`✅ Todo API running on http://localhost:${PORT}`);
  console.log(`📚 Available routes:`);
  console.log(`  GET    /api/todos        - Get all todos`);
  console.log(`  GET    /api/todos/:id    - Get single todo`);
  console.log(`  POST   /api/todos        - Create todo`);
  console.log(`  PUT    /api/todos/:id    - Update entire todo`);
  console.log(`  PATCH  /api/todos/:id    - Update partial todo`);
  console.log(`  DELETE /api/todos/:id    - Delete todo`);
});
```

---

## React Frontend Integration

Now let's create a React component to interact with the Todo API:

```javascript
// ============================================================
// FILE: src/components/TodoList.jsx
// PURPOSE: React component for Todo CRUD operations
// ============================================================

import React, { useState, useEffect } from 'react';
import axios from 'axios';

function TodoList() {
  // ============================================================
  // STATE: Manage todos, loading, errors, form
  // ============================================================
  
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  // Form state for creating new todo
  const [formData, setFormData] = useState({
    title: '',
    description: '',
    priority: 'medium'
  });
  
  // Filter state
  const [filter, setFilter] = useState('all'); // all, completed, pending
  
  // ============================================================
  // API BASE URL
  // ============================================================
  
  const API_URL = 'http://localhost:5000/api/todos';
  
  // ============================================================
  // FETCH TODOS: Get all todos when component mounts
  // ============================================================
  
  useEffect(() => {
    fetchTodos();
  }, []);
  
  const fetchTodos = async () => {
    try {
      setLoading(true);
      setError(null);
      
      // Build query based on filter
      let url = API_URL;
      if (filter === 'completed') {
        url += '?status=completed';
      } else if (filter === 'pending') {
        url += '?status=pending';
      }
      
      // FRONTEND: Call backend GET endpoint
      const response = await axios.get(url);
      
      console.log('Todos from backend:', response.data);
      
      // UPDATE STATE: Save todos to state
      setTodos(response.data.data);
      
    } catch (err) {
      console.error('Error fetching todos:', err);
      setError('Failed to load todos');
    } finally {
      setLoading(false);
    }
  };
  
  // ============================================================
  // CREATE TODO: Post new todo
  // ============================================================
  
  const handleCreateTodo = async (e) => {
    e.preventDefault();
    
    // Validate form
    if (!formData.title.trim()) {
      alert('Title is required');
      return;
    }
    
    try {
      // FRONTEND: Call backend POST endpoint
      const response = await axios.post(API_URL, {
        title: formData.title,
        description: formData.description,
        priority: formData.priority
      });
      
      console.log('New todo created:', response.data);
      
      // UPDATE STATE: Add new todo to list
      setTodos([response.data.data, ...todos]);
      
      // RESET FORM
      setFormData({ title: '', description: '', priority: 'medium' });
      
    } catch (err) {
      console.error('Error creating todo:', err);
      alert('Failed to create todo');
    }
  };
  
  // ============================================================
  // UPDATE TODO: Partially update todo
  // ============================================================
  
  const handleToggleTodo = async (id, currentCompleted) => {
    try {
      // FRONTEND: Call backend PATCH endpoint
      const response = await axios.patch(`${API_URL}/${id}`, {
        completed: !currentCompleted  // Toggle completed status
      });
      
      console.log('Todo updated:', response.data);
      
      // UPDATE STATE: Update todo in list
      setTodos(todos.map(todo =>
        todo.id === id ? response.data.data : todo
      ));
      
    } catch (err) {
      console.error('Error updating todo:', err);
      alert('Failed to update todo');
    }
  };
  
  // ============================================================
  // DELETE TODO: Remove todo
  // ============================================================
  
  const handleDeleteTodo = async (id) => {
    if (!window.confirm('Are you sure?')) return;
    
    try {
      // FRONTEND: Call backend DELETE endpoint
      const response = await axios.delete(`${API_URL}/${id}`);
      
      console.log('Todo deleted:', response.data);
      
      // UPDATE STATE: Remove from list
      setTodos(todos.filter(todo => todo.id !== id));
      
    } catch (err) {
      console.error('Error deleting todo:', err);
      alert('Failed to delete todo');
    }
  };
  
  // ============================================================
  // RENDER: Display UI
  // ============================================================
  
  if (loading) return <div>⏳ Loading todos...</div>;
  if (error) return <div>❌ {error}</div>;
  
  return (
    <div style={{ maxWidth: '600px', margin: '20px auto', fontFamily: 'Arial' }}>
      <h1>📋 My Todos</h1>
      
      {/* CREATE FORM */}
      <form onSubmit={handleCreateTodo} style={{ marginBottom: '20px' }}>
        <input
          type="text"
          placeholder="Todo title..."
          value={formData.title}
          onChange={(e) => setFormData({ ...formData, title: e.target.value })}
          style={{ padding: '8px', width: '100%', marginBottom: '10px' }}
        />
        <input
          type="text"
          placeholder="Description..."
          value={formData.description}
          onChange={(e) => setFormData({ ...formData, description: e.target.value })}
          style={{ padding: '8px', width: '100%', marginBottom: '10px' }}
        />
        <select
          value={formData.priority}
          onChange={(e) => setFormData({ ...formData, priority: e.target.value })}
          style={{ padding: '8px', marginRight: '10px' }}
        >
          <option value="low">Low Priority</option>
          <option value="medium">Medium Priority</option>
          <option value="high">High Priority</option>
        </select>
        <button type="submit" style={{ padding: '8px 20px' }}>
          ➕ Add Todo
        </button>
      </form>
      
      {/* FILTER BUTTONS */}
      <div style={{ marginBottom: '20px' }}>
        <button
          onClick={() => { setFilter('all'); fetchTodos(); }}
          style={{ marginRight: '10px', fontWeight: filter === 'all' ? 'bold' : 'normal' }}
        >
          All ({todos.length})
        </button>
        <button
          onClick={() => { setFilter('pending'); fetchTodos(); }}
          style={{ marginRight: '10px', fontWeight: filter === 'pending' ? 'bold' : 'normal' }}
        >
          Pending
        </button>
        <button
          onClick={() => { setFilter('completed'); fetchTodos(); }}
          style={{ fontWeight: filter === 'completed' ? 'bold' : 'normal' }}
        >
          Completed
        </button>
      </div>
      
      {/* TODOS LIST */}
      <div>
        {todos.length === 0 ? (
          <p>No todos yet. Create one above!</p>
        ) : (
          todos.map((todo) => (
            <div
              key={todo.id}
              style={{
                padding: '15px',
                marginBottom: '10px',
                border: '1px solid #ddd',
                borderRadius: '5px',
                opacity: todo.completed ? 0.6 : 1,
                textDecoration: todo.completed ? 'line-through' : 'none'
              }}
            >
              <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                <div style={{ flex: 1 }}>
                  <h3>{todo.title}</h3>
                  <p>{todo.description}</p>
                  <small>Priority: {todo.priority}</small>
                </div>
                <div style={{ display: 'flex', gap: '10px' }}>
                  <button
                    onClick={() => handleToggleTodo(todo.id, todo.completed)}
                    style={{ padding: '5px 10px' }}
                  >
                    {todo.completed ? '↩️ Undo' : '✅ Done'}
                  </button>
                  <button
                    onClick={() => handleDeleteTodo(todo.id)}
                    style={{ padding: '5px 10px', backgroundColor: '#ff6b6b', color: 'white' }}
                  >
                    🗑️ Delete
                  </button>
                </div>
              </div>
            </div>
          ))
        )}
      </div>
    </div>
  );
}

export default TodoList;
```

---

## Error Handling & Validation

### Backend Validation

```javascript
// ============================================================
// VALIDATION HELPER: Check if input is valid
// ============================================================

function validateTodoInput(title, description, priority) {
  const errors = [];
  
  // Title validation
  if (!title) {
    errors.push('Title is required');
  } else if (typeof title !== 'string') {
    errors.push('Title must be text');
  } else if (title.trim() === '') {
    errors.push('Title cannot be empty');
  } else if (title.length < 3) {
    errors.push('Title must be at least 3 characters');
  } else if (title.length > 200) {
    errors.push('Title cannot exceed 200 characters');
  }
  
  // Description validation
  if (description && typeof description !== 'string') {
    errors.push('Description must be text');
  }
  
  // Priority validation
  const validPriorities = ['low', 'medium', 'high'];
  if (priority && !validPriorities.includes(priority)) {
    errors.push('Priority must be low, medium, or high');
  }
  
  return errors;
}

// USE IN ROUTE:
app.post('/api/todos', (req, res) => {
  const { title, description, priority } = req.body;
  
  // Validate
  const errors = validateTodoInput(title, description, priority);
  
  if (errors.length > 0) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors: errors
    });
  }
  
  // Continue with creation...
});
```

### Try-Catch for Error Handling

```javascript
// ============================================================
// ASYNC ROUTES WITH TRY-CATCH
// ============================================================

app.get('/api/todos', async (req, res) => {
  try {
    // This might fail
    const todos = await fetchFromDatabase();
    
    res.json({
      success: true,
      data: todos
    });
    
  } catch (error) {
    // Something went wrong
    console.error('Error fetching todos:', error);
    
    res.status(500).json({
      success: false,
      message: 'Failed to fetch todos',
      error: error.message
    });
  }
});
```

---

## Best Practices

### 1. **Consistent Response Format**
```javascript
// ✅ GOOD: Always same structure
res.json({
  success: true/false,
  message: 'Human readable message',
  data: actualData,
  error: errorCode  // If error
});
```

### 2. **Validate Before Processing**
```javascript
// ✅ GOOD: Validate at start
if (!title) return res.status(400).json(...);
if (title.length < 3) return res.status(400).json(...);
// Then process
```

### 3. **Use Correct Status Codes**
```javascript
res.status(201).json(...);  // Create
res.status(200).json(...);  // Success (default)
res.status(400).json(...);  // Bad request
res.status(404).json(...);  // Not found
res.status(500).json(...);  // Server error
```

### 4. **Comment Your Routes**
```javascript
// ✅ GOOD: Clear comments
// GET /api/todos/:id
// Get single todo by ID
// Returns 200 with todo or 404 if not found
app.get('/api/todos/:id', (req, res) => { ... });
```

### 5. **Separate Validation Logic**
```javascript
// ✅ GOOD: Separate validation function
function validateTodo(data) {
  const errors = [];
  // Check data...
  return errors;
}

app.post('/api/todos', (req, res) => {
  const errors = validateTodo(req.body);
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  // Create...
});
```

---

## Project Task

### 🎯 Build a Complete Blog API

**Time:** 3-4 hours  
**Difficulty:** Beginner  

### Requirements

#### Backend
1. Create Express server with these routes:
   - `GET /api/posts` - Get all posts (with pagination/filtering)
   - `GET /api/posts/:id` - Get single post
   - `POST /api/posts` - Create new post
   - `PUT /api/posts/:id` - Update entire post
   - `PATCH /api/posts/:id` - Partially update post
   - `DELETE /api/posts/:id` - Delete post

2. Data Model for Post:
   ```javascript
   {
     id: string,
     title: string (required, 5-200 chars),
     content: string (required, 10+ chars),
     author: string (required),
     category: string (tech, life, travel, etc.),
     published: boolean,
     likes: number,
     createdAt: Date,
     updatedAt: Date
   }
   ```

3. Validation:
   - Validate all required fields
   - Check field types and lengths
   - Return proper error messages

4. Filtering/Querying:
   - Filter by category: `GET /api/posts?category=tech`
   - Filter by author: `GET /api/posts?author=john`
   - Pagination: `GET /api/posts?page=1&limit=10`

#### Frontend (React)
1. Create component to display all posts
2. Create form to add new post
3. Show single post details (fetch by ID)
4. Edit posts (PUT)
5. Delete posts
6. Filter posts by category

#### Tests
1. Test GET endpoints (should return posts)
2. Test POST (should create with validation)
3. Test invalid data (should return 400 errors)
4. Test non-existent post (should return 404)

### Checklist
- [ ] All routes working
- [ ] Validation working
- [ ] React component fetching data
- [ ] CRUD operations all work
- [ ] Error handling for all cases
- [ ] Code is commented
- [ ] No console errors

---

## Interview Questions

### Conceptual Q&A

**Q1: What's the difference between PUT and PATCH?**
<details>
<summary>Answer</summary>

- **PUT:** Replace ENTIRE resource. Provide all fields.
  ```javascript
  PUT /api/users/1
  { name: "John", email: "john@ex.com" }
  // ALL fields must be provided
  ```

- **PATCH:** Update PARTIAL resource. Only provide fields to change.
  ```javascript
  PATCH /api/users/1
  { name: "John" }
  // Only name updated, other fields stay same
  ```

**Interview Tip:** Mention you understand idempotence and the difference.
</details>

**Q2: What are route parameters vs query parameters?**
<details>
<summary>Answer</summary>

- **Route Parameters** (in path): `/api/posts/:id`
  - Part of URL structure
  - Required
  - `req.params.id`

- **Query Parameters** (after `?`): `/api/posts?category=tech&limit=10`
  - Optional filtering
  - `req.query.category`, `req.query.limit`
  - Good for filtering/sorting/pagination

Use route params for **identifying resources**, query params for **filtering**.
</details>

### Practical Q&A

**Q3: How would you handle this scenario: User submits form with missing required field?**
<details>
<summary>Answer</summary>

```javascript
app.post('/api/posts', (req, res) => {
  const { title, content, author } = req.body;
  
  // Validate required fields
  if (!title) {
    return res.status(400).json({
      success: false,
      message: 'Title is required',
      field: 'title'
    });
  }
  
  if (!content) {
    return res.status(400).json({
      success: false,
      message: 'Content is required',
      field: 'content'
    });
  }
  
  // Continue with creation...
});
```

**Frontend should also validate before sending**, but always validate on backend too!

**Interview Tip:** Show you understand server-side validation is critical.
</details>

**Q4: Write code to get all completed todos and limit to 5 results:**
<details>
<summary>Answer</summary>

```javascript
app.get('/api/todos', (req, res) => {
  let todos = [
    { id: 1, title: 'Task 1', completed: true },
    { id: 2, title: 'Task 2', completed: false },
    { id: 3, title: 'Task 3', completed: true },
    // ... more todos
  ];
  
  // Filter by status query param
  const { status, limit = 5 } = req.query;
  
  if (status === 'completed') {
    todos = todos.filter(t => t.completed === true);
  }
  
  // Apply limit
  todos = todos.slice(0, parseInt(limit));
  
  res.json({
    success: true,
    data: todos,
    total: todos.length
  });
});

// Usage: GET /api/todos?status=completed&limit=5
```

**Interview Tip:** Show you can work with query parameters and filtering.
</details>

---

## Key Takeaways

✅ **Routing** = Matching URLs to handler functions  
✅ **HTTP Methods** = GET (read), POST (create), PUT (replace), PATCH (update), DELETE (remove)  
✅ **Route Parameters** = Part of URL path (`/api/todos/:id`)  
✅ **Query Parameters** = After `?` (`/api/todos?status=done`)  
✅ **Request Body** = Data sent with POST/PUT/PATCH  
✅ **Status Codes** = Tell client what happened (200, 201, 400, 404, 500)  
✅ **Validation** = Always validate on backend  
✅ **Error Handling** = Always return proper error responses  

---

## 🎯 Practice Projects

Apply what you learned about routing and CRUD operations:

**Recommended Projects:**
- [Project #04: Todo List API](../projects/04-todo-list-api.md) - Full CRUD with validation ⭐
- [Project #05: Blog Post API](../projects/05-blog-post-api.md) - Posts with comments
- [Project #06: URL Shortener](../projects/06-url-shortener.md) - Hash algorithms
- [Project #07: Contact Form API](../projects/07-contact-form-api.md) - Email integration
- [Project #08: Note Taking App](../projects/08-note-taking-app.md) - Folders and tags
- [Project #09: Bookmark Manager](../projects/09-bookmark-manager.md) - Collections
- [Project #11: Random User Generator](../projects/11-random-user-generator.md) - Faker.js
- [Project #12: Polling System](../projects/12-polling-system.md) - Voting system
- [Project #13: Expense Tracker](../projects/13-expense-tracker.md) - Financial tracking

**Time Required:** 3-5 hours per project

---

## Next Chapter

🎉 Completed **B02: Express & Routing**

### What's Coming in B03:
- **MVC Architecture** - Proper code organization
- **Separation of Concerns** - Models, Views, Controllers
- **Project:** Refactor Todo API into MVC pattern

### Homework:
1. ✅ Complete the Blog API project
2. ✅ Build at least TWO practice projects from above
3. ✅ Get React fully working with backend
4. ✅ Practice all CRUD operations

---

| Previous | Index | Next |
|----------|-------|------|
| [B01: What is Backend?](B01_WHAT_IS_BACKEND.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B03: MVC Architecture](B03_MVC_ARCHITECTURE.md) |

---

**Author:** Tushar Parlikar  
**Last Updated:** October 20, 2025  
**Chapter Time:** 2-3 hours
