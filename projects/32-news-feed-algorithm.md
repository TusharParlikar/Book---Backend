# Project 32: News Feed Algorithm ⭐

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 6-8 hours  
**Related Chapter:** [B12 - Complete Social Media Project](../chapters/B12_COMPLETE_PROJECT.md)

---

## 📝 Description

Build an intelligent news feed ranking algorithm like Facebook/Instagram using engagement signals.

**What You'll Learn:**
- Feed ranking algorithms
- Engagement scoring
- Relevance calculations
- Personalized feeds
- Feed optimization

---

## 🎯 Project Goals

- ✅ Rank posts by relevance (not just chronological)
- ✅ Factor in: likes, comments, recency, engagement
- ✅ Personalize based on user interests
- ✅ Prevent spam/low-quality posts
- ✅ React feed with ranked posts

---

## 🗺️ Roadmap

### Step 1: Engagement Metrics
**Todo:**
- [ ] Track post metrics:
  - viewsCount
  - likesCount
  - commentsCount
  - sharesCount
  - clicksCount
- [ ] Track user engagement with posts

---

### Step 2: Engagement Score Formula
**Todo:**
- [ ] Calculate engagement score per post
- [ ] Formula:
  ```
  score = (likes * 1) + 
          (comments * 3) + 
          (shares * 5) - 
          (age_penalty)
  ```
- [ ] Age penalty: older posts rank lower

**Age Penalty:**
```javascript
const ageInHours = (Date.now() - post.createdAt) / (1000 * 60 * 60);
const agePenalty = ageInHours * 0.5; // decrease score by 0.5 per hour
```

---

### Step 3: Relevance Score
**Todo:**
- [ ] Calculate how relevant post is to user
- [ ] Factors:
  - Is author someone user follows? (+10 points)
  - Has user interacted with similar posts? (+5 points)
  - Is post in user's favorite category? (+3 points)
  - Has user's friends engaged? (+2 per friend)

---

### Step 4: Combined Ranking
**Todo:**
- [ ] Final Score = Engagement Score + Relevance Score
- [ ] `GET /api/feed/ranked`
  - Get all potential posts
  - Calculate scores for each
  - Sort by final score (descending)
  - Return top 50

**Algorithm:**
```javascript
function rankFeed(userId) {
  const posts = getAllEligiblePosts(userId);
  
  posts.forEach(post => {
    const engagement = calculateEngagement(post);
    const relevance = calculateRelevance(post, userId);
    post.score = engagement + relevance;
  });
  
  return posts.sort((a, b) => b.score - a.score).slice(0, 50);
}
```

---

### Step 5: Diversity
**Todo:**
- [ ] Don't show 10 posts from same author in a row
- [ ] Mix content types (text, image, video)
- [ ] Inject trending posts from non-followed users
- [ ] Show new creators (discovery)

**Diversity Logic:**
```
Every 5 posts from following, insert 1 from explore
Group by author, show max 3 consecutive posts per author
```

---

### Step 6: User Interest Modeling
**Todo:**
- [ ] Track: posts user liked, commented, viewed
- [ ] Extract topics/categories
- [ ] Build user interest profile
- [ ] Boost posts matching interests

**Interest Profile:**
```javascript
userInterests = {
  sports: 20,
  technology: 35,
  food: 15,
  // ... based on engagement
}
```

---

### Step 7: Freshness Boost
**Todo:**
- [ ] Boost recent posts (< 1 hour old)
- [ ] "New" badge on very recent posts
- [ ] Prevent old posts from appearing

**Freshness Multiplier:**
```javascript
if (ageInMinutes < 60) {
  score *= 1.5; // 50% boost
}
```

---

### Step 8: Spam Prevention
**Todo:**
- [ ] Detect and penalize:
  - Posts with excessive hashtags (> 10)
  - Posts from users with low engagement history
  - Duplicate content
  - Clickbait titles

---

### Step 9: A/B Testing
**Todo:**
- [ ] Randomly assign users to algorithm variants
- [ ] Track: time spent, engagement rate
- [ ] Compare which algorithm performs better

---

### Step 10: React Frontend
**Todo:**
- [ ] Infinite scroll feed
- [ ] Pull-to-refresh
- [ ] Show "New posts" button when new content available
- [ ] Engagement indicators (trending, popular)
- [ ] Personalized "For You" vs "Following" tabs

---

## 💡 Hints & Tips

### Engagement Score:
```javascript
function calculateEngagement(post) {
  const likes = post.likesCount * 1;
  const comments = post.commentsCount * 3;
  const shares = post.sharesCount * 5;
  const ageInHours = (Date.now() - post.createdAt) / (1000 * 60 * 60);
  const agePenalty = ageInHours * 0.5;
  
  return likes + comments + shares - agePenalty;
}
```

### Relevance Score:
```javascript
function calculateRelevance(post, userId) {
  let score = 0;
  
  if (user.following.includes(post.userId)) score += 10;
  if (post.category in user.interests) score += user.interests[post.category];
  
  const friendEngagements = post.likes.filter(id => 
    user.friends.includes(id)
  ).length;
  score += friendEngagements * 2;
  
  return score;
}
```

### Cache Feed:
```javascript
// Regenerate feed every 5 minutes, not on every request
if (!cache[userId] || cache[userId].age > 5 * 60 * 1000) {
  cache[userId] = { feed: generateFeed(userId), timestamp: Date.now() };
}
```

---

## 📚 Concepts

- Feed ranking algorithms
- Engagement scoring
- Relevance calculations
- Personalization
- Spam detection
- A/B testing

---

## 🎨 Enhancement Ideas

- Machine learning for personalization
- Click-through rate prediction
- User retention optimization
- Content diversity algorithms

---

**Next:** [Project 33: Follow System API](33-follow-system.md)
