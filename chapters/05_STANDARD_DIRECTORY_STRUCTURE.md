# Chapter 5: Standard Backend Directory Structure

## 5.1 Why Structure Matters

Professional projects aren't just random files in a folder. **Organization matters because:**

✅ **Other developers** can understand your code  
✅ **You can find things** quickly  
✅ **Code is reusable** and maintainable  
✅ **Easy to scale** as project grows  
✅ **Industry standard** - same across companies  

### Bad vs Good Structure

```
❌ BAD:
project/
├── app.js
├── users.js
├── posts.js
├── comments.js
├── database.js
├── auth.js
├── validation.js
└── ... (chaos!)

✅ GOOD:
project/
├── config/
├── routes/
├── controllers/
├── models/
├── middlewares/
├── utils/
└── public/
```

---

## 5.2 The `db/` Directory - Database Connection

### Purpose
This folder contains **database connection code** and setup.

### Contents

```
db/
├── index.js          # Main connection file
└── mongoose.js       # Mongoose configuration (optional)
```

### Example: `db/index.js`

```javascript
// db/index.js
// This file handles connecting to MongoDB

const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    // Connect to MongoDB using connection string from .env
    const conn = await mongoose.connect(process.env.MONGODB_URI);
    
    console.log(`✅ MongoDB Connected: ${conn.connection.host}`);
    
    return conn;
  } catch (error) {
    console.error(`❌ Database connection failed: ${error.message}`);
    // Exit process if database connection fails
    process.exit(1);
  }
};

module.exports = connectDB;
```

### How It's Used

```javascript
// server.js
const connectDB = require('./db');

const app = require('express')();

// Connect to database when server starts
connectDB();

app.listen(5000);
```

---

## 5.3 The `models/` Directory - Data Schemas

### Purpose
Defines the **structure and validation rules** for your data.

### Contents

```
models/
├── User.js          # User schema
├── Post.js          # Post schema
├── Comment.js       # Comment schema
└── index.js         # Export all models (optional)
```

### Example: `models/User.js`

```javascript
// models/User.js
// This file defines what a User looks like in the database

const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

// Define the structure
const userSchema = new mongoose.Schema(
  {
    // Username field
    username: {
      type: String,
      required: true,           // Must be provided
      unique: true,             // No duplicate usernames
      trim: true,               // Remove whitespace
      lowercase: true,          // Convert to lowercase
      minlength: [3, 'Username must be at least 3 characters'],
      maxlength: [30, 'Username cannot exceed 30 characters']
    },

    // Email field
    email: {
      type: String,
      required: true,
      unique: true,
      trim: true,
      lowercase: true,
      match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Invalid email format']
    },

    // Password field
    password: {
      type: String,
      required: true,
      minlength: [8, 'Password must be at least 8 characters'],
      select: false  // Don't return password by default in queries
    },

    // Profile information
    avatar: {
      type: String,
      default: null  // Optional field, default is null
    },

    // User role
    role: {
      type: String,
      enum: ['user', 'admin', 'moderator'],  // Only these values allowed
      default: 'user'
    },

    // Track when created
    createdAt: {
      type: Date,
      default: Date.now
    }
  },
  {
    timestamps: true  // Automatically add createdAt and updatedAt
  }
);

// Pre-save hook: hash password before saving
userSchema.pre('save', async function(next) {
  // Only hash if password is new or modified
  if (!this.isModified('password')) return next();

  try {
    // Generate salt and hash password
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Custom method: Compare password
userSchema.methods.comparePassword = async function(enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

// Create and export model
const User = mongoose.model('User', userSchema);

module.exports = User;
```

### Model File Conventions

Each model file should:
1. Define the schema with validation
2. Add any pre/post hooks
3. Add custom methods
4. Export the model

---

## 5.4 The `controllers/` Directory - Business Logic

### Purpose
Contains the **business logic** for handling requests.

### Contents

```
controllers/
├── userController.js      # Handle user operations
├── postController.js      # Handle post operations
├── authController.js      # Handle auth operations
└── index.js               # Export all controllers (optional)
```

### Example: `controllers/userController.js`

```javascript
// controllers/userController.js
// Contains all user-related business logic

const User = require('../models/User');

// GET: Retrieve all users
exports.getAllUsers = async (req, res) => {
  try {
    // Query database
    const users = await User.find().select('-password');

    // Send response
    res.status(200).json({
      success: true,
      count: users.length,
      data: users
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// GET: Retrieve single user by ID
exports.getUserById = async (req, res) => {
  try {
    // Extract ID from URL parameter
    const { id } = req.params;

    // Find user in database
    const user = await User.findById(id).select('-password');

    // Check if user exists
    if (!user) {
      return res.status(404).json({
        success: false,
        error: 'User not found'
      });
    }

    res.status(200).json({
      success: true,
      data: user
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// POST: Create new user
exports.createUser = async (req, res) => {
  try {
    // Extract data from request body
    const { username, email, password } = req.body;

    // Validate required fields
    if (!username || !email || !password) {
      return res.status(400).json({
        success: false,
        error: 'Missing required fields'
      });
    }

    // Check if user already exists
    const existingUser = await User.findOne({
      $or: [{ email }, { username }]
    });

    if (existingUser) {
      return res.status(409).json({
        success: false,
        error: 'User with this email or username already exists'
      });
    }

    // Create new user
    const newUser = await User.create({
      username,
      email,
      password
    });

    // Return success response (excluding password)
    res.status(201).json({
      success: true,
      message: 'User created successfully',
      data: {
        _id: newUser._id,
        username: newUser.username,
        email: newUser.email
      }
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// PUT: Update user
exports.updateUser = async (req, res) => {
  try {
    const { id } = req.params;
    const updateData = req.body;

    // Update user
    const updatedUser = await User.findByIdAndUpdate(id, updateData, {
      new: true,         // Return updated document
      runValidators: true // Run schema validators
    }).select('-password');

    if (!updatedUser) {
      return res.status(404).json({
        success: false,
        error: 'User not found'
      });
    }

    res.status(200).json({
      success: true,
      message: 'User updated successfully',
      data: updatedUser
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// DELETE: Remove user
exports.deleteUser = async (req, res) => {
  try {
    const { id } = req.params;

    // Find and delete user
    const deletedUser = await User.findByIdAndDelete(id);

    if (!deletedUser) {
      return res.status(404).json({
        success: false,
        error: 'User not found'
      });
    }

    res.status(200).json({
      success: true,
      message: 'User deleted successfully'
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};
```

---

## 5.5 The `routes/` Directory - API Endpoints

### Purpose
Defines the **URL endpoints** and connects them to controllers.

### Contents

```
routes/
├── userRoutes.js      # /api/users endpoints
├── postRoutes.js      # /api/posts endpoints
├── authRoutes.js      # /api/auth endpoints
└── index.js           # Combine all routes
```

### Example: `routes/userRoutes.js`

```javascript
// routes/userRoutes.js
// Defines all user-related endpoints

const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { authenticate } = require('../middlewares/auth');

// GET /api/users
// Retrieve all users
router.get('/', userController.getAllUsers);

// GET /api/users/:id
// Retrieve specific user by ID
router.get('/:id', userController.getUserById);

// POST /api/users
// Create new user
router.post('/', userController.createUser);

// PUT /api/users/:id
// Update user (requires authentication)
router.put('/:id', authenticate, userController.updateUser);

// DELETE /api/users/:id
// Delete user (requires authentication)
router.delete('/:id', authenticate, userController.deleteUser);

module.exports = router;
```

### Example: `routes/index.js` - Combining Routes

```javascript
// routes/index.js
// Combines all routes into one

const express = require('express');
const router = express.Router();

const userRoutes = require('./userRoutes');
const postRoutes = require('./postRoutes');
const authRoutes = require('./authRoutes');

// Mount routes at different paths
router.use('/users', userRoutes);      // /api/users
router.use('/posts', postRoutes);      // /api/posts
router.use('/auth', authRoutes);       // /api/auth

module.exports = router;
```

---

## 5.6 The `middlewares/` Directory - Request Processing

### Purpose
Contains **middleware functions** that process requests.

### Contents

```
middlewares/
├── auth.js            # Authentication middleware
├── errorHandler.js    # Error handling middleware
├── validation.js      # Input validation
└── logger.js          # Request logging
```

### Example: `middlewares/auth.js`

```javascript
// middlewares/auth.js
// Middleware to verify user is authenticated

const jwt = require('jsonwebtoken');

exports.authenticate = (req, res, next) => {
  try {
    // Get token from header
    const token = req.headers.authorization?.split(' ')[1];

    if (!token) {
      return res.status(401).json({
        error: 'No token provided'
      });
    }

    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Attach user to request
    req.user = decoded;

    // Continue to next middleware/route
    next();
  } catch (error) {
    res.status(401).json({
      error: 'Invalid token'
    });
  }
};
```

### Example: `middlewares/errorHandler.js`

```javascript
// middlewares/errorHandler.js
// Catches and handles errors

exports.errorHandler = (error, req, res, next) => {
  console.error('Error:', error);

  // Set default status and message
  let status = error.status || 500;
  let message = error.message || 'Internal Server Error';

  // Send error response
  res.status(status).json({
    success: false,
    error: message
  });
};
```

### Middleware Order in Main Server File

```javascript
// server.js

const app = require('express')();

// 1. Body parsing middleware (must be first)
app.use(express.json());

// 2. Logging middleware
app.use(require('./middlewares/logger'));

// 3. Authentication middleware (only on protected routes)
// Applied per-route, not globally

// 4. Routes
app.use('/api', require('./routes'));

// 5. Error handler (must be last)
app.use(require('./middlewares/errorHandler'));

app.listen(5000);
```

---

## 5.7 The `utils/` Directory - Reusable Functions

### Purpose
Contains **utility functions** used across controllers.

### Contents

```
utils/
├── validators.js      # Validation functions
├── formatters.js      # Format data functions
├── uploadFile.js      # File upload utility
└── apiResponse.js     # Standardized response format
```

### Example: `utils/apiResponse.js`

```javascript
// utils/apiResponse.js
// Standardized API response format

class ApiResponse {
  constructor(statusCode, data, message = 'Success') {
    this.statusCode = statusCode;
    this.data = data;
    this.message = message;
    this.success = statusCode < 400;
  }
}

module.exports = ApiResponse;

// Usage in controller:
const ApiResponse = require('../utils/apiResponse');

exports.getUsers = async (req, res) => {
  const users = await User.find();
  return res
    .status(200)
    .json(new ApiResponse(200, users, 'Users fetched successfully'));
};
```

---

## 5.8 The `public/` Directory - Static Files

### Purpose
Stores **static files** served directly to clients.

### Contents

```
public/
├── temp/              # Temporary file storage
├── uploads/           # Uploaded files
└── images/            # Static images
```

### Example Usage

```javascript
// server.js

const express = require('express');
const app = express();

// Serve static files from public directory
// Access at: http://localhost:5000/images/logo.png
app.use(express.static('public'));
```

---

## 5.9 Complete Project Structure

### Full Example Project

```
my-backend-project/
│
├── db/
│   └── index.js                 # MongoDB connection
│
├── models/
│   ├── User.js
│   ├── Post.js
│   └── Comment.js
│
├── controllers/
│   ├── userController.js
│   ├── postController.js
│   └── authController.js
│
├── routes/
│   ├── userRoutes.js
│   ├── postRoutes.js
│   ├── authRoutes.js
│   └── index.js
│
├── middlewares/
│   ├── auth.js
│   ├── errorHandler.js
│   └── validation.js
│
├── utils/
│   ├── apiResponse.js
│   ├── validators.js
│   └── asyncHandler.js
│
├── public/
│   ├── temp/
│   └── uploads/
│
├── .env                         # Environment variables
├── .gitignore                   # Git ignore rules
├── .env.example                 # Example env file
├── package.json                 # Dependencies
├── server.js                    # Main entry point
└── README.md                    # Project documentation
```

### Main Server File: `server.js`

```javascript
// server.js
// Entry point for the application

const express = require('express');
const dotenv = require('dotenv');
const connectDB = require('./db');

// Load environment variables
dotenv.config();

// Initialize Express app
const app = express();

// Connect to database
connectDB();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// Routes
app.use('/api', require('./routes'));

// Error handling middleware (must be last)
app.use(require('./middlewares/errorHandler'));

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`✅ Server running on port ${PORT}`);
});
```

---

## 5.10 Benefits of This Structure

✅ **Organized:** Easy to find files  
✅ **Scalable:** Can add features without chaos  
✅ **Maintainable:** Other developers understand it  
✅ **Professional:** Follows industry standards  
✅ **Testable:** Each component is isolated  
✅ **Reusable:** Utilities shared across app  

---

## 5.11 Key Takeaways

✅ **`db/`** - Database connection  
✅ **`models/`** - Data schemas and validation  
✅ **`controllers/`** - Business logic  
✅ **`routes/`** - API endpoints  
✅ **`middlewares/`** - Request processing  
✅ **`utils/`** - Reusable functions  
✅ **`public/`** - Static files  
✅ **`server.js`** - Application entry point  

---

## 🎯 Next Steps

Now you understand the professional structure. Next: **Chapter 6: JavaScript Backend Scenarios** - common real-world situations you'll handle!
