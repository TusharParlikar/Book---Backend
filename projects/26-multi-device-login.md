# Project 26: Multi-Device Login System

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B08 - JWT Authentication](../chapters/B08_JWT_AUTHENTICATION.md), [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Build a system that tracks user sessions across multiple devices with logout from all devices feature.

**What You'll Learn:**
- Session management
- Device tracking
- Refresh tokens
- Multiple sessions
- Security tokens

---

## 🎯 Project Goals

- ✅ Track all active sessions
- ✅ Show device information
- ✅ Logout from specific device
- ✅ Logout from all devices
- ✅ React active sessions UI

---

## 🗺️ Roadmap

### Step 1: Session Model
**Todo:**
- [ ] userId, refreshToken (hashed)
- [ ] deviceName, deviceType (mobile/desktop)
- [ ] browser, os, ipAddress
- [ ] lastActive, createdAt
- [ ] isActive (boolean)

---

### Step 2: Login with Session Tracking
**Todo:**
- [ ] `POST /api/auth/login`
  - Verify credentials
  - Generate access token + refresh token
  - Extract device info from User-Agent
  - Get IP address
  - Create session record
  - Return tokens

**Device Info Extraction:**
Use `ua-parser-js` package
```javascript
const UAParser = require('ua-parser-js');
const parser = new UAParser(req.headers['user-agent']);
const device = parser.getResult();
```

---

### Step 3: Refresh Token
**Todo:**
- [ ] `POST /api/auth/refresh`
  - Body: { refreshToken }
  - Find session by token (hashed)
  - Check: session active, not expired
  - Generate new access token
  - Update lastActive
  - Return new access token

---

### Step 4: Active Sessions List
**Todo:**
- [ ] `GET /api/auth/sessions`
  - Return user's all active sessions
  - Include: device, browser, location, last active
  - Mark current session

---

### Step 5: Logout Single Device
**Todo:**
- [ ] `DELETE /api/auth/sessions/:sessionId`
  - Mark session as inactive
  - Or delete session record
  - Invalidate refresh token

---

### Step 6: Logout All Devices
**Todo:**
- [ ] `POST /api/auth/logout-all`
  - Find all user's sessions
  - Mark all as inactive (or delete all)
  - Keep current session OR logout completely

---

### Step 7: Security Features
**Todo:**
- [ ] Detect suspicious login (different country)
- [ ] Email notification on new device login
- [ ] Limit: max 5 active sessions per user
- [ ] Auto-logout inactive sessions after 30 days

---

### Step 8: React Frontend
**Todo:**
- [ ] Active sessions page
  - List all devices
  - Show: device name, browser, location, last active
  - Highlight current session
  - Logout button for each session
- [ ] "Logout all other devices" button
- [ ] Security notifications

---

## 💡 Hints & Tips

### Getting IP Address:
```javascript
const ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
```

### Device Fingerprint:
Combine: User-Agent + IP + Screen resolution (from client)

### Session Expiry:
```javascript
const thirtyDaysAgo = new Date();
thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

// Find and delete old sessions
sessions.filter(s => s.lastActive < thirtyDaysAgo);
```

---

## 📚 Concepts

- Session management
- Device tracking
- Refresh tokens
- Token invalidation
- Security monitoring

---

**Next:** [Project 27: Product Recommendation Engine](27-product-recommendation.md)
