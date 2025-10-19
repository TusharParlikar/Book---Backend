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
    const post = await Post.findById(req.params.id);
    if (!post.likes.includes(req.user.userId)) {
      post.likes.push(req.user.userId);
      await post.save();
    }
    res.json({ success: true, data: post });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const unlikePost = async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    post.likes = post.likes.filter(id => !id.equals(req.user.userId));
    await post.save();
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
    const userToFollow = await User.findById(req.params.userId);
    const currentUser = await User.findById(req.user.userId);
    
    if (!currentUser.following.includes(userToFollow._id)) {
      currentUser.following.push(userToFollow._id);
      userToFollow.followers.push(currentUser._id);
      await currentUser.save();
      await userToFollow.save();
    }
    
    res.json({ success: true, message: 'Followed' });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

const unfollowUser = async (req, res) => {
  try {
    const userToUnfollow = await User.findById(req.params.userId);
    const currentUser = await User.findById(req.user.userId);
    
    currentUser.following = currentUser.following.filter(id => !id.equals(userToUnfollow._id));
    userToUnfollow.followers = userToUnfollow.followers.filter(id => !id.equals(currentUser._id));
    
    await currentUser.save();
    await userToUnfollow.save();
    
    res.json({ success: true, message: 'Unfollowed' });
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

| Previous | Index | Next |
|----------|-------|------|
| [B11: File Uploads](B11_FILE_UPLOADS.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B13: Docker & Containerization](B13_DOCKER_CONTAINERIZATION.md) |
