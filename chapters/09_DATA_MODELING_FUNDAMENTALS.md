# Chapter 9: Data Modeling Fundamentals

## 9.1 Why Pre-Code Data Modeling Matters

Before writing any code, **plan your database structure**.

### Bad Approach (Code First)
```
Start coding → Run into data problems → Rewrite everything
```

### Good Approach (Design First)
```
Plan database → Write schemas → Clean, scalable code
```

## 9.2 Professional Data Modeling Tools

### Tools to Use

1. **Moon Modeler** (Paid)
   - Visual diagram editor
   - Supports MongoDB, SQL
   - Generates code

2. **Eraser.io** (Free)
   - Online collaborative tool
   - Create ER diagrams
   - Real-time sharing

3. **Pen and Paper** (Effective)
   - Draw entity relationships
   - Identify fields
   - Plan connections

### Example ER Diagram

```
┌─────────────┐         ┌──────────────┐
│    User     │         │     Post     │
├─────────────┤         ├──────────────┤
│ _id         │─────1:M─│ _id          │
│ name        │         │ title        │
│ email       │         │ userId (ref) │
│ password    │         │ content      │
└─────────────┘         └──────────────┘
```

## 9.3 Introduction to Mongoose (ODM for MongoDB)

Mongoose makes MongoDB easier by adding:
- Schema validation
- Type checking
- Relationships
- Custom methods
- Pre/post hooks

### Install Mongoose

```bash
npm install mongoose
```

### Basic Connection

```javascript
// db/index.js

const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log('✅ MongoDB connected');
  } catch (error) {
    console.error('❌ Connection failed:', error);
    process.exit(1);
  }
};

module.exports = connectDB;
```

---

## 🎯 Next: Chapter 10 - Basic Data Models with Mongoose
