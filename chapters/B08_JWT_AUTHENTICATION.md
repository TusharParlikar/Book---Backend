# B08: JWT Authentication 🎫

> **Create Tokens for Stateless Authentication**

**Chapter Level:** Beginner  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B07  

---

## What is JWT?

**JWT** = JSON Web Token

A token proves user is authenticated without storing sessions on server.

```
USER LOGS IN
    ↓
SERVER creates JWT token
    ↓
CLIENT receives token
    ↓
CLIENT sends token with each request
    ↓
SERVER verifies token
    ↓
IF VALID: Allow access
IF INVALID: Reject request
```

---

## JWT Structure

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

[HEADER].[PAYLOAD].[SIGNATURE]

HEADER:    { alg: 'HS256', typ: 'JWT' }
PAYLOAD:   { sub: '123', name: 'John', iat: 1516239022 }
SIGNATURE: Proves header + payload weren't tampered with
```

---

## Implementation

```bash
npm install jsonwebtoken
```

```javascript
// ============================================================
// FILE: config/jwt.js
// ============================================================

const jwt = require('jsonwebtoken');

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';
const JWT_EXPIRE = '7d';

// CREATE TOKEN
const generateToken = (userId, email) => {
  return jwt.sign(
    { userId, email },  // Payload
    JWT_SECRET,         // Secret
    { expiresIn: JWT_EXPIRE }  // Options
  );
};

// VERIFY TOKEN
const verifyToken = (token) => {
  try {
    return jwt.verify(token, JWT_SECRET);
  } catch (error) {
    return null;
  }
};

module.exports = { generateToken, verifyToken };


// ============================================================
// FILE: controllers/authController.js (Updated)
// ============================================================

const User = require('../models/User');
const { generateToken } = require('../config/jwt');

const login = async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = await User.findOne({ email }).select('+password');
    if (!user || !(await user.comparePassword(password))) {
      return res.status(401).json({
        success: false,
        message: 'Invalid credentials'
      });
    }

    // GENERATE TOKEN
    const token = generateToken(user._id, user.email);

    // Send token to client
    res.json({
      success: true,
      message: 'Login successful',
      data: {
        id: user._id,
        name: user.name,
        email: user.email,
        token  // Client stores this
      }
    });

  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

module.exports = { login };


// ============================================================
// FILE: middleware/auth.js
// ============================================================

const { verifyToken } = require('../config/jwt');

const authMiddleware = (req, res, next) => {
  try {
    // GET TOKEN from headers
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({
        success: false,
        message: 'No token provided'
      });
    }

    // EXTRACT TOKEN (Remove "Bearer " prefix)
    const token = authHeader.substring(7);

    // VERIFY TOKEN
    const decoded = verifyToken(token);

    if (!decoded) {
      return res.status(401).json({
        success: false,
        message: 'Invalid or expired token'
      });
    }

    // ATTACH to request
    req.user = decoded;
    next();

  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

module.exports = authMiddleware;


// ============================================================
// FILE: routes/todoRoutes.js (Protected)
// ============================================================

const express = require('express');
const authMiddleware = require('../middleware/auth');
const todoController = require('../controllers/todoController');

const router = express.Router();

// PROTECTED: Only authenticated users can access
router.get('/', authMiddleware, todoController.getAllTodos);
router.post('/', authMiddleware, todoController.createTodo);
router.put('/:id', authMiddleware, todoController.updateTodo);
router.delete('/:id', authMiddleware, todoController.deleteTodo);

module.exports = router;
```

---

## React Frontend

```javascript
// ============================================================
// FILE: src/components/ProtectedTodos.jsx
// ============================================================

import React, { useState, useEffect } from 'react';
import axios from 'axios';

function ProtectedTodos() {
  const [todos, setTodos] = useState([]);
  const [token, setToken] = useState(localStorage.getItem('token'));

  // CREATE AXIOS INSTANCE with token
  const api = axios.create({
    baseURL: 'http://localhost:5000/api',
    headers: {
      Authorization: `Bearer ${token}`
    }
  });

  useEffect(() => {
    fetchTodos();
  }, [token]);

  const fetchTodos = async () => {
    try {
      const response = await api.get('/todos');
      setTodos(response.data.data);
    } catch (error) {
      if (error.response?.status === 401) {
        // Token invalid, redirect to login
        localStorage.removeItem('token');
        setToken(null);
      }
    }
  };

  const createTodo = async (title) => {
    try {
      const response = await api.post('/todos', { title });
      setTodos([...todos, response.data.data]);
    } catch (error) {
      alert('Failed to create todo');
    }
  };

  if (!token) {
    return <div>Please login first</div>;
  }

  return (
    <div>
      <h1>Protected Todos</h1>
      <ul>
        {todos.map(todo => (
          <li key={todo._id}>{todo.title}</li>
        ))}
      </ul>
    </div>
  );
}

export default ProtectedTodos;
```

---

## Interview Questions

**Q1: Why use JWT instead of sessions?**

- Stateless (no server storage)
- Scalable (works with multiple servers)
- Mobile-friendly
- Can be used across services

---

| Previous | Index | Next |
|----------|-------|------|
| [B07: Password Security](B07_PASSWORD_SECURITY.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B09: Complete Auth Flow](B09_COMPLETE_AUTH_FLOW.md) |
