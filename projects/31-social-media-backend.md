# Project 31: Social Media Backend ⭐

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 8-10 hours  
**Related Chapter:** [B12 - Complete Social Media Project](../chapters/B12_COMPLETE_PROJECT.md)

---

## 📝 Description

Build a complete social media platform backend with posts, likes, comments, and user profiles.

**What You'll Learn:**
- Social graph modeling
- Post creation and feeds
- Like and comment systems
- User profiles
- Activity feeds

---

## 🎯 Project Goals

- ✅ User profiles with bio and avatar
- ✅ Create posts with text/images
- ✅ Like and comment on posts
- ✅ User timelines and feeds
- ✅ React social media UI

---

## 🗺️ Roadmap

### Step 1: User Profile Model
**Todo:**
- [ ] Extend User with:
  - username (unique)
  - bio, avatar, coverPhoto
  - location, website
  - followers array, following array
  - postsCount, followersCount, followingCount

---

### Step 2: Post Model
**Todo:**
- [ ] Post structure:
  - userId, content (text)
  - images array (URLs)
  - likes array (user IDs)
  - commentsCount, likesCount
  - createdAt, updatedAt

---

### Step 3: Create Post
**Todo:**
- [ ] `POST /api/posts`
  - Body: { content, images }
  - Validate: content not empty
  - Save post
  - Increment user's postsCount
  - Return created post

---

### Step 4: Get Posts (Feed)
**Todo:**
- [ ] `GET /api/posts/feed`
  - Return posts from:
    - Users you follow
    - Your own posts
  - Sort by: newest first
  - Pagination (20 per page)
  - Populate user info (name, avatar)

---

### Step 5: Like System
**Todo:**
- [ ] `POST /api/posts/:id/like`
  - Toggle like (like if not liked, unlike if already liked)
  - Add/remove userId from likes array
  - Update likesCount
  - Return updated post

---

### Step 6: Comment System
**Todo:**
- [ ] Comment Model:
  - postId, userId, content
  - createdAt
- [ ] `POST /api/posts/:id/comments`
  - Body: { content }
  - Save comment
  - Increment post's commentsCount
- [ ] `GET /api/posts/:id/comments`
  - Return all comments for post
  - Populate user info

---

### Step 7: Follow System
**Todo:**
- [ ] `POST /api/users/:id/follow`
  - Add target user to current user's following
  - Add current user to target's followers
  - Increment both counts
- [ ] `POST /api/users/:id/unfollow`
  - Remove from arrays
  - Decrement counts

---

### Step 8: User Profile Pages
**Todo:**
- [ ] `GET /api/users/:username`
  - Return profile info
  - Posts count, followers, following
  - Recent posts
- [ ] `GET /api/users/:username/posts`
  - All posts by user
  - Sort by newest

---

### Step 9: Explore/Discover
**Todo:**
- [ ] `GET /api/posts/explore`
  - Return popular posts (most likes/comments)
  - Exclude posts you've already seen
  - Trending posts

---

### Step 10: React Frontend
**Todo:**
- [ ] Home feed (posts from following)
- [ ] Create post form
- [ ] Post card (with like, comment buttons)
- [ ] Comment section
- [ ] User profile page
- [ ] Follow/unfollow button
- [ ] Explore page

---

## 💡 Hints & Tips

### Feed Query (SQL-like logic):
```javascript
const following = user.following; // array of user IDs
const feedPosts = posts.filter(p => 
  following.includes(p.userId) || p.userId === user.id
);
```

### Toggle Like:
```javascript
if (post.likes.includes(userId)) {
  // Unlike
  post.likes = post.likes.filter(id => id !== userId);
  post.likesCount--;
} else {
  // Like
  post.likes.push(userId);
  post.likesCount++;
}
```

### Follow/Unfollow:
```javascript
currentUser.following.push(targetUserId);
targetUser.followers.push(currentUserId);
currentUser.followingCount++;
targetUser.followersCount++;
```

---

## 📚 Concepts

- Social graph modeling
- Activity feeds
- Like/comment systems
- Follow relationships
- Timeline generation

---

**Next:** [Project 32: News Feed Algorithm](32-news-feed-algorithm.md)
