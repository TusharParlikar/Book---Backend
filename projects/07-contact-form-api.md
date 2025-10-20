# Project 7: Contact Form API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 2-3 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build a contact form backend with email sending capability. Learn about email services, input sanitization, and spam prevention.

**What You'll Learn:**
- Email sending with Nodemailer
- Input sanitization
- Rate limiting
- CAPTCHA integration
- Email templates

---

## 🎯 Project Goals

- ✅ Accept contact form submissions
- ✅ Send emails via Nodemailer
- ✅ Validate and sanitize input
- ✅ Prevent spam with rate limiting
- ✅ React contact form with validation

---

## 🗺️ Roadmap

### Step 1: Setup Email Service
**Todo:**
- [ ] Install nodemailer
- [ ] Choose email provider (Gmail, SendGrid, Mailtrap)
- [ ] Get SMTP credentials
- [ ] Store in .env file
- [ ] Test email sending

**Email Providers:**
- **Gmail:** Free, 500 emails/day limit
- **SendGrid:** Free tier, 100 emails/day
- **Mailtrap:** Testing only (dev environment)

---

### Step 2: Create Email Configuration
**Todo:**
- [ ] Create `config/emailConfig.js`
- [ ] Setup Nodemailer transporter
- [ ] Configure SMTP settings
- [ ] Test connection

**Hint:** For Gmail, enable "Less secure app access" or use App Password

---

### Step 3: Create Contact Route
**Todo:**
- [ ] `POST /api/contact` endpoint
- [ ] Accept body:
  - name (required)
  - email (required, valid format)
  - subject (required)
  - message (required, min 10 characters)
- [ ] Validate all fields
- [ ] Return success/error response

---

### Step 4: Input Validation & Sanitization
**Todo:**
- [ ] Install `validator` package
- [ ] Validate email format
- [ ] Sanitize inputs (remove harmful characters)
- [ ] Check required fields
- [ ] Limit message length (max 1000 characters)
- [ ] Prevent XSS attacks

**Hint:** Use `validator.isEmail()` and `validator.escape()`

---

### Step 5: Send Email
**Todo:**
- [ ] Create email template
- [ ] Include sender's info
- [ ] Format message nicely
- [ ] Send to your email
- [ ] Handle send errors
- [ ] Return appropriate response

**Email Template Structure:**
```
From: [Name] <[Email]>
Subject: [Subject]

Message:
[Message Content]

---
Sent from Contact Form
```

---

### Step 6: Rate Limiting
**Todo:**
- [ ] Install `express-rate-limit`
- [ ] Limit: 3 submissions per 15 minutes per IP
- [ ] Return 429 Too Many Requests if exceeded
- [ ] Custom error message

**Why?** Prevent spam and abuse

---

### Step 7: Store Submissions (Optional)
**Todo:**
- [ ] Save all submissions to array/file
- [ ] Include timestamp
- [ ] IP address (for tracking)
- [ ] Status (sent/failed)

**Benefit:** Keep record of all contact attempts

---

### Step 8: React Frontend
**Todo:**
- [ ] Create contact form with fields
- [ ] Client-side validation
- [ ] Show loading state when submitting
- [ ] Success message on send
- [ ] Error handling
- [ ] Reset form after success
- [ ] Disable submit button during send

**Form Fields:**
- Name input
- Email input
- Subject input
- Message textarea
- Submit button

---

## 💡 Hints & Tips

### Nodemailer Setup (Gmail):
```javascript
const transporter = nodemailer.createTransporter({
  service: 'gmail',
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS
  }
});
```

### Email Validation:
```javascript
const validator = require('validator');
if (!validator.isEmail(email)) {
  return res.status(400).json({ error: 'Invalid email' });
}
```

### Rate Limiting:
```javascript
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 3 // limit each IP to 3 requests per windowMs
});
app.use('/api/contact', limiter);
```

### React Form Validation:
Validate before sending, show inline errors

---

## 🎨 Enhancement Ideas

**Easy:**
- Auto-reply email to sender
- Copy email to sender
- Phone number field
- Company name field

**Medium:**
- File attachment support
- Email templates (HTML)
- Admin notification dashboard
- CAPTCHA integration

**Advanced:**
- Queue system for emails
- Email tracking (read receipts)
- Multi-language support
- Custom form builder

---

## ✅ Completion Checklist

**Backend:**
- [ ] Email sending works
- [ ] All validations in place
- [ ] Rate limiting prevents spam
- [ ] Error handling complete
- [ ] Submissions logged
- [ ] Environment variables secure

**Frontend:**
- [ ] Form displays correctly
- [ ] Client-side validation
- [ ] Submit sends request
- [ ] Loading state shows
- [ ] Success message displays
- [ ] Form resets after success
- [ ] Errors shown to user

---

## 📚 Concepts You'll Practice

- Email sending
- Nodemailer
- Input validation
- Sanitization
- Rate limiting
- Environment variables
- React form handling
- Error handling

---

## 🔗 Related Resources

**Chapters:**
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

**External:**
- [Nodemailer Docs](https://nodemailer.com/)
- [Validator.js](https://www.npmjs.com/package/validator)
- [Express Rate Limit](https://www.npmjs.com/package/express-rate-limit)

---

**Next:** [Project 8: Note Taking App API](08-note-taking-app.md)
