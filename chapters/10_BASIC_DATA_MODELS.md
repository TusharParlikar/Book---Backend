# Chapter 10: Basic Data Models with Mongoose

## 10.1 Schema Field Types

### String Type

```javascript
const userSchema = new mongoose.Schema({
  name: String,
  email: {
    type: String,
    required: true,
    unique: true
  }
});
```

### Number Type

```javascript
const productSchema = new mongoose.Schema({
  price: {
    type: Number,
    required: true,
    min: 0
  },
  quantity: Number
});
```

### Boolean Type

```javascript
const userSchema = new mongoose.Schema({
  isActive: {
    type: Boolean,
    default: true
  }
});
```

### Date Type

```javascript
const postSchema = new mongoose.Schema({
  createdAt: {
    type: Date,
    default: Date.now
  },
  publishedAt: Date
});
```

### Array Type

```javascript
const userSchema = new mongoose.Schema({
  hobbies: [String],
  scores: [Number],
  tags: {
    type: [String],
    default: []
  }
});
```

### Object/Mixed Type

```javascript
const documentSchema = new mongoose.Schema({
  metadata: {
    type: mongoose.Schema.Types.Mixed,
    default: {}
  }
});
```

## 10.2 Validation Options

```javascript
const userSchema = new mongoose.Schema({
  // String validations
  username: {
    type: String,
    required: [true, 'Username is required'],
    minlength: [3, 'Min 3 characters'],
    maxlength: [30, 'Max 30 characters'],
    trim: true,
    lowercase: true,
    match: [/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, underscore']
  },

  // Number validations
  age: {
    type: Number,
    min: [0, 'Must be positive'],
    max: [150, 'Invalid age']
  },

  // Email validation
  email: {
    type: String,
    required: true,
    unique: true,
    match: [/^\S+@\S+\.\S+$/, 'Invalid email']
  }
});
```

## 10.3 Automatic Timestamps

```javascript
const postSchema = new mongoose.Schema(
  {
    title: String,
    content: String
  },
  {
    timestamps: true  // Adds createdAt and updatedAt automatically
  }
);

// Document will have:
// { title: '...', createdAt: Date, updatedAt: Date }
```

## 10.4 Model Relationships

### Reference (ObjectId)

```javascript
// User has many Posts
const postSchema = new mongoose.Schema({
  title: String,
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  }
});

// Query with population
const post = await Post.findById(id).populate('userId');
// Returns post with full user object
```

## 10.5 Arrays and Nested Schemas

```javascript
const userSchema = new mongoose.Schema({
  name: String,
  // Nested object
  address: {
    street: String,
    city: String,
    country: String
  },
  // Array of nested objects
  phoneNumbers: [{
    type: String,
    label: String
  }]
});
```

## 10.6 Practical Models

### User Model

```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true
  },
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: String,
  avatar: String,
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  }
}, { timestamps: true });

const User = mongoose.model('User', userSchema);
module.exports = User;
```

### Post Model

```javascript
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
}, { timestamps: true });

const Post = mongoose.model('Post', postSchema);
module.exports = Post;
```

---

## 🎯 Next: Chapter 11 - Advanced Data Modeling

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 9: Data Modeling Fundamentals](./09_DATA_MODELING_FUNDAMENTALS.md) | [📖 Course Index](../INDEX.md) | [Chapter 11: Advanced E-commerce Models →](./11_ADVANCED_ECOMMERCE_MODELS.md) |
