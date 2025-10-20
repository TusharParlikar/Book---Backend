# B06: Database Relationships 🔗

> **Connect Multiple Collections: One-to-Many, Many-to-Many**

**Chapter Level:** Beginner  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B05  

---

## Core Concepts

### Relationships in MongoDB

#### 1. One-to-One

```javascript
// User has ONE profile
const userSchema = new mongoose.Schema({
  name: String,
  profile: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Profile'
  }
});

const profileSchema = new mongoose.Schema({
  bio: String,
  avatar: String
});
```

#### 2. One-to-Many

```javascript
// User has MANY posts
const postSchema = new mongoose.Schema({
  title: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
});

// Get user with all posts
const user = await User.findById(userId).populate('posts');
```

#### 3. Many-to-Many

```javascript
// Post has MANY tags, Tag has MANY posts
const postSchema = new mongoose.Schema({
  title: String,
  tags: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Tag'
  }]
});

const tagSchema = new mongoose.Schema({
  name: String
});
```

### Populate: Load Related Data

```javascript
// Without populate: only ID returned
const post = await Post.findById(id);
// { _id: '123', title: 'Hello', author: '456' }

// With populate: full document returned
const post = await Post.findById(id).populate('author');
// { _id: '123', title: 'Hello', author: { _id: '456', name: 'John' } }
```

---

## Complete Example

```javascript
// ============================================================
// Models
// ============================================================

// Author Model
const authorSchema = new mongoose.Schema({
  name: String,
  email: String
});
const Author = mongoose.model('Author', authorSchema);

// Post Model (One-to-Many: Author has many Posts)
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Author',
    required: true
  },
  tags: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Tag'
  }]
}, { timestamps: true });
const Post = mongoose.model('Post', postSchema);

// Tag Model (Many-to-Many)
const tagSchema = new mongoose.Schema({
  name: String
});
const Tag = mongoose.model('Tag', tagSchema);

// ============================================================
// Controller
// ============================================================

// CREATE post with author
const createPost = async (req, res) => {
  try {
    const { title, content, authorId, tags } = req.body;
    
    // Create post
    const post = await Post.create({
      title,
      content,
      author: authorId,
      tags
    });
    
    // Populate author and tags
    await post.populate(['author', 'tags']);
    
    res.status(201).json({ success: true, data: post });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

// GET post with related data
const getPost = async (req, res) => {
  try {
    const post = await Post.findById(req.params.id)
      .populate('author')
      .populate('tags');
    
    res.json({ success: true, data: post });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

// GET author with all posts
const getAuthorWithPosts = async (req, res) => {
  try {
    const author = await Author.findById(req.params.id);
    const posts = await Post.find({ author: author._id });
    
    res.json({
      success: true,
      data: {
        author,
        posts,
        postCount: posts.length
      }
    });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

module.exports = {
  createPost,
  getPost,
  getAuthorWithPosts
};
```

---

## Interview Questions

**Q1: What does populate() do?**

Replaces ObjectId references with actual documents from related collection.

**Q2: Difference between embedding and referencing?**

- **Embedding:** Store related data directly in document
- **Referencing:** Store ObjectId and use populate() to fetch

---

| Previous | Index | Next |
|----------|-------|------|
| [B05: Mongoose Schemas](B05_MONGOOSE_SCHEMAS.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B07: Password Security](B07_PASSWORD_SECURITY.md) |
