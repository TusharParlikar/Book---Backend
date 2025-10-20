# Project 20: Chat Messages API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B06 - Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md), [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Build a messaging API for one-on-one and group chats with read receipts and typing indicators.

**What You'll Learn:**
- Real-time messaging patterns
- Message threading
- Read receipts
- Typing indicators
- Message search

---

## 🎯 Project Goals

- ✅ One-on-one messaging
- ✅ Group chats
- ✅ Read receipts
- ✅ Message search
- ✅ React chat interface

---

## 🗺️ Roadmap

### Step 1: Models

**Conversations:**
- [ ] participants (array of user IDs)
- [ ] type (direct/group)
- [ ] lastMessage, lastMessageTime
- [ ] groupName (for groups), groupIcon

**Messages:**
- [ ] conversationId, senderId
- [ ] content, timestamp
- [ ] readBy (array of user IDs)
- [ ] type (text/image/file)

---

### Step 2: Conversation Management
**Todo:**
- [ ] `POST /api/conversations` - Start conversation
  - For direct: check if conversation exists between 2 users
  - For group: create with multiple participants
- [ ] `GET /api/conversations` - User's all conversations
- [ ] `GET /api/conversations/:id` - Single conversation details

---

### Step 3: Messaging
**Todo:**
- [ ] `POST /api/conversations/:id/messages` - Send message
  - Verify sender is participant
  - Add message to conversation
  - Update lastMessage fields
- [ ] `GET /api/conversations/:id/messages` - Get all messages
  - Pagination (load 50 at a time)
  - Newest first

---

### Step 4: Read Receipts
**Todo:**
- [ ] `PATCH /api/messages/:id/read`
  - Add current user to readBy array
  - Don't add sender (auto-read)
- [ ] Display double checkmark when all participants read

---

### Step 5: Search Messages
**Todo:**
- [ ] `GET /api/conversations/:id/search?q=hello`
- [ ] Search message content
- [ ] Return matching messages with context

---

### Step 6: Group Features
**Todo:**
- [ ] Add participant to group
- [ ] Remove participant
- [ ] Leave group
- [ ] Update group name/icon
- [ ] Admin system (group creator is admin)

---

### Step 7: React Frontend
**Todo:**
- [ ] Conversations list (sidebar)
- [ ] Chat window
- [ ] Message input
- [ ] Read receipts display
- [ ] Typing indicator (optional)
- [ ] Search in conversation
- [ ] Group management UI

---

## 💡 Hints & Tips

### Finding Existing Conversation:
```javascript
const conversation = conversations.find(c => 
  c.participants.includes(user1Id) && 
  c.participants.includes(user2Id) &&
  c.type === 'direct'
);
```

### Read Receipt Logic:
```javascript
if (!message.readBy.includes(userId)) {
  message.readBy.push(userId);
}
```

### Pagination:
```javascript
const messages = allMessages
  .slice(page * limit, (page + 1) * limit)
  .reverse(); // newest first
```

---

## 📚 Concepts

- Messaging systems
- Read receipts
- Group management
- Message threading
- Pagination

---

**Next:** [Project 21: Notification System API](21-notification-system.md)
