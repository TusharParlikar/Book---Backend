# B07: Password Security with bcrypt 🔐

> **Hash Passwords Securely: Never Store Plain Text**

**Chapter Level:** Beginner  
**Time Required:** 2 hours  
**Prerequisites:** Completed B01-B06  

---

## Why Password Security?

```
❌ DANGEROUS: Storing plain text passwords
- If DB hacked, attackers get all passwords
- Can access other services (people reuse passwords)
- Legal liability, user trust lost

✅ SAFE: Hashing passwords
- Even if DB hacked, passwords are protected
- Each password unique (because of salt)
- Cannot reverse (one-way function)
```

---

## How Hashing Works

```
PLAIN TEXT PASSWORD: "MyPassword123"
              ↓
        HASHING ALGORITHM (bcrypt)
              ↓
HASHED PASSWORD: "$2b$10$abcdefghijklmnopqrstuvwxyz..."
```

Key points:
- **One-way:** Cannot reverse hash to get password
- **Unique salt:** Same password = different hash (because of salt)
- **Slow:** Takes time to hash (prevents brute force)

---

## Setup

```bash
npm install bcrypt
```

---

## Complete Implementation

```javascript
// ============================================================
// FILE: models/User.js
// ============================================================

const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true,
    minlength: 6,
    select: false  // Don't return password in queries
  }
});

// MIDDLEWARE: Hash password before saving
userSchema.pre('save', async function(next) {
  // Only hash if password is modified
  if (!this.isModified('password')) {
    return next();
  }

  try {
    // STEP 1: Generate salt
    // Salt rounds: higher = more secure but slower
    const salt = await bcrypt.genSalt(10);

    // STEP 2: Hash password with salt
    this.password = await bcrypt.hash(this.password, salt);

    next();
  } catch (error) {
    next(error);
  }
});

// METHOD: Compare passwords (for login)
userSchema.methods.comparePassword = async function(passwordToCheck) {
  try {
    // bcrypt.compare returns true/false
    return await bcrypt.compare(passwordToCheck, this.password);
  } catch (error) {
    throw error;
  }
};

module.exports = mongoose.model('User', userSchema);


// ============================================================
// FILE: controllers/authController.js
// ============================================================

const User = require('../models/User');

// REGISTER: Create new user with hashed password
const register = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // VALIDATION
    if (!name || !email || !password) {
      return res.status(400).json({
        success: false,
        message: 'All fields required'
      });
    }

    // CHECK: Email already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: 'Email already registered'
      });
    }

    // CREATE: New user
    // Password will be hashed by pre-save middleware
    const user = await User.create({
      name,
      email,
      password
    });

    // RESPONSE: Don't send password back
    res.status(201).json({
      success: true,
      message: 'User registered successfully',
      data: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });

  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// LOGIN: Verify password
const login = async (req, res) => {
  try {
    const { email, password } = req.body;

    // VALIDATION
    if (!email || !password) {
      return res.status(400).json({
        success: false,
        message: 'Email and password required'
      });
    }

    // FIND: User by email
    // .select('+password') because password has select: false
    const user = await User.findOne({ email }).select('+password');

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password'
      });
    }

    // COMPARE: Check password
    const isPasswordValid = await user.comparePassword(password);

    if (!isPasswordValid) {
      return res.status(401).json({
        success: false,
        message: 'Invalid email or password'
      });
    }

    // SUCCESS: Password matches
    res.json({
      success: true,
      message: 'Login successful',
      data: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });

  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

module.exports = { register, login };


// ============================================================
// FILE: routes/authRoutes.js
// ============================================================

const express = require('express');
const { register, login } = require('../controllers/authController');

const router = express.Router();

router.post('/register', register);
router.post('/login', login);

module.exports = router;
```

---

## React Frontend

```javascript
// ============================================================
// FILE: src/components/Auth.jsx
// ============================================================

import React, { useState } from 'react';
import axios from 'axios';

function Auth() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    confirmPassword: ''
  });

  const [isRegistering, setIsRegistering] = useState(false);
  const [message, setMessage] = useState('');

  // REGISTER
  const handleRegister = async (e) => {
    e.preventDefault();

    if (formData.password !== formData.confirmPassword) {
      setMessage('Passwords do not match');
      return;
    }

    try {
      const response = await axios.post(
        'http://localhost:5000/api/auth/register',
        {
          name: formData.name,
          email: formData.email,
          password: formData.password
        }
      );

      setMessage('✅ Registration successful! Login now.');
      setFormData({ name: '', email: '', password: '', confirmPassword: '' });
    } catch (error) {
      setMessage('❌ ' + (error.response?.data?.message || 'Registration failed'));
    }
  };

  // LOGIN
  const handleLogin = async (e) => {
    e.preventDefault();

    try {
      const response = await axios.post(
        'http://localhost:5000/api/auth/login',
        {
          email: formData.email,
          password: formData.password
        }
      );

      setMessage('✅ Login successful!');
      // Save user info
      localStorage.setItem('user', JSON.stringify(response.data.data));
    } catch (error) {
      setMessage('❌ ' + (error.response?.data?.message || 'Login failed'));
    }
  };

  return (
    <div style={{ maxWidth: '400px', margin: '50px auto' }}>
      <h1>{isRegistering ? 'Register' : 'Login'}</h1>

      <form onSubmit={isRegistering ? handleRegister : handleLogin}>
        {isRegistering && (
          <input
            type="text"
            placeholder="Full name"
            value={formData.name}
            onChange={(e) => setFormData({ ...formData, name: e.target.value })}
            style={{ width: '100%', padding: '10px', marginBottom: '10px' }}
          />
        )}

        <input
          type="email"
          placeholder="Email"
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
          style={{ width: '100%', padding: '10px', marginBottom: '10px' }}
        />

        <input
          type="password"
          placeholder="Password"
          value={formData.password}
          onChange={(e) => setFormData({ ...formData, password: e.target.value })}
          style={{ width: '100%', padding: '10px', marginBottom: '10px' }}
        />

        {isRegistering && (
          <input
            type="password"
            placeholder="Confirm password"
            value={formData.confirmPassword}
            onChange={(e) => setFormData({ ...formData, confirmPassword: e.target.value })}
            style={{ width: '100%', padding: '10px', marginBottom: '10px' }}
          />
        )}

        <button type="submit" style={{ width: '100%', padding: '10px' }}>
          {isRegistering ? 'Register' : 'Login'}
        </button>
      </form>

      <button
        onClick={() => setIsRegistering(!isRegistering)}
        style={{ marginTop: '20px', width: '100%' }}
      >
        {isRegistering ? 'Already have account? Login' : 'New user? Register'}
      </button>

      {message && <p style={{ color: 'red', marginTop: '10px' }}>{message}</p>}
    </div>
  );
}

export default Auth;
```

---

## Best Practices

✅ **Hash on the backend** - Never hash on frontend  
✅ **Use bcrypt** - Industry standard  
✅ **Never log passwords** - Don't log passwords for debugging  
✅ **Don't return passwords** - Use `select: false` in schema  
✅ **HTTPS only** - Transport passwords encrypted  
✅ **Minimum length** - At least 6-8 characters  

---

## Interview Questions

**Q1: Why is bcrypt better than MD5?**

- MD5 is fast (easy to brute force)
- bcrypt is slow (hard to brute force)
- bcrypt uses salt (unique for each password)

**Q2: What happens if user forgets password?**

Send reset email with token, not plain password.

---

| Previous | Index | Next |
|----------|-------|------|
| [B06: Database Relationships](B06_DATABASE_RELATIONSHIPS.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B08: JWT Authentication](B08_JWT_AUTHENTICATION.md) |
