# B09: Complete Auth Flow 🔑

> **Full Authentication System: Register, Login, Protected Routes**

**Chapter Level:** Beginner  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B08  

---

## Complete System

```javascript
// ============================================================
// FILE: models/User.js
// ============================================================

const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true, select: false },
  role: { type: String, enum: ['user', 'admin'], default: 'user' }
}, { timestamps: true });

userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

userSchema.methods.comparePassword = async function(pwd) {
  return await bcrypt.compare(pwd, this.password);
};

module.exports = mongoose.model('User', userSchema);


// ============================================================
// FILE: controllers/authController.js
// ============================================================

const User = require('../models/User');
const { generateToken } = require('../config/jwt');

const register = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    if (await User.findOne({ email })) {
      return res.status(409).json({ success: false, message: 'Email exists' });
    }

    const user = await User.create({ name, email, password });

    res.status(201).json({
      success: true,
      data: { id: user._id, name: user.name, email: user.email }
    });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

const login = async (req, res) => {
  try {
    const { email, password } = req.body;

    const user = await User.findOne({ email }).select('+password');
    if (!user || !(await user.comparePassword(password))) {
      return res.status(401).json({ success: false, message: 'Invalid credentials' });
    }

    const token = generateToken(user._id, user.email);

    res.json({
      success: true,
      data: {
        id: user._id,
        name: user.name,
        email: user.email,
        token
      }
    });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const getCurrentUser = async (req, res) => {
  try {
    const user = await User.findById(req.user.userId);
    res.json({ success: true, data: user });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const logout = (req, res) => {
  // Frontend removes token from localStorage
  res.json({ success: true, message: 'Logged out' });
};

module.exports = { register, login, getCurrentUser, logout };


// ============================================================
// FILE: routes/authRoutes.js
// ============================================================

const express = require('express');
const authController = require('../controllers/authController');
const authMiddleware = require('../middleware/auth');

const router = express.Router();

router.post('/register', authController.register);
router.post('/login', authController.login);
router.get('/me', authMiddleware, authController.getCurrentUser);
router.post('/logout', authMiddleware, authController.logout);

module.exports = router;


// ============================================================
// FILE: server.js
// ============================================================

const express = require('express');
const mongoose = require('mongoose');
const authRoutes = require('./routes/authRoutes');
const todoRoutes = require('./routes/todoRoutes');

const app = express();

app.use(express.json());
app.use(cors());

const connectDB = async () => {
  try {
    await mongoose.connect('mongodb://localhost:27017/app');
    console.log('✅ MongoDB connected');
  } catch (error) {
    console.error('❌ MongoDB connection error:', error.message);
    process.exit(1);
  }
};

connectDB();

app.use('/api/auth', authRoutes);
app.use('/api/todos', todoRoutes);

app.listen(5000, () => console.log('✅ Server on 5000'));
```

<details>
<summary>💡 Expected Server Output</summary>

```bash
✅ MongoDB connected
✅ Server on 5000
```
</details>

---

## React Complete Auth

```javascript
// ============================================================
// FILE: src/App.jsx (Auth Context)
// ============================================================

import React, { createContext, useState, useEffect } from 'react';
import axios from 'axios';

export const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(localStorage.getItem('token'));
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const verifyToken = async () => {
      if (token) {
        try {
          const res = await axios.get('http://localhost:5000/api/auth/me', {
            headers: { Authorization: `Bearer ${token}` }
          });
          setUser(res.data.data);
        } catch (error) {
          setToken(null);
          localStorage.removeItem('token');
        }
      }
      setLoading(false);
    };

    verifyToken();
  }, [token]);

  const login = async (email, password) => {
    try {
      const res = await axios.post('http://localhost:5000/api/auth/login', {
        email,
        password
      });
      const { token, ...userData } = res.data.data;
      setToken(token);
      setUser(userData);
      localStorage.setItem('token', token);
      return res.data;
    } catch (error) {
      console.error('Login failed:', error);
      throw error; // Re-throw to be caught by the component
    }
  };

  const register = async (name, email, password) => {
    try {
      const res = await axios.post('http://localhost:5000/api/auth/register', {
        name,
        email,
        password
      });
      return res.data;
    } catch (error) {
      console.error('Registration failed:', error);
      throw error; // Re-throw to be caught by the component
    }
  };

  const logout = () => {
    setUser(null);
    setToken(null);
    localStorage.removeItem('token');
  };

  return (
    <AuthContext.Provider value={{ user, token, loading, login, register, logout }}>
      {children}
    </Auth.Provider>
  );
}


// ============================================================
// FILE: src/components/ProtectedRoute.jsx
// ============================================================

import React, { useContext } from 'react';
import { AuthContext } from '../App';

function ProtectedRoute({ children }) {
  const { token, loading } = useContext(AuthContext);

  if (loading) return <div>Loading...</div>;
  if (!token) return <div>Please login</div>;

  return children;
}

export default ProtectedRoute;
```

---

## Interview Questions

**Q1: What's the complete auth flow?**

1. User enters credentials
2. Backend validates
3. Backend creates JWT token
4. Client stores token
5. Client sends token with each request
6. Backend verifies token
7. If valid, allow access

---

| Previous | Index | Next |
|----------|-------|------|
| [B08: JWT Authentication](B08_JWT_AUTHENTICATION.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B10: E-Commerce Backend](B10_ECOMMERCE_BACKEND.md) |
