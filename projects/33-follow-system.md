# Project 33: Follow System API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B06 - Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md), [B12 - Social Media Project](../chapters/B12_COMPLETE_PROJECT.md)

---

## 📝 Description

Build a comprehensive follow/unfollow system with followers, following, and suggestions.

**What You'll Learn:**
- Many-to-many relationships
- Follow/unfollow logic
- Follower suggestions
- Mutual friends
- Social connections

---

## 🎯 Project Goals

- ✅ Follow and unfollow users
- ✅ View followers and following lists
- ✅ Suggest users to follow
- ✅ Mutual connections
- ✅ React follow UI

---

## 🗺️ Roadmap

### Step 1: Follow Model
**Todo:**
- [ ] Approach 1: Arrays in User model
  - followers: [userId1, userId2]
  - following: [userId1, userId2]
- [ ] Approach 2: Separate Follow model
  - followerId, followingId, createdAt

**Recommended:** Separate model for better querying

---

### Step 2: Follow User
**Todo:**
- [ ] `POST /api/users/:id/follow`
  - Create Follow record
  - Increment followersCount (target)
  - Increment followingCount (current user)
  - Send notification to target user
  - Return success

---

### Step 3: Unfollow User
**Todo:**
- [ ] `DELETE /api/users/:id/unfollow`
  - Remove Follow record
  - Decrement counts
  - Return success

---

### Step 4: Get Followers
**Todo:**
- [ ] `GET /api/users/:id/followers`
  - Find all users following this user
  - Return with profile info (name, avatar, bio)
  - Pagination (50 per page)

---

### Step 5: Get Following
**Todo:**
- [ ] `GET /api/users/:id/following`
  - Find all users this user follows
  - Return with profile info
  - Pagination

---

### Step 6: Check Follow Status
**Todo:**
- [ ] `GET /api/users/:id/follow-status`
  - Check if current user follows target
  - Check if target follows current user
  - Return: { isFollowing, isFollower, isMutual }

---

### Step 7: Mutual Connections
**Todo:**
- [ ] `GET /api/users/:id/mutual`
  - Find users both current user and target follow
  - Return mutual connections
  - Use for: "X mutual friends"

**Algorithm:**
```
userAFollowing ∩ userBFollowing = mutual
```

---

### Step 8: Suggest Users to Follow
**Todo:**
- [ ] `GET /api/users/suggestions`
  - Suggest based on:
    - Friends of friends
    - Similar interests
    - Popular users
    - Same location
  - Exclude: already following, current user
  - Return top 10

**Friends of Friends:**
```
Get all users you follow
Get all users THEY follow
Exclude users you already follow
Return most common
```

---

### Step 9: Remove Follower
**Todo:**
- [ ] `DELETE /api/users/followers/:id/remove`
  - Remove someone from your followers
  - They still follow you, but you block them
  - Soft delete or hard delete

---

### Step 10: React Frontend
**Todo:**
- [ ] Follow/Unfollow button
  - Show "Follow" if not following
  - Show "Following" if following
  - Click to toggle
- [ ] Followers list page
- [ ] Following list page
- [ ] Suggested users carousel
- [ ] Mutual friends display

---

## 💡 Hints & Tips

### Follow Logic:
```javascript
const follow = new Follow({
  follower: currentUserId,
  following: targetUserId
});
await follow.save();

await User.findByIdAndUpdate(targetUserId, { $inc: { followersCount: 1 } });
await User.findByIdAndUpdate(currentUserId, { $inc: { followingCount: 1 } });
```

### Check if Following:
```javascript
const isFollowing = await Follow.exists({
  follower: currentUserId,
  following: targetUserId
});
```

### Mutual Connections:
```javascript
const userAFollowing = await Follow.find({ follower: userA }).select('following');
const userBFollowing = await Follow.find({ follower: userB }).select('following');

const mutual = userAFollowing.filter(id => userBFollowing.includes(id));
```

### Suggestions (Friends of Friends):
```javascript
const following = await Follow.find({ follower: userId }).select('following');
const friendsOfFriends = await Follow.find({ 
  follower: { $in: following },
  following: { $ne: userId, $nin: following }
});
// Count frequency and return top
```

---

## 📚 Concepts

- Many-to-many relationships
- Social graphs
- Mutual connections
- Recommendation algorithms
- Set operations

---

**Next:** [Project 34: Image Gallery API](34-image-gallery.md)
