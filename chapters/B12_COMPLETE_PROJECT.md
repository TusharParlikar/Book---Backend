# B12: Complete Project - Social Media API 📱

> **Build a Full Social Media Backend: Users, Posts, Comments, Likes**

**Chapter Level:** Intermediate  
**Time Required:** 4-5 hours  
**Prerequisites:** Completed B01-B11  

---

## Project Overview

```
MODELS:
├─ User: name, email, password, avatar, followers, following
├─ Post: user, content, image, likes, comments, timestamps
├─ Comment: user, post, content, likes, timestamps
└─ Like: user, post/comment, timestamp

ROUTES:
├─ Auth: register, login, logout
├─ User: getProfile, updateProfile, follow/unfollow
├─ Post: createPost, getPosts, updatePost, deletePost
├─ Comment: addComment, deleteComment
└─ Like: likePost, unlikePost, likeComment, unlikeComment
```

---

## Implementation (Condensed)

```javascript
// ============================================================
// models/User.js, Post.js, Comment.js schemas (see B05-B06)
// ============================================================

// User Schema
{
  name, email, password, avatar,
  followers: [ObjectId], following: [ObjectId]
}

// Post Schema  
{
  user: ObjectId, content, image,
  likes: [ObjectId], comments: [ObjectId],
  timestamps
}

// Comment Schema
{
  user: ObjectId, post: ObjectId, content,
  likes: [ObjectId], timestamps
}


// ============================================================
// controllers/postController.js
// ============================================================

const Post = require('../models/Post');

const createPost = async (req, res) => {
  try {
    const post = await Post.create({
      user: req.user.userId,
      content: req.body.content,
      image: req.body.image
    });
    await post.populate('user', 'name avatar');
    res.status(201).json({ success: true, data: post });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

const getPosts = async (req, res) => {
  try {
    const posts = await Post.find()
      .populate('user', 'name avatar')
      .populate('comments')
      .sort({ createdAt: -1 });
    res.json({ success: true, data: posts });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const likePost = async (req, res) => {
  try {
    const post = await Post.findByIdAndUpdate(
      req.params.id,
      { $addToSet: { likes: req.user.userId } }, // Use $addToSet to prevent duplicates
      { new: true }
    );

    if (!post) {
      return res.status(404).json({ success: false, message: 'Post not found' });
    }
    
    res.json({ success: true, data: post });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const unlikePost = async (req, res) => {
  try {
    const post = await Post.findByIdAndUpdate(
      req.params.id,
      { $pull: { likes: req.user.userId } },
      { new: true }
    );

    if (!post) {
      return res.status(404).json({ success: false, message: 'Post not found' });
    }

    res.json({ success: true, data: post });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

module.exports = { createPost, getPosts, likePost, unlikePost };


// ============================================================
// controllers/commentController.js
// ============================================================

const Comment = require('../models/Comment');
const Post = require('../models/Post');

const addComment = async (req, res) => {
  try {
    const comment = await Comment.create({
      user: req.user.userId,
      post: req.params.postId,
      content: req.body.content
    });
    
    const post = await Post.findByIdAndUpdate(
      req.params.postId,
      { $push: { comments: comment._id } },
      { new: true }
    );
    
    res.status(201).json({ success: true, data: comment });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
};

const deleteComment = async (req, res) => {
  try {
    const comment = await Comment.findByIdAndDelete(req.params.id);
    await Post.findByIdAndUpdate(
      comment.post,
      { $pull: { comments: comment._id } }
    );
    res.json({ success: true, message: 'Deleted' });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

module.exports = { addComment, deleteComment };


// ============================================================
// controllers/userController.js
// ============================================================

const User = require('../models/User');

const getUserProfile = async (req, res) => {
  try {
    const user = await User.findById(req.params.userId)
      .populate('followers', 'name avatar')
      .populate('following', 'name avatar');
    res.json({ success: true, data: user });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const followUser = async (req, res) => {
  try {
    // Prevent user from following themselves
    if (req.user.userId === req.params.userId) {
      return res.status(400).json({ success: false, message: "You can't follow yourself" });
    }

    // Use Promise.all to run updates in parallel
    const [currentUser, userToFollow] = await Promise.all([
      User.findByIdAndUpdate(
        req.user.userId,
        { $addToSet: { following: req.params.userId } },
        { new: true }
      ),
      User.findByIdAndUpdate(
        req.params.userId,
        { $addToSet: { followers: req.user.userId } },
        { new: true }
      )
    ]);

    if (!userToFollow) {
      return res.status(404).json({ success: false, message: 'User to follow not found' });
    }
    
    res.json({ success: true, message: `Now following ${userToFollow.name}` });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const unfollowUser = async (req, res) => {
  try {
    const [currentUser, userToUnfollow] = await Promise.all([
      User.findByIdAndUpdate(
        req.user.userId,
        { $pull: { following: req.params.userId } },
        { new: true }
      ),
      User.findByIdAndUpdate(
        req.params.userId,
        { $pull: { followers: req.user.userId } },
        { new: true }
      )
    ]);

    if (!userToUnfollow) {
      return res.status(404).json({ success: false, message: 'User to unfollow not found' });
    }
    
    res.json({ success: true, message: `Unfollowed ${userToUnfollow.name}` });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

module.exports = { getUserProfile, followUser, unfollowUser };
```

---

## React Components

```javascript
// Feed, Post, Comment, UserProfile components...
// Use axios with Bearer token to access protected routes
```

---

## 🎯 Practice Projects

Build complete full-stack applications:

**Recommended Projects:**
- [Project #31: Social Media Backend](../projects/31-social-media-backend.md) - Posts, likes, comments ⭐⭐
- [Project #32: News Feed Algorithm](../projects/32-news-feed-algorithm.md) - Engagement ranking ⭐⭐
- [Project #33: Follow System](../projects/33-follow-system.md) - Social graph
- [Project #37: Real-Time Chat](../projects/37-realtime-chat.md) - Socket.io messaging

**Time Required:** 10-15 hours per project

---

| Previous | Index | Next |
|----------|-------|------|
| [B11: File Uploads](B11_FILE_UPLOADS.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B13: Docker & Containerization](B13_DOCKER_CONTAINERIZATION.md) |
