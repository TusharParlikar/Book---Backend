# Project 53: Social Media Platform with News Feed Algorithm ⭐⭐⭐

**Build Instagram/Twitter Clone with Intelligent Feed Ranking**

---

## 🎯 Project Overview

**Difficulty:** Expert ⭐⭐⭐⭐⭐  
**Time Required:** 50-55 hours  
**Skills Used:** B01-B20 (All chapters)

Build a complete social media platform with intelligent news feed, real-time updates, and engagement algorithms.

---

## 💡 What You'll Build

### Core Features:

1. **User Profiles**
   - Bio, avatar, cover photo
   - Follower/following counts
   - Post count and statistics
   - Profile verification badge

2. **Posts**
   - Text posts (280 characters)
   - Image posts (multiple images)
   - Video posts (upload to Cloudinary)
   - Link previews with Open Graph
   - Edit and delete posts

3. **Stories**
   - 24-hour temporary posts
   - View count tracking
   - Auto-delete after 24h
   - Story highlights

4. **News Feed Algorithm**
   - Engagement-based ranking
   - Time decay factor
   - Personalized content
   - Trending posts

5. **Follow System**
   - Follow/unfollow users
   - Followers list
   - Following list
   - Follow suggestions (mutual friends)

6. **Likes & Comments**
   - Like posts and comments
   - Nested comment threads
   - Real-time like counter
   - Edit and delete comments

7. **Notifications**
   - In-app notifications
   - Push notifications (Web Push API)
   - Email notifications
   - Notification preferences

8. **Direct Messaging**
   - One-on-one chat
   - Group chats
   - Typing indicators
   - Read receipts
   - Message reactions

9. **Hashtags & Mentions**
   - #hashtag support
   - @mention users
   - Trending hashtags
   - Hashtag following

10. **Content Moderation**
    - AI detects inappropriate content
    - Report/flag system
    - User blocking
    - Content filtering

11. **Analytics**
    - Post insights (reach, engagement)
    - Profile analytics
    - Best posting times
    - Follower demographics

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────┐
│           React SPA (TypeScript)                │
│  - Feed  - Profile  - Messages  - Stories       │
└────────────────┬────────────────────────────────┘
                 │
    ┌────────────▼──────────────────────┐
    │   Node.js API                     │
    │   - JWT Auth                      │
    │   - Rate Limiting                 │
    └────┬──────────┬──────────┬────────┘
         │          │          │
    ┌────▼───┐ ┌───▼────┐ ┌──▼───────┐
    │MongoDB │ │Redis   │ │Socket.io │
    │(Sharded│ │Pub/Sub │ │WebSockets│
    └────────┘ └────────┘ └──────────┘
         │
    ┌────▼──────────────────────┐
    │   CDN (CloudFlare)        │
    │   - Images/Videos         │
    │   - Static Assets         │
    └───────────────────────────┘
```

---

## 📋 Database Schema

### User Model
```javascript
{
  _id: ObjectId,
  username: String, // Unique
  email: String,
  password: String, // Hashed
  profile: {
    name: String,
    bio: String,
    avatar: String, // Cloudinary URL
    coverPhoto: String,
    website: String,
    location: String,
    verified: Boolean
  },
  stats: {
    followers: Number,
    following: Number,
    posts: Number
  },
  privacy: {
    isPrivate: Boolean,
    allowMessageFrom: 'everyone' | 'followers' | 'none'
  },
  createdAt: Date
}
```

### Post Model
```javascript
{
  _id: ObjectId,
  author: ObjectId, // User reference
  type: 'text' | 'image' | 'video',
  content: String,
  media: [{
    url: String,
    type: 'image' | 'video',
    width: Number,
    height: Number
  }],
  hashtags: [String],
  mentions: [ObjectId], // User references
  engagement: {
    likes: Number,
    comments: Number,
    shares: Number,
    views: Number
  },
  likedBy: [ObjectId], // For quick lookup
  createdAt: Date,
  updatedAt: Date
}
```

### Comment Model
```javascript
{
  _id: ObjectId,
  post: ObjectId,
  author: ObjectId,
  text: String,
  parentComment: ObjectId, // For nested comments
  likes: Number,
  likedBy: [ObjectId],
  createdAt: Date
}
```

### Follow Model
```javascript
{
  _id: ObjectId,
  follower: ObjectId, // User who follows
  following: ObjectId, // User being followed
  createdAt: Date
}
```

---

## 📊 News Feed Algorithm

### Engagement Score Formula

```javascript
function calculateEngagementScore(post) {
  const now = new Date();
  const postAge = (now - post.createdAt) / (1000 * 60 * 60); // Hours
  
  // Weights
  const LIKE_WEIGHT = 1;
  const COMMENT_WEIGHT = 2;
  const SHARE_WEIGHT = 3;
  
  // Engagement score
  const engagementScore = 
    (post.engagement.likes * LIKE_WEIGHT) +
    (post.engagement.comments * COMMENT_WEIGHT) +
    (post.engagement.shares * SHARE_WEIGHT);
  
  // Time decay (older posts get lower score)
  const timeDecay = Math.exp(-0.1 * postAge);
  
  // Final score
  return engagementScore * timeDecay;
}
```

### Feed Generation

```javascript
router.get('/feed', auth, async (req, res) => {
  const { page = 1, limit = 20 } = req.query;
  
  // Get users that current user follows
  const following = await Follow.find({ follower: req.user._id })
    .select('following');
  const followingIds = following.map(f => f.following);
  
  // Fetch posts from followed users
  const posts = await Post.find({
    author: { $in: followingIds }
  })
  .populate('author', 'username profile.avatar profile.name profile.verified')
  .sort({ createdAt: -1 })
  .limit(100); // Get more posts to rank
  
  // Calculate engagement score for each post
  const rankedPosts = posts.map(post => ({
    ...post.toObject(),
    score: calculateEngagementScore(post)
  }));
  
  // Sort by score
  rankedPosts.sort((a, b) => b.score - a.score);
  
  // Paginate
  const startIndex = (page - 1) * limit;
  const paginatedPosts = rankedPosts.slice(startIndex, startIndex + limit);
  
  res.json({
    posts: paginatedPosts,
    hasMore: rankedPosts.length > startIndex + limit
  });
});
```

---

## 🔥 Real-Time Features

### Live Likes with Socket.io

```javascript
// Server-side
io.on('connection', (socket) => {
  // User likes a post
  socket.on('like-post', async ({ postId }) => {
    const post = await Post.findById(postId);
    post.engagement.likes += 1;
    post.likedBy.push(socket.user._id);
    await post.save();
    
    // Broadcast to all users viewing this post
    io.to(`post:${postId}`).emit('post-liked', {
      postId,
      likes: post.engagement.likes,
      likedBy: socket.user.username
    });
  });
  
  // User views a post
  socket.on('view-post', ({ postId }) => {
    socket.join(`post:${postId}`);
  });
});
```

### Typing Indicators in Chat

```javascript
socket.on('typing-start', ({ conversationId }) => {
  socket.to(conversationId).emit('user-typing', {
    userId: socket.user._id,
    username: socket.user.username
  });
});

socket.on('typing-stop', ({ conversationId }) => {
  socket.to(conversationId).emit('user-stopped-typing', {
    userId: socket.user._id
  });
});
```

---

## 📱 React Frontend Components

### Feed Component
```jsx
function Feed() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const observerRef = useRef();
  
  // Infinite scroll
  useEffect(() => {
    const observer = new IntersectionObserver(
      entries => {
        if (entries[0].isIntersecting && !loading) {
          loadMorePosts();
        }
      },
      { threshold: 1.0 }
    );
    
    if (observerRef.current) {
      observer.observe(observerRef.current);
    }
    
    return () => observer.disconnect();
  }, [loading]);
  
  const loadMorePosts = async () => {
    setLoading(true);
    const response = await fetch('/api/feed?page=' + nextPage);
    const data = await response.json();
    setPosts(prev => [...prev, ...data.posts]);
    setLoading(false);
  };
  
  return (
    <div className="feed">
      {posts.map(post => (
        <PostCard key={post._id} post={post} />
      ))}
      <div ref={observerRef} />
      {loading && <Spinner />}
    </div>
  );
}
```

### Post Card Component
```jsx
function PostCard({ post }) {
  const [liked, setLiked] = useState(post.isLikedByCurrentUser);
  const [likeCount, setLikeCount] = useState(post.engagement.likes);
  const socket = useSocket();
  
  useEffect(() => {
    socket.emit('view-post', { postId: post._id });
    
    socket.on('post-liked', (data) => {
      if (data.postId === post._id) {
        setLikeCount(data.likes);
      }
    });
    
    return () => socket.off('post-liked');
  }, [post._id]);
  
  const handleLike = async () => {
    setLiked(!liked);
    setLikeCount(prev => liked ? prev - 1 : prev + 1);
    
    socket.emit('like-post', { postId: post._id });
  };
  
  return (
    <div className="post-card">
      <PostHeader author={post.author} timestamp={post.createdAt} />
      <PostContent content={post.content} media={post.media} />
      <PostActions
        liked={liked}
        likeCount={likeCount}
        commentCount={post.engagement.comments}
        onLike={handleLike}
        onComment={() => setShowComments(true)}
      />
    </div>
  );
}
```

---

## 🤖 Content Moderation with AI

```javascript
const openai = require('../utils/openai');

async function moderateContent(text) {
  const response = await openai.moderations.create({
    input: text
  });
  
  const results = response.results[0];
  
  return {
    flagged: results.flagged,
    categories: results.categories,
    scores: results.category_scores
  };
}

// Before creating post
router.post('/posts', auth, async (req, res) => {
  const { content } = req.body;
  
  // Moderate content
  const moderation = await moderateContent(content);
  
  if (moderation.flagged) {
    return res.status(400).json({
      message: 'Content violates community guidelines',
      categories: moderation.categories
    });
  }
  
  // Create post...
});
```

---

## 📊 Analytics Dashboard

### Post Insights
```javascript
router.get('/posts/:id/insights', auth, async (req, res) => {
  const post = await Post.findById(req.params.id);
  
  if (post.author.toString() !== req.user._id.toString()) {
    return res.status(403).json({ message: 'Not authorized' });
  }
  
  // Get hourly views
  const hourlyViews = await PostView.aggregate([
    { $match: { post: post._id } },
    {
      $group: {
        _id: { $hour: '$createdAt' },
        count: { $sum: 1 }
      }
    },
    { $sort: { '_id': 1 } }
  ]);
  
  // Get demographics
  const viewerDemographics = await PostView.aggregate([
    { $match: { post: post._id } },
    {
      $lookup: {
        from: 'users',
        localField: 'user',
        foreignField: '_id',
        as: 'viewer'
      }
    },
    { $unwind: '$viewer' },
    {
      $group: {
        _id: '$viewer.profile.location',
        count: { $sum: 1 }
      }
    }
  ]);
  
  res.json({
    views: post.engagement.views,
    likes: post.engagement.likes,
    comments: post.engagement.comments,
    shares: post.engagement.shares,
    engagementRate: ((post.engagement.likes + post.engagement.comments) / post.engagement.views * 100).toFixed(2),
    hourlyViews,
    viewerDemographics
  });
});
```

---

## 🚀 Scaling Considerations

### MongoDB Sharding
```javascript
// Shard by user_id for balanced distribution
sh.enableSharding("socialDB")
sh.shardCollection("socialDB.posts", { "author": 1 })
```

### Redis Caching
```javascript
// Cache trending posts
async function getTrendingPosts() {
  const cached = await redis.get('trending:posts');
  if (cached) return JSON.parse(cached);
  
  const posts = await Post.find()
    .sort({ 'engagement.likes': -1, 'engagement.comments': -1 })
    .limit(10);
  
  await redis.setex('trending:posts', 300, JSON.stringify(posts)); // 5 min cache
  
  return posts;
}
```

---

## ✅ Deliverables

- [ ] User authentication with JWT
- [ ] Profile management
- [ ] Post creation (text, image, video)
- [ ] News feed with ranking algorithm
- [ ] Follow system
- [ ] Like and comment functionality
- [ ] Real-time updates with Socket.io
- [ ] Direct messaging
- [ ] Hashtags and mentions
- [ ] Content moderation with AI
- [ ] Analytics dashboard
- [ ] Stories feature
- [ ] Notifications system
- [ ] Mobile responsive UI
- [ ] Docker deployment

---

**Related Chapters:** B01-B20, B11 (File Uploads), B17 (AI), B20 (Real-time)

**Made with ❤️ by Tushar Parlikar**

[← Back: E-Commerce Platform](52-ai-ecommerce-platform.md) | [Next: Collaboration Platform →](54-collaboration-platform.md)
