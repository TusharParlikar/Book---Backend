# Project 24: Password Reset System

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3 hours  
**Related Chapter:** [B07 - Password Security](../chapters/B07_PASSWORD_SECURITY.md), [B09 - Complete Auth Flow](../chapters/B09_COMPLETE_AUTH_FLOW.md)

---

## 📝 Description

Implement a secure password reset flow with email tokens and expiry.

**What You'll Learn:**
- Password reset tokens
- Secure token generation
- Email verification
- Token expiry
- Security best practices

---

## 🎯 Project Goals

- ✅ Request password reset via email
- ✅ Send reset link with token
- ✅ Verify token and reset password
- ✅ Token expiry (15 minutes)
- ✅ React password reset UI

---

## 🗺️ Roadmap

### Step 1: Reset Token Model
**Todo:**
- [ ] userId, token (random string)
- [ ] expiresAt (Date)
- [ ] used (boolean)
- [ ] createdAt

---

### Step 2: Request Reset
**Todo:**
- [ ] `POST /api/auth/forgot-password`
  - Body: { email }
- [ ] Find user by email
- [ ] Generate random token (crypto)
- [ ] Set expiry: now + 15 minutes
- [ ] Save token to database
- [ ] Send email with reset link

**Token Generation:**
```javascript
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');
```

**Reset Link:**
```
http://frontend.com/reset-password?token=GENERATED_TOKEN
```

---

### Step 3: Send Email
**Todo:**
- [ ] Use Nodemailer
- [ ] Email template with reset button
- [ ] Include: reset link, expiry time
- [ ] Handle email sending errors

---

### Step 4: Verify Token
**Todo:**
- [ ] `POST /api/auth/verify-reset-token`
  - Body: { token }
- [ ] Find token in database
- [ ] Check: exists, not used, not expired
- [ ] Return: valid (boolean)

**Verification:**
```
1. Find token
2. Check used === false
3. Check expiresAt > now
4. Return result
```

---

### Step 5: Reset Password
**Todo:**
- [ ] `POST /api/auth/reset-password`
  - Body: { token, newPassword }
- [ ] Verify token (as above)
- [ ] Validate new password strength
- [ ] Hash new password (bcrypt)
- [ ] Update user's password
- [ ] Mark token as used
- [ ] Send confirmation email

---

### Step 6: Security Measures
**Todo:**
- [ ] Rate limit reset requests (3 per hour per email)
- [ ] Invalidate all old tokens on successful reset
- [ ] Log all reset attempts
- [ ] Don't reveal if email exists (security)

**Response (whether email exists or not):**
```
"If email exists, reset link sent"
```

---

### Step 7: React Frontend
**Todo:**
- [ ] Forgot password page
  - Email input
  - Submit button
  - Success message
- [ ] Reset password page
  - Extract token from URL
  - New password input
  - Confirm password input
  - Submit button
  - Validation feedback

---

## 💡 Hints & Tips

### Secure Token:
```javascript
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');
const hashedToken = crypto.createHash('sha256').update(token).digest('hex');
// Store hashedToken in DB, send original token in email
```

### Check Expiry:
```javascript
if (new Date() > new Date(resetToken.expiresAt)) {
  return res.status(400).json({ error: 'Token expired' });
}
```

### Password Validation:
- Min 8 characters
- At least 1 uppercase, 1 lowercase, 1 number

---

## 📚 Concepts

- Token generation
- Email verification
- Security tokens
- Expiry management
- Password hashing

---

**Next:** [Project 25: Email Verification System](25-email-verification.md)
