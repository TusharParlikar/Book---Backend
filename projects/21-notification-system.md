# Project 21: Notification System API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Build a notification center for in-app notifications with read/unread status and categories.

**What You'll Learn:**
- Notification patterns
- Real-time updates (polling/websockets)
- Notification categories
- Badge counts
- Notification preferences

---

## 🎯 Project Goals

- ✅ Create system and user notifications
- ✅ Mark as read/unread
- ✅ Categorize notifications
- ✅ Notification preferences
- ✅ React notification dropdown

---

## 🗺️ Roadmap

### Step 1: Notification Model
**Todo:**
- [ ] userId, type, title, message
- [ ] read (boolean), readAt
- [ ] category (like, comment, follow, system)
- [ ] link (where to navigate)
- [ ] createdAt

---

### Step 2: Create Notifications
**Todo:**
- [ ] `POST /api/notifications` - Create notification
  - Usually triggered by other actions (new comment, new follower)
- [ ] `GET /api/notifications` - Get user's notifications
  - Sort by newest first
- [ ] `GET /api/notifications/unread` - Only unread

---

### Step 3: Mark as Read
**Todo:**
- [ ] `PATCH /api/notifications/:id/read` - Mark single as read
- [ ] `PATCH /api/notifications/read-all` - Mark all as read
- [ ] Update read: true, readAt: now

---

### Step 4: Badge Count
**Todo:**
- [ ] `GET /api/notifications/count` - Unread count
  - Return number for badge display
- [ ] Real-time update (polling every 30s or WebSocket)

---

### Step 5: Categories & Filtering
**Todo:**
- [ ] `GET /api/notifications?category=comment`
- [ ] Different icons for each category
- [ ] Priority levels (high, normal, low)

---

### Step 6: Preferences
**Todo:**
- [ ] User can mute certain notification types
- [ ] Email notification toggle
- [ ] `PUT /api/notifications/preferences`
  - Body: { likes: true, comments: true, follows: false }

---

### Step 7: React Frontend
**Todo:**
- [ ] Bell icon with badge (unread count)
- [ ] Dropdown showing recent notifications
- [ ] Mark as read on click
- [ ] "Mark all as read" button
- [ ] Notification preferences page

---

## 💡 Hints & Tips

### Unread Count:
```javascript
const unreadCount = notifications.filter(n => !n.read).length;
```

### Auto-Read on View:
When notification dropdown opens, mark all as read

### Polling:
```javascript
// React useEffect
setInterval(() => {
  fetchNotifications();
}, 30000); // every 30s
```

---

## 📚 Concepts

- Notification systems
- Badge counts
- Read/unread logic
- User preferences
- Real-time polling

---

**Next:** [Project 22: Rate Limiter API](22-rate-limiter.md)
