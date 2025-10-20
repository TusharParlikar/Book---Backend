# Project 5: Blog Post API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md), [B03 - MVC Architecture](../chapters/B03_MVC_ARCHITECTURE.md)

---

## 📝 Description

Create a blog API with posts and comments. Learn about nested resources, relationships between data, and MVC architecture.

**What You'll Learn:**
- MVC pattern implementation
- Nested routes (posts → comments)
- Data relationships
- Slug generation
- Rich text handling

---

## 🎯 Project Goals

- ✅ CRUD for blog posts
- ✅ CRUD for comments (nested under posts)
- ✅ URL slugs for SEO
- ✅ MVC architecture
- ✅ React blog with post list and detail pages

---

## 🗺️ Roadmap

### Step 1: Setup MVC Structure
**Todo:**
- [ ] Create folders: models, controllers, routes
- [ ] Install: express, cors, uuid, slugify
- [ ] Setup basic server
- [ ] Create posts model (data structure)
- [ ] Create comments model

**MVC Folders:**
```
- models/      (data structures)
- controllers/ (business logic)
- routes/      (API endpoints)
```

**Hint:** Use `slugify` package to generate URL-friendly slugs from titles

---

### Step 2: Posts Model
**Todo:**
- [ ] Create `models/Post.js`
- [ ] Define post structure:
  - id (unique)
  - title (required)
  - slug (auto-generated from title)
  - content (required, can be long)
  - author (string)
  - excerpt (short description)
  - featuredImage (URL)
  - tags (array of strings)
  - published (boolean)
  - createdAt, updatedAt (timestamps)
- [ ] In-memory storage array

**Hint:** Slug example: "My First Post" → "my-first-post"

---

### Step 3: Comments Model
**Todo:**
- [ ] Create `models/Comment.js`
- [ ] Define comment structure:
  - id (unique)
  - postId (which post it belongs to)
  - author (string)
  - email (string)
  - content (required)
  - createdAt (timestamp)
- [ ] In-memory storage array

**Relationship:** Comment belongs to Post (using postId)

---

### Step 4: Posts Controller
**Todo:**
- [ ] Create `controllers/postController.js`
- [ ] Implement functions:
  - getAllPosts()
  - getPostById()
  - getPostBySlug()
  - createPost()
  - updatePost()
  - deletePost()
  - getPublishedPosts()
  - getPostsByTag()

**Hint:** Keep controllers thin - just business logic, no routing

---

### Step 5: Posts Routes
**Todo:**
- [ ] Create `routes/posts.js`
- [ ] Define routes:
  - `GET /api/posts` - All posts
  - `GET /api/posts/published` - Only published
  - `GET /api/posts/:slug` - Get by slug
  - `POST /api/posts` - Create new
  - `PUT /api/posts/:id` - Update
  - `DELETE /api/posts/:id` - Delete
  - `GET /api/posts/tags/:tag` - Filter by tag

**Hint:** Import controller functions and use them in routes

---

### Step 6: Comments Routes
**Todo:**
- [ ] Create `routes/comments.js`
- [ ] Nested routes under posts:
  - `GET /api/posts/:postId/comments` - All comments for post
  - `POST /api/posts/:postId/comments` - Add comment to post
  - `DELETE /api/posts/:postId/comments/:commentId` - Delete comment
- [ ] Validate postId exists before adding comment

**Hint:** Nested routes maintain relationship: comments always tied to a post

---

### Step 7: Add Features
**Todo:**
- [ ] Search posts by title/content
- [ ] Pagination (page, limit)
- [ ] Sort by date (newest/oldest)
- [ ] Post view count
- [ ] Related posts (by tags)
- [ ] Comment count per post

**Algorithm for Related Posts:**
```
Find posts that share at least one tag with current post
Sort by number of matching tags
Exclude current post
Return top 3
```

---

### Step 8: React Frontend
**Todo:**
- [ ] Blog home page (list of posts)
- [ ] Post detail page (full post + comments)
- [ ] Create post form (admin)
- [ ] Comment form
- [ ] Tag filter
- [ ] Search bar
- [ ] Pagination controls

**React Integration Example (Post List):**
```jsx
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';

function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchPosts = async () => {
      try {
        const response = await fetch('http://localhost:5000/api/posts/published');
        if (!response.ok) {
          throw new Error('Failed to fetch posts');
        }
        const data = await response.json();
        setPosts(data);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };

    fetchPosts();
  }, []);

  if (loading) return <div>Loading posts...</div>;

  return (
    <div>
      <h1>Blog</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>
            <Link to={`/post/${post.slug}`}>{post.title}</Link>
          </h2>
          <p>{post.excerpt}</p>
          <small>By {post.author} on {new Date(post.createdAt).toLocaleDateString()}</small>
        </article>
      ))}
    </div>
  );
}

export default PostList;
```

---

## 💡 Hints & Tips

### Generating Slugs:
```javascript
const slugify = require('slugify');
const slug = slugify(title, { lower: true, strict: true });
```

### Ensure Unique Slugs:
If slug exists, append number:
```
"my-post" → "my-post-1" → "my-post-2"
```

### Getting Comments for a Post:
```javascript
const postComments = comments.filter(c => c.postId === postId);
```

### MVC Pattern:
```
Request → Route → Controller → Model → Controller → Response
```

### React Post Preview:
Show excerpt in list view, full content in detail view

---

## 🎨 Enhancement Ideas

**Easy:**
- Reading time estimation
- Post categories
- Featured posts
- Draft/Published status toggle

**Medium:**
- Markdown support for content
- Comment replies (nested comments)
- Like/reaction system
- Author profiles

**Advanced:**
- Connect to MongoDB
- Image upload for featured images
- User authentication
- Admin dashboard
- Comment moderation

---

## ✅ Completion Checklist

**Backend:**
- [ ] MVC structure implemented
- [ ] All post CRUD operations work
- [ ] Comments system working
- [ ] Slug generation automatic
- [ ] Search functionality
- [ ] Pagination works
- [ ] Tag filtering works

**Frontend:**
- [ ] Post list displays
- [ ] Single post view works
- [ ] Comments display under post
- [ ] Can add new comment
- [ ] Tag filter works
- [ ] Search works
- [ ] Routing implemented

---

## 📚 Concepts You'll Practice

- MVC architecture
- Data relationships
- Nested routes
- Slug generation
- Filtering and searching
- Pagination
- React Router
- Form handling

---

## 🔗 Related Resources

**Chapters:**
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)
- [B03: MVC Architecture](../chapters/B03_MVC_ARCHITECTURE.md)

**External:**
- [Slugify Package](https://www.npmjs.com/package/slugify)
- [React Router](https://reactrouter.com/)

---

**Next:** [Project 6: URL Shortener](06-url-shortener.md)
