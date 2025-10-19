# Chapter 7: Project Setup and Deployment

## 7.1 Project Initialization with npm

### Step 1: Create Project Directory

```bash
# Create new directory for project
mkdir my-backend-app
cd my-backend-app

# Initialize as Node.js project (creates package.json)
npm init -y

# The -y flag automatically accepts all defaults
```

### Understanding package.json

```json
{
  "name": "my-backend-app",
  "version": "1.0.0",
  "description": "My first backend application",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "keywords": ["backend", "api"],
  "author": "Your Name",
  "license": "ISC",
  "dependencies": {},
  "devDependencies": {}
}
```

---

## 7.2 Installing and Setting Up Express

### Step 1: Install Express

```bash
# Install Express framework
npm install express

# This adds express to dependencies in package.json
```

### Step 2: Create Basic Server

Create `server.js`:

```javascript
// server.js
// Main entry point for the application

const express = require('express');
const app = express();

// Middleware: Parse incoming JSON
app.use(express.json());

// Route: GET request
app.get('/', (req, res) => {
  res.json({ message: 'Hello from Express!' });
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`✅ Server running on port ${PORT}`);
});
```

### Step 3: Run Your Server

```bash
# Run the server
node server.js

# You should see: ✅ Server running on port 5000
```

### Test with React

```jsx
// React component
import { useEffect } from 'react';
import axios from 'axios';

export default function App() {
  useEffect(() => {
    axios.get('http://localhost:5000/')
      .then(res => console.log(res.data))
      .catch(err => console.error(err));
  }, []);

  return <h1>Check console</h1>;
}
```

---

## 7.3 Creating a Production Server Structure

### Project Structure

```
my-backend-app/
├── db/
│   └── index.js
├── models/
│   └── User.js
├── controllers/
│   └── userController.js
├── routes/
│   └── userRoutes.js
├── middlewares/
│   └── errorHandler.js
├── utils/
│   └── apiResponse.js
├── .env
├── .gitignore
├── package.json
└── server.js
```

### Complete server.js

```javascript
// server.js
// Production-ready Express server

const express = require('express');
const cors = require('cors');
const dotenv = require('dotenv');
const connectDB = require('./db');

// Load environment variables
dotenv.config();

// Connect to database
connectDB();

// Create Express app
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors());
app.use(express.static('public'));

// Routes
app.use('/api/users', require('./routes/userRoutes'));
app.use('/api/auth', require('./routes/authRoutes'));

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    success: false,
    error: err.message
  });
});

// Start server
const PORT = process.env.PORT || 5000;
const server = app.listen(PORT, () => {
  console.log(`✅ Server running on port ${PORT}`);
  console.log(`🌐 http://localhost:${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
  });
});
```

---

## 7.4 Environment Variables with dotenv

### Why Environment Variables?

```
❌ WRONG - Secrets in code:
const mongoURI = 'mongodb+srv://user:password@cluster.mongodb.net';

✅ RIGHT - Secrets in .env file:
MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net
```

### Step 1: Install dotenv

```bash
npm install dotenv
```

### Step 2: Create .env File

Create `.env` file in project root:

```
# Database
MONGODB_URI=mongodb://localhost:27017/myapp

# Server
PORT=5000
NODE_ENV=development

# JWT
JWT_SECRET=your-super-secret-key-change-this

# Email
EMAIL_USER=your-email@gmail.com
EMAIL_PASSWORD=your-app-password

# Cloudinary (optional - for file uploads)
CLOUDINARY_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

### Step 3: Load Environment Variables

```javascript
// server.js - at the very top

// Load environment variables from .env file
require('dotenv').config();

// Now access variables
const PORT = process.env.PORT;  // 5000
const URI = process.env.MONGODB_URI;  // mongodb://...
```

### Step 4: Create .gitignore

```
# .gitignore
# Never commit sensitive files

.env
node_modules/
dist/
.DS_Store
*.log
*.pid
npm-debug.log*
```

### Accessing Environment Variables

```javascript
// In any file, after dotenv.config()

console.log(process.env.PORT);           // 5000
console.log(process.env.JWT_SECRET);     // your-secret-key
console.log(process.env.NODE_ENV);       // development

// Check if variable exists before using
if (!process.env.MONGODB_URI) {
  console.error('❌ MONGODB_URI not defined in .env');
  process.exit(1);
}
```

---

## 7.5 Development Tools

### Install Nodemon (Auto-restart)

```bash
# Install as dev dependency
npm install --save-dev nodemon
```

Update `package.json`:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

Now run:
```bash
npm run dev
# Server restarts automatically when you save files!
```

### Install Prettier (Code Formatting)

```bash
npm install --save-dev prettier
```

Create `.prettierrc`:

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "arrowParens": "always"
}
```

Create `.prettierignore`:

```
node_modules
.env
dist
```

Add to `package.json`:

```json
{
  "scripts": {
    "format": "prettier --write ."
  }
}
```

Run `npm run format` to auto-format all code!

---

## 7.6 Essential npm Packages

### Core Dependencies

```bash
# Web framework
npm install express

# Database
npm install mongoose mongodb

# Security
npm install bcrypt jsonwebtoken

# Utilities
npm install dotenv cors

# File uploads
npm install multer cloudinary

# HTTP client (for calling APIs)
npm install axios
```

### Development Dependencies

```bash
# Auto-restart on file changes
npm install --save-dev nodemon

# Code formatter
npm install --save-dev prettier

# Code linter
npm install --save-dev eslint
```

---

## 7.7 Deploying to Cloud Providers

### Option 1: DigitalOcean

#### Step 1: Create Droplet

1. Go to DigitalOcean.com
2. Click "Create" → "Droplets"
3. Choose Node.js image
4. Select $5/month plan
5. Click "Create Droplet"

#### Step 2: Connect via SSH

```bash
# SSH into your droplet
ssh root@your_droplet_ip

# Update system
apt update && apt upgrade -y

# Install Node.js and npm
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
apt install -y nodejs
```

#### Step 3: Upload Your Code

```bash
# From your local machine, upload project
scp -r ~/my-backend-app root@your_droplet_ip:/var/www/

# SSH into droplet
ssh root@your_droplet_ip

# Navigate to project
cd /var/www/my-backend-app

# Install dependencies
npm install --production

# Start server
node server.js
```

### Option 2: Heroku (Easiest for Beginners)

#### Step 1: Install Heroku CLI

```bash
# Download and install from heroku.com/download
# Then login
heroku login
```

#### Step 2: Create Heroku App

```bash
# Create new app
heroku create my-awesome-app

# View your app
# https://my-awesome-app.herokuapp.com
```

#### Step 3: Set Environment Variables

```bash
# Set variables on Heroku
heroku config:set MONGODB_URI=mongodb://...
heroku config:set JWT_SECRET=your-secret

# View variables
heroku config
```

#### Step 4: Deploy

```bash
# Deploy your code
git push heroku main

# View logs
heroku logs --tail
```

### Option 3: Railway (Simple & Fast)

#### Step 1: Connect GitHub

1. Go to Railway.app
2. Click "New Project"
3. Click "Deploy from GitHub"
4. Select your repository

#### Step 2: Set Environment Variables

1. Go to "Variables"
2. Add your `.env` variables
3. Click "Deploy"

That's it! Railway automatically deploys when you push to GitHub!

---

## 7.8 Git Configuration and Version Control

### Initialize Git Repository

```bash
# Initialize git in your project
git init

# Add all files
git add .

# Create first commit
git commit -m "Initial commit: Basic Express server setup"

# Connect to GitHub repository
git remote add origin https://github.com/yourusername/my-backend-app.git

# Push to GitHub
git branch -M main
git push -u origin main
```

### Useful Git Commands

```bash
# Check status
git status

# Add changes
git add .

# Commit changes
git commit -m "Fix bug in user controller"

# Push to GitHub
git push

# Pull latest changes
git pull

# Create new branch
git checkout -b feature/add-posts

# Switch back to main
git checkout main
```

---

## 7.9 Tracking Empty Folders with .gitkeep

Git doesn't track empty folders. Use `.gitkeep` to track them:

```bash
# Create .gitkeep in empty folders

touch db/.gitkeep
touch models/.gitkeep
touch controllers/.gitkeep
touch middlewares/.gitkeep
touch utils/.gitkeep
touch public/temp/.gitkeep
```

These folders will now be tracked by Git even when empty!

---

## 7.10 Key Takeaways

✅ **npm init** - Initialize project  
✅ **npm install** - Add dependencies  
✅ **package.json** - Project configuration  
✅ **.env** - Store secrets safely  
✅ **nodemon** - Auto-reload during development  
✅ **Prettier** - Automatic code formatting  
✅ **DigitalOcean/Heroku/Railway** - Deploy to cloud  
✅ **Git** - Version control your code  

---

## 🎯 Next Steps

Your project is set up and deployed! Next: **Chapter 8: Full-Stack Integration** - connecting your React frontend to your Express backend!
