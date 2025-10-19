# Chapter 8: Full-Stack Integration with React

## 8.1 Backend and Frontend Communication

The complete data flow when React interacts with Express:

```
React Component
    ↓ (Axios/Fetch sends HTTP request)
Express Server
    ↓ (Routes request to controller)
Controller (Business Logic)
    ↓ (Queries database)
MongoDB
    ↓ (Returns data)
Controller (Processes response)
    ↓ (Sends JSON)
React Component
    ↓ (Updates state)
UI Re-renders
```

## 8.2 Understanding CORS (Cross-Origin Resource Sharing)

### What is CORS?

CORS is a browser security feature preventing websites from requesting data from other domains without permission.

```
Example: Your React app at http://localhost:3000
          Your Backend at http://localhost:5000
          
Browser blocks this by default! Need CORS.
```

### Install CORS Package

```bash
npm install cors
```

### Enable CORS in Express

```javascript
// server.js

const cors = require('cors');
const express = require('express');
const app = express();

// Enable CORS for all routes
app.use(cors());

// Or configure specifically
app.use(cors({
  origin: 'http://localhost:3000',  // Allow only React frontend
  credentials: true                  // Allow cookies
}));
```

### Axios with CORS

```jsx
// React component

import axios from 'axios';

function UserList() {
  const getUsers = async () => {
    try {
      // CORS automatically handled by axios
      const response = await axios.get('http://localhost:5000/api/users');
      console.log(response.data);
    } catch (error) {
      console.error(error);
    }
  };

  return <button onClick={getUsers}>Get Users</button>;
}
```

### Fetch with CORS

```jsx
// React component using Fetch API

function UserList() {
  const getUsers = async () => {
    try {
      const response = await fetch('http://localhost:5000/api/users', {
        method: 'GET',
        headers: { 'Content-Type': 'application/json' },
        credentials: 'include'  // Include cookies in request
      });
      const data = await response.json();
      console.log(data);
    } catch (error) {
      console.error(error);
    }
  };

  return <button onClick={getUsers}>Get Users</button>;
}
```

## 8.3 Creating Backend API Endpoints

### REST API Principles

```
GET    /api/users           - Get all users
GET    /api/users/:id       - Get single user
POST   /api/users           - Create user
PUT    /api/users/:id       - Update user
DELETE /api/users/:id       - Delete user
```

### Express Routes

```javascript
// routes/userRoutes.js

const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

// Create all REST endpoints
router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

module.exports = router;
```

## 8.4 Building React Frontend with Vite

### Create Vite React App

```bash
# Create new Vite + React project
npm create vite@latest my-frontend -- --template react

# Navigate to project
cd my-frontend

# Install dependencies
npm install

# Start dev server (runs on http://localhost:5173)
npm run dev
```

### React Component Calling Backend

```jsx
// src/components/Users.jsx

import { useState, useEffect } from 'react';
import axios from 'axios';

export default function Users() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Fetch users from backend
    axios
      .get('http://localhost:5000/api/users')
      .then((res) => {
        setUsers(res.data.users);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users.map((user) => (
          <li key={user._id}>{user.name} - {user.email}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 8.5 Solving CORS Issues

### Common CORS Errors

```
❌ Access to XMLHttpRequest at 'http://localhost:5000/api/users' 
   from origin 'http://localhost:3000' has been blocked by CORS policy
```

### Solutions

**Option 1: Server-side (Recommended)**
```javascript
// Add to server.js
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000'
}));
```

**Option 2: Proxy in Vite**
```javascript
// vite.config.js

export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true
      }
    }
  }
}
```

## 8.6 HTTP Status Codes

### Common Status Codes

```
2xx Success:
200 OK               - Request successful
201 Created          - Resource created
204 No Content       - Success, no response body

3xx Redirection:
301 Moved Permanently
302 Found

4xx Client Error:
400 Bad Request      - Invalid input
401 Unauthorized     - Need authentication
403 Forbidden        - No permission
404 Not Found        - Resource doesn't exist

5xx Server Error:
500 Internal Server Error
503 Service Unavailable
```

### Using Status Codes

```javascript
// Backend responses

// Success
res.status(200).json({ message: 'OK' });
res.status(201).json({ message: 'Created' });

// Client error
res.status(400).json({ error: 'Invalid input' });
res.status(404).json({ error: 'Not found' });

// Server error
res.status(500).json({ error: 'Server error' });
```

## 8.7 Axios vs Fetch Comparison

### Axios
```javascript
// More concise
axios.get('/api/users')
  .then(res => console.log(res.data))
  .catch(err => console.log(err));

// Automatic JSON conversion
// Request interceptors
// Better error handling
```

### Fetch
```javascript
// More verbose
fetch('/api/users')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.log(err));

// Built-in (no install)
// More control
// Native to browsers
```

**Recommendation:** Use Axios for React projects (simpler syntax)

## 8.8 Key Takeaways

✅ **CORS** - Handle cross-origin requests  
✅ **REST API** - Organize endpoints logically  
✅ **HTTP Methods** - GET, POST, PUT, DELETE  
✅ **Status Codes** - Communicate result  
✅ **Vite** - Modern React development  
✅ **Axios** - Simplified HTTP client  
✅ **Frontend and backend connected!**

---

## 🎯 Next Steps

Frontend connected to backend! Next: **Chapter 9: Data Modeling Fundamentals**

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 7: Project Setup & Deployment](./07_PROJECT_SETUP_DEPLOYMENT.md) | [📖 Course Index](../INDEX.md) | [Chapter 9: Data Modeling Fundamentals →](./09_DATA_MODELING_FUNDAMENTALS.md) |
