# Project 25: Email Verification System

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3 hours  
**Related Chapter:** [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Implement email verification for new user registrations with verification tokens.

**What You'll Learn:**
- Email verification flow
- Verification tokens
- Account activation
- Resend verification
- Welcome emails

---

## 🎯 Project Goals

- ✅ Send verification email on signup
- ✅ Verify email with token
- ✅ Resend verification link
- ✅ Restrict unverified users
- ✅ React verification UI

---

## 🗺️ Roadmap

### Step 1: User Model Update
**Todo:**
- [ ] Add fields to User:
  - isEmailVerified (boolean, default false)
  - emailVerificationToken (string)
  - emailVerificationExpiry (Date)

---

### Step 2: Registration with Verification
**Todo:**
- [ ] `POST /api/auth/register`
  - Accept: name, email, password
  - Hash password
  - Set isEmailVerified = false
  - Generate verification token
  - Set expiry: 24 hours
  - Save user
  - Send verification email
  - Return success (don't auto-login)

**Verification Link:**
```
http://frontend.com/verify-email?token=TOKEN
```

---

### Step 3: Verification Email
**Todo:**
- [ ] Create email template
- [ ] Include: verification button/link
- [ ] Friendly message
- [ ] Mention expiry time
- [ ] Send with Nodemailer

---

### Step 4: Verify Email
**Todo:**
- [ ] `GET /api/auth/verify-email/:token`
  - Find user by token
  - Check: token valid, not expired
  - Set isEmailVerified = true
  - Clear token and expiry
  - Return success
  - Redirect to login page

---

### Step 5: Resend Verification
**Todo:**
- [ ] `POST /api/auth/resend-verification`
  - Body: { email }
  - Find user by email
  - Check: already verified? (return message)
  - Generate new token
  - Send email again
  - Rate limit: 3 per hour

---

### Step 6: Protect Routes
**Todo:**
- [ ] Middleware: `requireVerifiedEmail`
  - Check if req.user.isEmailVerified === true
  - If not, return 403 Forbidden
- [ ] Apply to protected routes
- [ ] Allow: login, verify email, resend

---

### Step 7: React Frontend
**Todo:**
- [ ] After registration: show "Check your email" message
- [ ] Email verification page
  - Extract token from URL
  - Auto-verify on page load
  - Show success/error
- [ ] Resend verification button
- [ ] Banner for unverified users

---

## 💡 Hints & Tips

### Generate Token:
```javascript
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');
```

### Check Verification Status:
```javascript
if (!req.user.isEmailVerified) {
  return res.status(403).json({ 
    error: 'Please verify your email first' 
  });
}
```

### Prevent Multiple Verifications:
```javascript
if (user.isEmailVerified) {
  return res.status(400).json({ error: 'Email already verified' });
}
```

---

## 📚 Concepts

- Email verification
- Account activation
- Token validation
- Route protection
- User onboarding

---

**Next:** [Project 26: Multi-Device Login System](26-multi-device-login.md)
