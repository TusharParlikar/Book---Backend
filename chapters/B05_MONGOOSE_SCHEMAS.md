# B05: Mongoose Schemas & Validation 📋

> **Master Data Validation, Field Types, and Schema Design**

**Chapter Level:** Beginner  
**Time Required:** 2 hours  
**Prerequisites:** Completed B01-B04  

---

## Core Concepts

### Mongoose Schema

Schema defines the structure of MongoDB documents:

```javascript
const userSchema = new mongoose.Schema({
  // FIELD NAME: TYPE
  name: String,
  
  // WITH VALIDATION
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: /^[^\s@]+@[^\s@]+\.[^\s@]+$/  // Email regex
  },
  
  // WITH DEFAULTS
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  
  // ARRAYS
  tags: [String],
  
  // NESTED OBJECTS
  address: {
    street: String,
    city: String,
    country: String
  },
  
  // AUTO FIELDS
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);
```

### Field Types

```javascript
// Basic Types
String, Number, Boolean, Date, Array, Object

// Validation Examples
const schema = new mongoose.Schema({
  age: {
    type: Number,
    min: 0,
    max: 150
  },
  
  status: {
    type: String,
    enum: ['active', 'inactive', 'deleted']
  },
  
  phone: {
    type: String,
    match: /^\d{10}$/  // Exactly 10 digits
  },
  
  avatar: {
    type: String,
    required: true,
    minlength: 5
  }
});
```

### Custom Validators

```javascript
const userSchema = new mongoose.Schema({
  password: {
    type: String,
    validate: {
      validator: function(v) {
        // Password must be at least 6 characters
        return v && v.length >= 6;
      },
      message: 'Password must be at least 6 characters'
    }
  },
  
  confirmPassword: {
    type: String,
    validate: {
      validator: function(v) {
        // Must match password
        return v === this.password;
      },
      message: 'Passwords do not match'
    }
  }
});
```

---

## Complete Example

```javascript
// ============================================================
// FILE: models/User.js
// ============================================================

const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  // NAME
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [2, 'Name too short'],
    maxlength: [50, 'Name too long']
  },

  // EMAIL
  email: {
    type: String,
    required: [true, 'Email required'],
    unique: [true, 'Email already exists'],
    lowercase: true,
    trim: true,
    match: [
      /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      'Please provide a valid email'
    ]
  },

  // PHONE
  phone: {
    type: String,
    validate: {
      validator: function(v) {
        return /^\+?[1-9]\d{1,14}$/.test(v);
      },
      message: 'Invalid phone number'
    }
  },

  // PASSWORD (will be hashed)
  password: {
    type: String,
    required: [true, 'Password required'],
    minlength: [6, 'Password too short'],
    select: false  // Don't return in queries by default
  },

  // ROLE
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },

  // STATUS
  isActive: {
    type: Boolean,
    default: true
  },

  // NESTED OBJECT
  profile: {
    bio: String,
    avatar: String,
    website: String
  },

  // ARRAYS
  interests: [String],
  
  followers: {
    type: [mongoose.Schema.Types.ObjectId],
    ref: 'User'
  },

  // TIMESTAMPS
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
}, { timestamps: true });

// MIDDLEWARE: Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// METHOD: Compare passwords
userSchema.methods.comparePassword = async function(password) {
  return await bcrypt.compare(password, this.password);
};

// VIRTUAL: Full info (not stored in DB)
userSchema.virtual('fullInfo').get(function() {
  return `${this.name} (${this.email})`;
});

const User = mongoose.model('User', userSchema);

module.exports = User;
```

---

## Interview Questions

**Q1: What's the difference between required and default?**

- `required`: Field must be provided
- `default`: Field gets this value if not provided

**Q2: How do you validate email in Mongoose?**

```javascript
email: {
  type: String,
  match: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
}
```

---

| Previous | Index | Next |
|----------|-------|------|
| [B04: MongoDB Basics](B04_MONGODB_BASICS.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B06: Database Relationships](B06_DATABASE_RELATIONSHIPS.md) |
