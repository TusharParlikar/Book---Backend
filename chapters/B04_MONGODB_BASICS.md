# B04: MongoDB Basics 🗄️

> **Master NoSQL Databases: Build a Complete Library Management System**

**Chapter Level:** Beginner  
**Time Required:** 3-4 hours  
**Prerequisites:** Completed B01-B03  
**What You'll Build:** Complete Library Book Catalog API with CRUD operations

---

## 📚 Table of Contents

1. [What is MongoDB?](#what-is-mongodb)
2. [SQL vs NoSQL](#sql-vs-nosql)
3. [MongoDB Setup](#mongodb-setup)
4. [Database Concepts](#database-concepts)
5. [CRUD Operations](#crud-operations)
6. [Querying Data](#querying-data)
7. [Complete Library Project](#complete-library-project)
8. [React Integration](#react-integration)
9. [Best Practices](#best-practices)
10. [Project Task](#project-task)
11. [Interview Questions](#interview-questions)

---

## What is MongoDB?

### Definition

**MongoDB** = **NoSQL database that stores data as JSON-like documents**

Think of it like:
```
SQL Database = Excel spreadsheet (rigid rows and columns)
MongoDB = Collection of sticky notes (flexible, each can be different)
```

### Key Features

```javascript
✅ FLEXIBLE SCHEMA
// Documents in same collection can have different fields
{ name: "Book 1", pages: 200 }
{ name: "Book 2", pages: 300, author: "John" }  // Extra field OK!

✅ JSON-LIKE DOCUMENTS
// Data looks like JavaScript objects
{
  _id: "abc123",
  title: "The Great Gatsby",
  author: "F. Scott Fitzgerald",
  year: 1925,
  genres: ["Fiction", "Classic"],
  available: true
}

✅ SCALABLE
// Can handle millions of documents
// Horizontal scaling (add more servers)

✅ FAST QUERIES
// Indexing for quick searches
// Optimized for read/write operations
```

---

## SQL vs NoSQL

### Visual Comparison

```
SQL (MySQL, PostgreSQL):
┌─────────────────────────────────┐
│  BOOKS TABLE                    │
├────┬──────────┬────────┬────────┤
│ id │  title   │ author │  year  │
├────┼──────────┼────────┼────────┤
│ 1  │ Book 1   │ John   │ 2020   │
│ 2  │ Book 2   │ Jane   │ 2021   │
└────┴──────────┴────────┴────────┘

NoSQL (MongoDB):
┌─────────────────────────────────┐
│  books COLLECTION               │
├─────────────────────────────────┤
│ { _id: 1, title: "Book 1",      │
│   author: "John", year: 2020,   │
│   genres: ["Fiction"] }         │
├─────────────────────────────────┤
│ { _id: 2, title: "Book 2",      │
│   author: "Jane", year: 2021,   │
│   publisher: "ABC",             │  ← Extra field!
│   genres: ["Non-fiction"] }     │
└─────────────────────────────────┘
```

### When to Use Each

| Use SQL When | Use MongoDB When |
|-------------|------------------|
| Data is structured and relational | Data is unstructured or semi-structured |
| Need complex joins | Documents are self-contained |
| ACID transactions critical | Flexibility and scalability needed |
| Banking, accounting systems | Social media, catalogs, real-time apps |

---

## MongoDB Setup

### Option 1: Cloud (MongoDB Atlas - Recommended for Beginners)

```bash
# 1. Go to https://www.mongodb.com/cloud/atlas
# 2. Sign up for FREE tier
# 3. Create cluster
# 4. Get connection string:
#    mongodb+srv://username:password@cluster.mongodb.net/database
```

### Option 2: Local Installation

```bash
# Windows:
# Download from https://www.mongodb.com/try/download/community
# Install MongoDB Community Server

# Mac:
brew tap mongodb/brew
brew install mongodb-community

# Linux:
sudo apt-get install mongodb

# Verify:
mongod --version
```

### Install Mongoose (MongoDB ODM)

```bash
npm install mongoose dotenv
```

**What is Mongoose?**
- **ODM** = Object Data Modeling library
- Makes MongoDB easier to use with Node.js
- Provides schema validation
- Adds helpful methods

---

## Database Concepts

### Terminology

```
SQL          →  MongoDB
─────────────────────────────
Database     →  Database
Table        →  Collection
Row          →  Document
Column       →  Field
Primary Key  →  _id (auto-generated)
Index        →  Index
Join         →  Embedded docs or references
```

### Document Structure

```javascript
// A MongoDB document (similar to JavaScript object)
{
  _id: ObjectId("507f1f77bcf86cd799439011"),  // Auto-generated unique ID
  title: "To Kill a Mockingbird",             // String field
  author: "Harper Lee",                       // String field
  publishedYear: 1960,                        // Number field
  genres: ["Fiction", "Classic", "Drama"],    // Array field
  pages: 281,                                 // Number field
  available: true,                            // Boolean field
  borrowedBy: null,                           // Null value
  ratings: {                                  // Nested object
    average: 4.8,
    count: 1523
  },
  createdAt: ISODate("2024-01-15T10:30:00Z") // Date field
}
```

---

## CRUD Operations

### C = Create (Insert Documents)

```javascript
// ============================================================
// CREATE: Add new documents to collection
// ============================================================

// Single document
const book = await Book.create({
  title: "1984",
  author: "George Orwell",
  publishedYear: 1949,
  genres: ["Fiction", "Dystopian"],
  pages: 328,
  available: true
});

// Multiple documents
const books = await Book.insertMany([
  {
    title: "Animal Farm",
    author: "George Orwell",
    publishedYear: 1945,
    genres: ["Fiction", "Satire"],
    pages: 112
  },
  {
    title: "Brave New World",
    author: "Aldous Huxley",
    publishedYear: 1932,
    genres: ["Fiction", "Dystopian"],
    pages: 268
  }
]);

console.log('Created books:', books);
```

### R = Read (Find Documents)

```javascript
// ============================================================
// READ: Query documents from collection
// ============================================================

// Find ALL documents
const allBooks = await Book.find();
console.log('All books:', allBooks);

// Find ONE document by ID
const book = await Book.findById("507f1f77bcf86cd799439011");
console.log('Found book:', book);

// Find ONE document by criteria
const firstAvailable = await Book.findOne({ available: true });
console.log('First available book:', firstAvailable);

// Find with FILTER
const fictionBooks = await Book.find({ genres: "Fiction" });
console.log('Fiction books:', fictionBooks);

// Find with MULTIPLE conditions
const recentClassics = await Book.find({
  genres: "Classic",
  publishedYear: { $gte: 1900 }  // Greater than or equal
});

// Find with SORT
const booksByYear = await Book.find().sort({ publishedYear: -1 }); // -1 = descending

// Find with LIMIT
const first5Books = await Book.find().limit(5);

// Find with SELECT (choose fields)
const bookTitles = await Book.find().select('title author -_id'); // Include title and author, exclude _id
```

### U = Update (Modify Documents)

```javascript
// ============================================================
// UPDATE: Modify existing documents
// ============================================================

// Update ONE document
const updated = await Book.findByIdAndUpdate(
  "507f1f77bcf86cd799439011",     // ID
  { available: false },             // Update
  { new: true }                     // Return updated document
);
console.log('Updated book:', updated);

// Update ONE by criteria
const result = await Book.updateOne(
  { title: "1984" },                // Filter
  { $set: { available: false } }    // Update
);
console.log('Modified count:', result.modifiedCount);

// Update MANY documents
const manyUpdated = await Book.updateMany(
  { publishedYear: { $lt: 1950 } }, // All books before 1950
  { $set: { category: "Classic" } } // Add category field
);
console.log('Updated', manyUpdated.modifiedCount, 'books');

// Update with INCREMENT
await Book.findByIdAndUpdate(
  bookId,
  { $inc: { borrowCount: 1 } }      // Increment borrowCount by 1
);

// Update with PUSH (add to array)
await Book.findByIdAndUpdate(
  bookId,
  { $push: { genres: "Award-winning" } }  // Add genre to array
);
```

### D = Delete (Remove Documents)

```javascript
// ============================================================
// DELETE: Remove documents from collection
// ============================================================

// Delete ONE document by ID
const deleted = await Book.findByIdAndDelete("507f1f77bcf86cd799439011");
console.log('Deleted book:', deleted);

// Delete ONE by criteria
const result = await Book.deleteOne({ title: "Test Book" });
console.log('Deleted count:', result.deletedCount);

// Delete MANY documents
const manyDeleted = await Book.deleteMany({
  available: false,
  publishedYear: { $lt: 1900 }
});
console.log('Deleted', manyDeleted.deletedCount, 'old unavailable books');
```

---

## Querying Data

### Query Operators

```javascript
// ============================================================
// COMPARISON OPERATORS
// ============================================================

// $eq - Equal
const books = await Book.find({ publishedYear: { $eq: 2020 } });
// Same as: await Book.find({ publishedYear: 2020 });

// $ne - Not equal
const notFiction = await Book.find({ genres: { $ne: "Fiction" } });

// $gt, $gte - Greater than, Greater than or equal
const modern = await Book.find({ publishedYear: { $gt: 2000 } });
const since2000 = await Book.find({ publishedYear: { $gte: 2000 } });

// $lt, $lte - Less than, Less than or equal
const old = await Book.find({ publishedYear: { $lt: 1950 } });
const before1950 = await Book.find({ publishedYear: { $lte: 1950 } });

// $in - In array
const specificAuthors = await Book.find({
  author: { $in: ["George Orwell", "Aldous Huxley", "Ray Bradbury"] }
});

// $nin - Not in array
const excludeAuthors = await Book.find({
  author: { $nin: ["Test Author"] }
});

// ============================================================
// LOGICAL OPERATORS
// ============================================================

// $and - All conditions must be true
const andQuery = await Book.find({
  $and: [
    { genres: "Fiction" },
    { publishedYear: { $gte: 2000 } },
    { available: true }
  ]
});

// $or - At least one condition must be true
const orQuery = await Book.find({
  $or: [
    { author: "George Orwell" },
    { genres: "Dystopian" }
  ]
});

// $not - Negation
const notClassic = await Book.find({
  genres: { $not: { $eq: "Classic" } }
});

// ============================================================
// ARRAY OPERATORS
// ============================================================

// $all - Array contains all elements
const multiGenre = await Book.find({
  genres: { $all: ["Fiction", "Classic"] }  // Has BOTH genres
});

// $size - Array has specific length
const twoGenres = await Book.find({
  genres: { $size: 2 }  // Exactly 2 genres
});

// ============================================================
// STRING SEARCH
// ============================================================

// $regex - Regular expression match
const titleSearch = await Book.find({
  title: { $regex: /great/i }  // Case-insensitive search for "great"
});

// Text search (requires text index)
const textSearch = await Book.find({
  $text: { $search: "dystopian future" }
});
```

### Advanced Queries

```javascript
// ============================================================
// PAGINATION
// ============================================================

const page = 2;
const limit = 10;
const skip = (page - 1) * limit;

const paginatedBooks = await Book.find()
  .skip(skip)      // Skip first 10 documents
  .limit(limit)    // Return next 10 documents
  .sort({ createdAt: -1 });  // Newest first

const totalBooks = await Book.countDocuments();
const totalPages = Math.ceil(totalBooks / limit);

// ============================================================
// PROJECTION (Select specific fields)
// ============================================================

// Include only specific fields
const titles = await Book.find().select('title author');
// Returns: [{ _id: "...", title: "...", author: "..." }]

// Exclude specific fields
const withoutId = await Book.find().select('-_id -__v');

// ============================================================
// AGGREGATION (Advanced)
// ============================================================

// Count books by genre
const genreCount = await Book.aggregate([
  { $unwind: "$genres" },  // Split array into separate documents
  { $group: {
      _id: "$genres",
      count: { $sum: 1 }
    }
  },
  { $sort: { count: -1 } }
]);
// Result: [{ _id: "Fiction", count: 25 }, { _id: "Classic", count: 12 }]

// Average pages by author
const avgPagesByAuthor = await Book.aggregate([
  { $group: {
      _id: "$author",
      avgPages: { $avg: "$pages" },
      bookCount: { $sum: 1 }
    }
  }
]);
```

---

## Complete Library Project

### Step 1: Project Setup

```javascript
// ============================================================
// FILE: server.js
// ============================================================

const express = require('express');
const mongoose = require('mongoose');
require('dotenv').config();

const app = express();

// MIDDLEWARE
app.use(express.json());

// CONNECT TO MONGODB
const MONGO_URI = process.env.MONGO_URI || 'mongodb://localhost:27017/library-db';

mongoose.connect(MONGO_URI)
  .then(() => console.log('✅ Connected to MongoDB'))
  .catch((error) => console.error('❌ MongoDB connection error:', error));

// ROUTES
const bookRoutes = require('./routes/bookRoutes');
app.use('/api/books', bookRoutes);

// ERROR HANDLER
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ success: false, error: err.message });
});

// START SERVER
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`✅ Server running on port ${PORT}`);
});
```

### Step 2: Create Book Model

```javascript
// ============================================================
// FILE: models/Book.js
// PURPOSE: Define book schema and model
// ============================================================

const mongoose = require('mongoose');

// SCHEMA: Define structure of book documents
const bookSchema = new mongoose.Schema(
  {
    // REQUIRED FIELDS
    title: {
      type: String,
      required: [true, 'Book title is required'],
      trim: true,
      minlength: [1, 'Title must be at least 1 character'],
      maxlength: [200, 'Title cannot exceed 200 characters']
    },
    
    author: {
      type: String,
      required: [true, 'Author name is required'],
      trim: true
    },
    
    // OPTIONAL FIELDS
    isbn: {
      type: String,
      unique: true,  // No duplicate ISBNs
      sparse: true    // Allow null/undefined (optional unique)
    },
    
    publishedYear: {
      type: Number,
      min: [1000, 'Year must be after 1000'],
      max: [new Date().getFullYear(), 'Year cannot be in the future']
    },
    
    pages: {
      type: Number,
      min: [1, 'Pages must be at least 1']
    },
    
    genres: {
      type: [String],  // Array of strings
      default: []
    },
    
    language: {
      type: String,
      default: 'English'
    },
    
    publisher: String,
    
    description: {
      type: String,
      maxlength: [2000, 'Description too long']
    },
    
    coverImage: String,
    
    // AVAILABILITY
    available: {
      type: Boolean,
      default: true
    },
    
    quantity: {
      type: Number,
      default: 1,
      min: [0, 'Quantity cannot be negative']
    },
    
    // RATINGS
    rating: {
      average: {
        type: Number,
        min: 0,
        max: 5,
        default: 0
      },
      count: {
        type: Number,
        default: 0
      }
    }
  },
  {
    timestamps: true  // Adds createdAt and updatedAt automatically
  }
);

// INDEXES for faster queries
bookSchema.index({ title: 'text', author: 'text' });  // Text search
bookSchema.index({ genres: 1 });                       // Genre filter
bookSchema.index({ publishedYear: -1 });              // Sort by year

// MODEL: Create model from schema
const Book = mongoose.model('Book', bookSchema);

module.exports = Book;
```

### Step 3: Create Controllers

```javascript
// ============================================================
// FILE: controllers/bookController.js
// PURPOSE: Handle book-related logic
// ============================================================

const Book = require('../models/Book');

// ============================================================
// CREATE: Add new book
// ============================================================
const createBook = async (req, res) => {
  try {
    // Extract data from request body
    const bookData = req.body;
    
    // Create book in database
    const book = await Book.create(bookData);
    
    // Send success response
    res.status(201).json({
      success: true,
      message: 'Book created successfully',
      data: book
    });
    
  } catch (error) {
    // Handle validation errors
    if (error.name === 'ValidationError') {
      return res.status(400).json({
        success: false,
        error: 'Validation failed',
        details: error.message
      });
    }
    
    // Handle duplicate ISBN
    if (error.code === 11000) {
      return res.status(409).json({
        success: false,
        error: 'Book with this ISBN already exists'
      });
    }
    
    // Other errors
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// ============================================================
// READ: Get all books with filtering
// ============================================================
const getAllBooks = async (req, res) => {
  try {
    // EXTRACT QUERY PARAMETERS
    const {
      genre,
      author,
      year,
      minPages,
      maxPages,
      available,
      search,
      sort = '-createdAt',  // Default sort by newest
      page = 1,
      limit = 10
    } = req.query;
    
    // BUILD FILTER OBJECT
    const filter = {};
    
    if (genre) {
      filter.genres = genre;  // Books with this genre
    }
    
    if (author) {
      filter.author = { $regex: author, $options: 'i' };  // Case-insensitive partial match
    }
    
    if (year) {
      filter.publishedYear = parseInt(year);
    }
    
    if (minPages || maxPages) {
      filter.pages = {};
      if (minPages) filter.pages.$gte = parseInt(minPages);
      if (maxPages) filter.pages.$lte = parseInt(maxPages);
    }
    
    if (available !== undefined) {
      filter.available = available === 'true';
    }
    
    // TEXT SEARCH
    if (search) {
      filter.$text = { $search: search };
    }
    
    // PAGINATION
    const skip = (parseInt(page) - 1) * parseInt(limit);
    
    // EXECUTE QUERY
    const books = await Book.find(filter)
      .sort(sort)
      .skip(skip)
      .limit(parseInt(limit));
    
    // GET TOTAL COUNT
    const total = await Book.countDocuments(filter);
    
    // SEND RESPONSE
    res.json({
      success: true,
      data: books,
      pagination: {
        total,
        page: parseInt(page),
        limit: parseInt(limit),
        pages: Math.ceil(total / parseInt(limit))
      }
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// ============================================================
// READ: Get single book by ID
// ============================================================
const getBookById = async (req, res) => {
  try {
    const { id } = req.params;
    
    // Find book by ID
    const book = await Book.findById(id);
    
    // Check if book exists
    if (!book) {
      return res.status(404).json({
        success: false,
        error: 'Book not found'
      });
    }
    
    // Send response
    res.json({
      success: true,
      data: book
    });
    
  } catch (error) {
    // Handle invalid ID format
    if (error.name === 'CastError') {
      return res.status(400).json({
        success: false,
        error: 'Invalid book ID format'
      });
    }
    
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// ============================================================
// UPDATE: Update book
// ============================================================
const updateBook = async (req, res) => {
  try {
    const { id } = req.params;
    const updates = req.body;
    
    // Find and update
    const book = await Book.findByIdAndUpdate(
      id,
      updates,
      {
        new: true,           // Return updated document
        runValidators: true  // Run schema validation
      }
    );
    
    // Check if book exists
    if (!book) {
      return res.status(404).json({
        success: false,
        error: 'Book not found'
      });
    }
    
    // Send response
    res.json({
      success: true,
      message: 'Book updated successfully',
      data: book
    });
    
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(400).json({
        success: false,
        error: 'Validation failed',
        details: error.message
      });
    }
    
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// ============================================================
// DELETE: Remove book
// ============================================================
const deleteBook = async (req, res) => {
  try {
    const { id } = req.params;
    
    // Find and delete
    const book = await Book.findByIdAndDelete(id);
    
    // Check if book exists
    if (!book) {
      return res.status(404).json({
        success: false,
        error: 'Book not found'
      });
    }
    
    // Send response
    res.json({
      success: true,
      message: 'Book deleted successfully',
      data: book
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// ============================================================
// STATISTICS: Get book statistics
// ============================================================
const getStatistics = async (req, res) => {
  try {
    // Total books
    const totalBooks = await Book.countDocuments();
    
    // Available books
    const availableBooks = await Book.countDocuments({ available: true });
    
    // Books by genre
    const genreStats = await Book.aggregate([
      { $unwind: "$genres" },
      { $group: {
          _id: "$genres",
          count: { $sum: 1 }
        }
      },
      { $sort: { count: -1 } }
    ]);
    
    // Top authors (by book count)
    const topAuthors = await Book.aggregate([
      { $group: {
          _id: "$author",
          bookCount: { $sum: 1 }
        }
      },
      { $sort: { bookCount: -1 } },
      { $limit: 10 }
    ]);
    
    res.json({
      success: true,
      data: {
        totalBooks,
        availableBooks,
        borrowedBooks: totalBooks - availableBooks,
        genreStats,
        topAuthors
      }
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
};

// EXPORT ALL CONTROLLERS
module.exports = {
  createBook,
  getAllBooks,
  getBookById,
  updateBook,
  deleteBook,
  getStatistics
};
```

### Step 4: Create Routes

```javascript
// ============================================================
// FILE: routes/bookRoutes.js
// PURPOSE: Define API endpoints for books
// ============================================================

const express = require('express');
const router = express.Router();
const bookController = require('../controllers/bookController');

// ============================================================
// ROUTES
// ============================================================

// GET /api/books - Get all books (with filtering)
router.get('/', bookController.getAllBooks);

// GET /api/books/stats - Get statistics
router.get('/stats', bookController.getStatistics);

// GET /api/books/:id - Get single book
router.get('/:id', bookController.getBookById);

// POST /api/books - Create new book
router.post('/', bookController.createBook);

// PUT /api/books/:id - Update book
router.put('/:id', bookController.updateBook);

// DELETE /api/books/:id - Delete book
router.delete('/:id', bookController.deleteBook);

module.exports = router;
```

### Step 5: Environment Variables

```bash
# ============================================================
# FILE: .env
# ============================================================

MONGO_URI=mongodb://localhost:27017/library-db
# OR for MongoDB Atlas:
# MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/library-db

PORT=5000
NODE_ENV=development
```

---

## React Integration

### React Component: Book Catalog

```javascript
// ============================================================
// FILE: BookCatalog.jsx (React Component)
// PURPOSE: Display and manage books
// ============================================================

import React, { useState, useEffect } from 'react';

function BookCatalog() {
  // STATE
  const [books, setBooks] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [filters, setFilters] = useState({
    genre: '',
    author: '',
    search: ''
  });
  const [page, setPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);

  // FETCH BOOKS
  useEffect(() => {
    fetchBooks();
  }, [filters, page]);

  const fetchBooks = async () => {
    try {
      setLoading(true);
      
      // BUILD QUERY STRING
      const params = new URLSearchParams({
        page: page,
        limit: 10,
        ...filters
      });
      
      // FETCH FROM API
      const response = await fetch(`http://localhost:5000/api/books?${params}`);
      const data = await response.json();
      
      if (data.success) {
        setBooks(data.data);
        setTotalPages(data.pagination.pages);
      } else {
        setError(data.error);
      }
      
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  // DELETE BOOK
  const deleteBook = async (id) => {
    if (!window.confirm('Delete this book?')) return;
    
    try {
      const response = await fetch(`http://localhost:5000/api/books/${id}`, {
        method: 'DELETE'
      });
      
      const data = await response.json();
      
      if (data.success) {
        alert('Book deleted!');
        fetchBooks(); // Refresh list
      }
    } catch (err) {
      alert('Error deleting book');
    }
  };

  // RENDER
  if (loading) return <div>Loading books...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="book-catalog">
      <h1>Library Book Catalog</h1>
      
      {/* FILTERS */}
      <div className="filters">
        <input
          type="text"
          placeholder="Search books..."
          value={filters.search}
          onChange={(e) => setFilters({ ...filters, search: e.target.value })}
        />
        
        <input
          type="text"
          placeholder="Filter by author..."
          value={filters.author}
          onChange={(e) => setFilters({ ...filters, author: e.target.value })}
        />
        
        <select
          value={filters.genre}
          onChange={(e) => setFilters({ ...filters, genre: e.target.value })}
        >
          <option value="">All Genres</option>
          <option value="Fiction">Fiction</option>
          <option value="Non-fiction">Non-fiction</option>
          <option value="Classic">Classic</option>
          <option value="Science Fiction">Science Fiction</option>
        </select>
        
        <button onClick={() => setFilters({ genre: '', author: '', search: '' })}>
          Clear Filters
        </button>
      </div>

      {/* BOOK LIST */}
      <div className="book-list">
        {books.length === 0 ? (
          <p>No books found</p>
        ) : (
          books.map(book => (
            <div key={book._id} className="book-card">
              <h3>{book.title}</h3>
              <p>by {book.author}</p>
              <p>Year: {book.publishedYear}</p>
              <p>Genres: {book.genres.join(', ')}</p>
              <p>Pages: {book.pages}</p>
              <p className={book.available ? 'available' : 'borrowed'}>
                {book.available ? '✅ Available' : '❌ Borrowed'}
              </p>
              
              <button onClick={() => deleteBook(book._id)}>
                Delete
              </button>
            </div>
          ))
        )}
      </div>

      {/* PAGINATION */}
      <div className="pagination">
        <button 
          onClick={() => setPage(p => Math.max(1, p - 1))}
          disabled={page === 1}
        >
          Previous
        </button>
        
        <span>Page {page} of {totalPages}</span>
        
        <button 
          onClick={() => setPage(p => Math.min(totalPages, p + 1))}
          disabled={page === totalPages}
        >
          Next
        </button>
      </div>
    </div>
  );
}

export default BookCatalog;
```

### React Component: Add Book Form

```javascript
// ============================================================
// FILE: AddBookForm.jsx (React Component)
// ============================================================

import React, { useState } from 'react';

function AddBookForm({ onBookAdded }) {
  const [formData, setFormData] = useState({
    title: '',
    author: '',
    isbn: '',
    publishedYear: '',
    pages: '',
    genres: [],
    language: 'English',
    publisher: '',
    description: '',
    quantity: 1
  });
  
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState(null);

  // HANDLE INPUT CHANGE
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  // HANDLE GENRE SELECTION
  const handleGenreChange = (e) => {
    const selectedOptions = Array.from(e.target.selectedOptions, option => option.value);
    setFormData(prev => ({
      ...prev,
      genres: selectedOptions
    }));
  };

  // SUBMIT FORM
  const handleSubmit = async (e) => {
    e.preventDefault();
    setSubmitting(true);
    setError(null);
    
    try {
      // SEND POST REQUEST
      const response = await fetch('http://localhost:5000/api/books', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          ...formData,
          publishedYear: parseInt(formData.publishedYear),
          pages: parseInt(formData.pages),
          quantity: parseInt(formData.quantity)
        })
      });
      
      const data = await response.json();
      
      if (data.success) {
        alert('Book added successfully!');
        // Reset form
        setFormData({
          title: '',
          author: '',
          isbn: '',
          publishedYear: '',
          pages: '',
          genres: [],
          language: 'English',
          publisher: '',
          description: '',
          quantity: 1
        });
        
        // Callback to parent
        if (onBookAdded) onBookAdded();
      } else {
        setError(data.error || 'Failed to add book');
      }
      
    } catch (err) {
      setError(err.message);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <div className="add-book-form">
      <h2>Add New Book</h2>
      
      {error && <div className="error">{error}</div>}
      
      <form onSubmit={handleSubmit}>
        <div>
          <label>Title *</label>
          <input
            type="text"
            name="title"
            value={formData.title}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label>Author *</label>
          <input
            type="text"
            name="author"
            value={formData.author}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label>ISBN</label>
          <input
            type="text"
            name="isbn"
            value={formData.isbn}
            onChange={handleChange}
          />
        </div>

        <div>
          <label>Published Year</label>
          <input
            type="number"
            name="publishedYear"
            value={formData.publishedYear}
            onChange={handleChange}
            min="1000"
            max={new Date().getFullYear()}
          />
        </div>

        <div>
          <label>Pages</label>
          <input
            type="number"
            name="pages"
            value={formData.pages}
            onChange={handleChange}
            min="1"
          />
        </div>

        <div>
          <label>Genres</label>
          <select
            name="genres"
            multiple
            value={formData.genres}
            onChange={handleGenreChange}
          >
            <option value="Fiction">Fiction</option>
            <option value="Non-fiction">Non-fiction</option>
            <option value="Classic">Classic</option>
            <option value="Science Fiction">Science Fiction</option>
            <option value="Fantasy">Fantasy</option>
            <option value="Mystery">Mystery</option>
            <option value="Romance">Romance</option>
          </select>
        </div>

        <div>
          <label>Language</label>
          <input
            type="text"
            name="language"
            value={formData.language}
            onChange={handleChange}
          />
        </div>

        <div>
          <label>Publisher</label>
          <input
            type="text"
            name="publisher"
            value={formData.publisher}
            onChange={handleChange}
          />
        </div>

        <div>
          <label>Description</label>
          <textarea
            name="description"
            value={formData.description}
            onChange={handleChange}
            rows="4"
          />
        </div>

        <div>
          <label>Quantity</label>
          <input
            type="number"
            name="quantity"
            value={formData.quantity}
            onChange={handleChange}
            min="1"
          />
        </div>

        <button type="submit" disabled={submitting}>
          {submitting ? 'Adding...' : 'Add Book'}
        </button>
      </form>
    </div>
  );
}

export default AddBookForm;
```

---

## Best Practices

### 1. Connection Management

```javascript
// ✅ GOOD: Handle connection events
mongoose.connection.on('connected', () => {
  console.log('MongoDB connected');
});

mongoose.connection.on('error', (err) => {
  console.error('MongoDB error:', err);
});

mongoose.connection.on('disconnected', () => {
  console.log('MongoDB disconnected');
});

// Graceful shutdown
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  process.exit(0);
});
```

### 2. Error Handling

```javascript
// ✅ GOOD: Specific error handling
try {
  const book = await Book.create(data);
} catch (error) {
  if (error.name === 'ValidationError') {
    // Handle validation errors
  } else if (error.code === 11000) {
    // Handle duplicate key errors
  } else {
    // Handle other errors
  }
}
```

### 3. Indexing

```javascript
// ✅ GOOD: Add indexes for frequently queried fields
bookSchema.index({ title: 'text' });     // Text search
bookSchema.index({ author: 1 });         // Author queries
bookSchema.index({ genres: 1 });         // Genre filtering
bookSchema.index({ publishedYear: -1 }); // Year sorting
```

### 4. Query Optimization

```javascript
// ❌ BAD: Fetch all fields
const books = await Book.find();

// ✅ GOOD: Select only needed fields
const books = await Book.find().select('title author');

// ✅ GOOD: Use lean() for read-only data (faster)
const books = await Book.find().lean();
```

### 5. Environment Variables

```javascript
// ✅ GOOD: Never hardcode credentials
const MONGO_URI = process.env.MONGO_URI;

// ❌ BAD: Hardcoded
const MONGO_URI = 'mongodb://username:password@...';
```

---

## Project Task

### 🎯 Build Your Own Library Management API

**Time:** 4-5 hours  
**Difficulty:** Beginner to Intermediate

### Requirements

#### Part 1: Backend Setup (1 hour)

1. ✅ Set up MongoDB (Atlas or local)
2. ✅ Create Express server
3. ✅ Connect to MongoDB
4. ✅ Create Book model with all fields:
   - title, author, isbn, publishedYear, pages
   - genres (array), language, publisher, description
   - available, quantity, rating

#### Part 2: CRUD Operations (2 hours)

1. ✅ **CREATE**: Add new books
   - Validate required fields
   - Handle duplicate ISBN errors
   - Return created book

2. ✅ **READ**: Get books with filtering
   - Get all books (with pagination)
   - Get single book by ID
   - Filter by: genre, author, year, pages
   - Text search by title/author
   - Sort by different fields

3. ✅ **UPDATE**: Modify books
   - Update any field
   - Run validation
   - Return updated book

4. ✅ **DELETE**: Remove books
   - Delete by ID
   - Return deleted book

#### Part 3: Advanced Features (1 hour)

1. ✅ Statistics endpoint:
   - Total books
   - Available vs borrowed
   - Books by genre
   - Top 10 authors

2. ✅ Pagination:
   - Page and limit query params
   - Return total count and pages

3. ✅ Error handling:
   - Validation errors (400)
   - Not found errors (404)
   - Duplicate errors (409)
   - Server errors (500)

#### Part 4: React Frontend (1 hour)

1. ✅ Book catalog component:
   - Display all books
   - Search and filter
   - Pagination controls

2. ✅ Add book form:
   - All form fields
   - Validation
   - Success/error messages

3. ✅ Book details page:
   - Show single book
   - Edit and delete buttons

### Test Your API

```bash
# 1. Create book
POST http://localhost:5000/api/books
{
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald",
  "publishedYear": 1925,
  "pages": 180,
  "genres": ["Fiction", "Classic"]
}

# 2. Get all books
GET http://localhost:5000/api/books

# 3. Filter books
GET http://localhost:5000/api/books?genre=Fiction&page=1&limit=10

# 4. Search books
GET http://localhost:5000/api/books?search=gatsby

# 5. Get statistics
GET http://localhost:5000/api/books/stats

# 6. Update book
PUT http://localhost:5000/api/books/[ID]
{
  "available": false
}

# 7. Delete book
DELETE http://localhost:5000/api/books/[ID]
```

### Bonus Challenges

1. 🌟 Add book borrowing system (borrower name, due date)
2. 🌟 Add book reviews and ratings
3. 🌟 Add book categories/sections
4. 🌟 Add book reservations queue
5. 🌟 Add overdue book tracking

---

## Interview Questions

### Q1: What is MongoDB and why use it?

**Answer:** MongoDB is a NoSQL database that stores data as JSON-like documents. Use it when you need:
- Flexible schema (data structure can change)
- Horizontal scalability
- Fast development (no rigid tables)
- Document-based data (like user profiles, products)

### Q2: What's the difference between SQL and NoSQL?

**Answer:**
- **SQL:** Structured (tables, rows, columns), relational, ACID transactions, rigid schema
- **NoSQL:** Flexible documents, non-relational, eventually consistent, dynamic schema

### Q3: What is Mongoose?

**Answer:** Mongoose is an ODM (Object Data Modeling) library for MongoDB and Node.js. It provides:
- Schema definition and validation
- Middleware (hooks)
- Query helpers
- Easier MongoDB operations

### Q4: Explain CRUD operations

**Answer:**
- **C**reate: `Book.create()`, `insertOne()`, `insertMany()`
- **R**ead: `Book.find()`, `findOne()`, `findById()`
- **U**pdate: `updateOne()`, `updateMany()`, `findByIdAndUpdate()`
- **D**elete: `deleteOne()`, `deleteMany()`, `findByIdAndDelete()`

### Q5: What are MongoDB indexes?

**Answer:** Indexes improve query performance by creating a data structure that allows faster lookups. Like an index in a book - instead of reading every page, you jump to the right section.

### Q6: How do you handle pagination in MongoDB?

**Answer:**
```javascript
const page = 2;
const limit = 10;
const skip = (page - 1) * limit;

const books = await Book.find()
  .skip(skip)
  .limit(limit);
```

### Q7: What's the difference between `findByIdAndUpdate` with `new: true` vs `new: false`?

**Answer:**
- `new: true`: Returns the UPDATED document
- `new: false` (default): Returns the ORIGINAL document before update

### Q8: How do you search text in MongoDB?

**Answer:**
```javascript
// 1. Create text index
bookSchema.index({ title: 'text', author: 'text' });

// 2. Search
const books = await Book.find({
  $text: { $search: 'gatsby' }
});
```

### Q9: What's the purpose of `lean()` in Mongoose?

**Answer:** `lean()` returns plain JavaScript objects instead of Mongoose documents. It's faster but you lose Mongoose methods (like `.save()`). Use for read-only operations.

### Q10: How do you handle MongoDB connection errors?

**Answer:**
```javascript
mongoose.connect(URI)
  .then(() => console.log('Connected'))
  .catch(err => console.error('Error:', err));

mongoose.connection.on('error', (err) => {
  console.error('Connection error:', err);
});
```

---

## 🎯 Practice Projects

Now build these projects using MongoDB:

**Recommended Projects:**
- [Project #14: Recipe Manager](../projects/14-recipe-manager.md) - Complex schemas ⭐
- [Project #15: Task Scheduler](../projects/15-task-scheduler.md) - Recurring tasks
- [Project #16: Library Management](../projects/16-library-management.md) - Books & borrowing ⭐
- [Project #17: Course Management](../projects/17-course-management.md) - Hierarchical data

**Time Required:** 4-6 hours per project

---

| Previous | Index | Next |
|----------|-------|------|
| [B03: MVC Architecture](B03_MVC_ARCHITECTURE.md) | [INDEX.md](../INDEX.md) | [B05: Mongoose Schemas](B05_MONGOOSE_SCHEMAS.md) |

