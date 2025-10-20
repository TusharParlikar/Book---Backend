# Project 37: Real-Time Chat

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 6-8 hours  
**Related Chapter:** [B12 - Complete Project](../chapters/B12_COMPLETE_PROJECT.md)

---

## 📝 Description

Build real-time chat with WebSockets, typing indicators, and read receipts.

**What You'll Learn:**
- Socket.io integration
- Real-time events
- Online status
- Typing indicators
- Message delivery

---

## 🎯 Project Goals

- ✅ Real-time messaging
- ✅ Online/offline status
- ✅ Typing indicators
- ✅ Read receipts
- ✅ React chat UI

---

## 🗺️ Roadmap

### Step 1: Setup Socket.io
**Todo:**
- [ ] Install socket.io
- [ ] Setup server
- [ ] Connect from React
- [ ] Handle connections

### Step 2: Send Message
**Todo:**
- [ ] Emit: 'send_message' event
- [ ] Broadcast to conversation participants
- [ ] Save to database

### Step 3: Typing Indicator
**Todo:**
- [ ] Emit: 'typing' when user types
- [ ] Emit: 'stop_typing' when stopped
- [ ] Show "User is typing..." in UI

### Step 4: Online Status
**Todo:**
- [ ] Track connected users
- [ ] Emit 'user_online' on connect
- [ ] Emit 'user_offline' on disconnect
- [ ] Display green dot for online

### Step 5: React Frontend
**Todo:**
- [ ] Socket connection
- [ ] Real-time message display
- [ ] Typing indicator
- [ ] Online status badges

---

**Next:** [Project 38: Search Engine API](38-search-engine.md)
