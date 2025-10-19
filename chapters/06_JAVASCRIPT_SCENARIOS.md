# Chapter 6: JavaScript Backend Scenarios

## 6.1 Real-World Backend Challenges

In previous chapters, we learned concepts. Now let's explore **practical scenarios** you'll encounter as a backend developer using Node.js.

---

## 6.2 Scenario 1: Handling Direct Data (Synchronous Processing)

### The Situation
User submits a form, backend processes data immediately and returns response.

### Example: Simple Calculations

```javascript
// controllers/calculatorController.js

exports.calculateTotal = (req, res) => {
  try {
    // Extract data from request
    const { items } = req.body;
    // items = [{ price: 10, quantity: 2 }, { price: 5, quantity: 3 }]

    // Process data (synchronous - happens instantly)
    let total = 0;
    items.forEach(item => {
      total += item.price * item.quantity;
    });

    // Add tax
    const tax = total * 0.1;
    const finalTotal = total + tax;

    // Send response immediately
    res.json({
      subtotal: total,
      tax: tax,
      total: finalTotal
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### React Component Example

```jsx
// components/Calculator.jsx

import { useState } from 'react';
import axios from 'axios';

export default function Calculator() {
  const [items, setItems] = useState([]);
  const [result, setResult] = useState(null);

  const handleCalculate = async () => {
    try {
      // Send items to backend
      const response = await axios.post(
        'http://localhost:5000/api/calculate',
        { items }
      );

      // Backend processes and returns result immediately
      setResult(response.data);
    } catch (error) {
      console.error('Error:', error);
    }
  };

  return (
    <div>
      <h2>Price Calculator</h2>
      {/* Add items form here */}
      <button onClick={handleCalculate}>Calculate Total</button>
      
      {result && (
        <div>
          <p>Subtotal: ${result.subtotal.toFixed(2)}</p>
          <p>Tax: ${result.tax.toFixed(2)}</p>
          <p><strong>Total: ${result.total.toFixed(2)}</strong></p>
        </div>
      )}
    </div>
  );
}
```

---

## 6.3 Scenario 2: File Handling and Processing

### The Situation
User uploads a file, backend saves it, processes it, maybe stores in cloud.

### Example: Avatar Upload

```javascript
// middlewares/multer.js
// Configure file upload

const multer = require('multer');
const path = require('path');

// Configure storage
const storage = multer.diskStorage({
  // Where to save uploaded files
  destination: (req, file, cb) => {
    cb(null, 'public/temp');  // Save to temp folder
  },
  // What to name the file
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${file.originalname}`;
    cb(null, uniqueName);
  }
});

// Filter file types
const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);  // Accept file
  } else {
    cb(new Error('Invalid file type'), false);  // Reject file
  }
};

// Create upload middleware
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB max
});

module.exports = upload;
```

```javascript
// controllers/userController.js
// Handle file upload

const fs = require('fs').promises;
const cloudinary = require('cloudinary').v2;
const User = require('../models/User');

exports.uploadAvatar = async (req, res) => {
  try {
    // 1. Check if file was uploaded
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }

    // 2. Get current user
    const user = await User.findById(req.user._id);

    // 3. Upload to Cloudinary (cloud storage)
    const result = await cloudinary.uploader.upload(req.file.path, {
      folder: 'avatars',
      resource_type: 'auto'
    });

    // 4. Update user avatar URL
    user.avatar = result.secure_url;
    await user.save();

    // 5. Delete temporary file
    await fs.unlink(req.file.path);

    // 6. Send response
    res.json({
      success: true,
      message: 'Avatar uploaded successfully',
      avatar: user.avatar
    });
  } catch (error) {
    // Clean up temp file if error occurred
    if (req.file?.path) {
      await fs.unlink(req.file.path).catch(err => console.error(err));
    }

    res.status(500).json({ error: error.message });
  }
};
```

```javascript
// routes/userRoutes.js
// Add upload route

const express = require('express');
const router = express.Router();
const upload = require('../middlewares/multer');
const { authenticate } = require('../middlewares/auth');
const userController = require('../controllers/userController');

// POST /api/users/avatar
// Upload avatar (requires login, expects file in 'avatar' field)
router.post(
  '/avatar',
  authenticate,
  upload.single('avatar'),  // Middleware: handle single file upload
  userController.uploadAvatar
);

module.exports = router;
```

### React Component for File Upload

```jsx
// components/AvatarUpload.jsx

import { useState } from 'react';
import axios from 'axios';

export default function AvatarUpload() {
  const [selectedFile, setSelectedFile] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [avatar, setAvatar] = useState(null);

  const handleFileSelect = (e) => {
    // Get selected file
    const file = e.target.files[0];
    
    // Validate file type
    if (!file.type.startsWith('image/')) {
      alert('Please select an image');
      return;
    }
    
    // Validate file size
    if (file.size > 5 * 1024 * 1024) {
      alert('File too large (max 5MB)');
      return;
    }
    
    setSelectedFile(file);
  };

  const handleUpload = async () => {
    if (!selectedFile) {
      alert('Please select a file first');
      return;
    }

    setUploading(true);

    try {
      // Create FormData (required for file upload)
      const formData = new FormData();
      formData.append('avatar', selectedFile);

      // Get auth token from localStorage
      const token = localStorage.getItem('authToken');

      // Upload file to backend
      const response = await axios.post(
        'http://localhost:5000/api/users/avatar',
        formData,
        {
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'multipart/form-data'
          }
        }
      );

      // Update avatar display
      setAvatar(response.data.avatar);
      setSelectedFile(null);
    } catch (error) {
      console.error('Upload failed:', error);
      alert('Upload failed');
    } finally {
      setUploading(false);
    }
  };

  return (
    <div>
      <h3>Upload Avatar</h3>
      
      {avatar && (
        <img src={avatar} alt="Avatar" style={{ width: '100px', borderRadius: '50%' }} />
      )}

      <input
        type="file"
        accept="image/*"
        onChange={handleFileSelect}
        disabled={uploading}
      />

      <button onClick={handleUpload} disabled={uploading || !selectedFile}>
        {uploading ? 'Uploading...' : 'Upload'}
      </button>
    </div>
  );
}
```

---

## 6.4 Scenario 3: Third-Party API Integration

### The Situation
Your backend needs to call external APIs (payment processing, weather, social media, etc.).

### Example: Weather API Integration

```javascript
// controllers/weatherController.js

const axios = require('axios');

exports.getWeather = async (req, res) => {
  try {
    // Extract city from request
    const { city } = req.query;

    if (!city) {
      return res.status(400).json({ error: 'City is required' });
    }

    // Call external weather API
    const response = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather`,
      {
        params: {
          q: city,
          appid: process.env.OPENWEATHER_API_KEY,
          units: 'metric'
        }
      }
    );

    // Extract relevant data
    const weather = {
      city: response.data.name,
      temperature: response.data.main.temp,
      description: response.data.weather[0].description,
      humidity: response.data.main.humidity,
      windSpeed: response.data.wind.speed
    };

    res.json(weather);
  } catch (error) {
    if (error.response?.status === 404) {
      return res.status(404).json({ error: 'City not found' });
    }

    res.status(500).json({ error: 'Failed to fetch weather data' });
  }
};
```

### Why Backend Calls APIs?

```
WRONG: React directly calls external API
┌─────────────┐
│   React     │ ← API Key exposed in browser code!
│   Browser   │
└─────────────┘
        ↓
External API (security risk!)

RIGHT: React calls backend, backend calls external API
┌─────────────┐
│   React     │ (no API key)
│   Browser   │
└─────────────┘
        ↓
┌─────────────┐
│   Backend   │ ← API Key safely hidden
│   Server    │
└─────────────┘
        ↓
External API (secure!)
```

### React Component

```jsx
// components/Weather.jsx

import { useState } from 'react';
import axios from 'axios';

export default function Weather() {
  const [city, setCity] = useState('');
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const handleFetch = async () => {
    setLoading(true);
    setError(null);

    try {
      // Call backend (backend calls external API)
      const response = await axios.get(
        `http://localhost:5000/api/weather?city=${city}`
      );

      setWeather(response.data);
    } catch (err) {
      setError(err.response?.data?.error || 'Failed to fetch weather');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <input
        value={city}
        onChange={(e) => setCity(e.target.value)}
        placeholder="Enter city name"
      />
      <button onClick={handleFetch} disabled={loading}>
        Get Weather
      </button>

      {error && <p style={{ color: 'red' }}>{error}</p>}

      {weather && (
        <div>
          <h3>{weather.city}</h3>
          <p>Temperature: {weather.temperature}°C</p>
          <p>Description: {weather.description}</p>
          <p>Humidity: {weather.humidity}%</p>
        </div>
      )}
    </div>
  );
}
```

---

## 6.5 Scenario 4: Asynchronous Operations

### The Situation
Some operations take time (database queries, API calls). Backend must handle them efficiently.

### Example: Email Sending (Async Task)

```javascript
// utils/sendEmail.js

const nodemailer = require('nodemailer');

// Create email transporter
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASSWORD
  }
});

exports.sendWelcomeEmail = async (userEmail, userName) => {
  try {
    await transporter.sendMail({
      from: process.env.EMAIL_USER,
      to: userEmail,
      subject: 'Welcome!',
      html: `<h1>Welcome ${userName}!</h1><p>Thanks for joining us.</p>`
    });

    console.log(`✅ Email sent to ${userEmail}`);
  } catch (error) {
    console.error(`❌ Failed to send email: ${error.message}`);
  }
};
```

```javascript
// controllers/authController.js

const sendWelcomeEmail = require('../utils/sendEmail').sendWelcomeEmail;
const User = require('../models/User');

exports.register = async (req, res) => {
  try {
    const { email, name, password } = req.body;

    // Validate input
    if (!email || !name || !password) {
      return res.status(400).json({ error: 'Missing required fields' });
    }

    // Create user
    const newUser = await User.create({
      email,
      name,
      password
    });

    // Send welcome email (don't wait for it)
    // This runs in the background
    sendWelcomeEmail(email, name)
      .catch(err => console.error('Email send failed:', err));

    // Respond immediately (don't wait for email)
    res.status(201).json({
      success: true,
      message: 'Account created. Check your email.',
      user: {
        _id: newUser._id,
        email: newUser.email,
        name: newUser.name
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### Better: Using Message Queues

```javascript
// For production, use message queues (Redis, RabbitMQ)

const queue = require('bull');
const emailQueue = queue('email-queue', process.env.REDIS_URL);

// Process emails from queue
emailQueue.process(async (job) => {
  const { email, subject, message } = job.data;
  await sendEmail(email, subject, message);
});

// Add email to queue
exports.register = async (req, res) => {
  const newUser = await User.create(userData);

  // Add email job to queue
  await emailQueue.add({
    email: newUser.email,
    subject: 'Welcome!',
    message: `Hello ${newUser.name}`
  });

  // Respond immediately
  res.json({ success: true });
};
```

---

## 6.6 Scenario 5: Node.js Runtimes

### Beyond Node.js

Node.js is the standard, but alternatives exist:

### Node.js (Most Popular)
```javascript
// Traditional Node.js runtime

const http = require('http');
const server = http.createServer((req, res) => {
  res.end('Hello from Node.js');
});

server.listen(3000);
```

### Deno (Modern Alternative)
```typescript
// Deno - modern, secure JavaScript runtime

import { serve } from "https://deno.land/std@0.140.0/http/server.ts";

serve((req) => new Response("Hello from Deno"));
```

**Deno advantages:**
- Built-in TypeScript support
- Better security model
- Standard library
- Simpler module system

### Bun (Performance Focused)
```javascript
// Bun - ultra-fast JavaScript runtime

import { serve } from "bun";

export default {
  fetch() {
    return new Response("Hello from Bun");
  },
};
```

**Bun advantages:**
- 4x faster than Node.js
- Built-in package manager
- TypeScript support
- SQLite built-in

### Comparison

| Runtime | Speed | Ecosystem | Maturity | Job Market |
|---------|-------|-----------|----------|-----------|
| **Node.js** | Fast | ⭐⭐⭐⭐⭐ (npm) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Deno** | Fast | ⭐⭐ (growing) | ⭐⭐⭐ | ⭐⭐ |
| **Bun** | ⭐⭐⭐⭐⭐ | ⭐⭐ (early) | ⭐⭐ | ⭐ |

**For this course: We use Node.js** (most jobs, largest ecosystem, most mature)

---

## 6.7 Common Node.js Patterns

### Pattern 1: Async/Await

```javascript
// Modern approach (recommended)

async function getUserPosts(userId) {
  try {
    const user = await User.findById(userId);
    const posts = await Post.find({ userId });
    return { user, posts };
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### Pattern 2: Promises

```javascript
// Promise-based (older)

function getUserPosts(userId) {
  return User.findById(userId)
    .then(user => {
      return Post.find({ userId })
        .then(posts => ({ user, posts }));
    })
    .catch(error => console.error(error));
}
```

### Pattern 3: Callbacks

```javascript
// Callback-based (outdated)

function getUserPosts(userId, callback) {
  User.findById(userId, (err, user) => {
    if (err) return callback(err);
    
    Post.find({ userId }, (err, posts) => {
      if (err) return callback(err);
      
      callback(null, { user, posts });
    });
  });
}
```

**Best practice:** Use async/await (cleaner, easier to debug)

---

## 6.8 Error Handling in Scenarios

### Graceful Error Handling

```javascript
// controllers/userController.js

exports.getUser = async (req, res) => {
  try {
    const { id } = req.params;

    // Validate ID format
    if (!id.match(/^[0-9a-fA-F]{24}$/)) {
      return res.status(400).json({ error: 'Invalid user ID' });
    }

    // Query database
    const user = await User.findById(id);

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json(user);
  } catch (error) {
    // Log error for debugging
    console.error('Get user error:', error);

    // Generic error response (don't expose internal details)
    res.status(500).json({ error: 'Failed to fetch user' });
  }
};
```

---

## 6.9 Key Takeaways

✅ **Direct processing** - Immediate response  
✅ **File handling** - Upload, validate, store  
✅ **API integration** - Call external services  
✅ **Async operations** - Don't block responses  
✅ **Multiple runtimes** - Node.js most popular  
✅ **Use async/await** - Modern JavaScript pattern  
✅ **Error handling** - Always wrap in try/catch  

---

## 🎯 Next Steps

Now you understand real-world scenarios. Next: **Chapter 7: Project Setup and Deployment** - setting up your first professional project!

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 5: Directory Structure](./05_STANDARD_DIRECTORY_STRUCTURE.md) | [📖 Course Index](../INDEX.md) | [Chapter 7: Project Setup & Deployment →](./07_PROJECT_SETUP_DEPLOYMENT.md) |
