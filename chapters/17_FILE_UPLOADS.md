# Chapter 17: File Upload Implementation

## 17.1 Introduction to Cloudinary

Cloudinary is a cloud service for storing and managing files (images, videos).

### Setup Cloudinary

```bash
npm install cloudinary
```

```javascript
// config/cloudinary.js

const cloudinary = require('cloudinary').v2;

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET
});

module.exports = cloudinary;
```

## 17.2 Multer Middleware

Multer handles file uploads from React forms.

### Install Multer

```bash
npm install multer
```

### Configure Storage

```javascript
// middlewares/multer.js

const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'public/temp');
  },
  filename: (req, file, cb) => {
    cb(null, `${Date.now()}-${file.originalname}`);
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }
});

module.exports = upload;
```

## 17.3 File Upload to Cloudinary

```javascript
// utils/uploadOnCloudinary.js

const fs = require('fs').promises;
const cloudinary = require('../config/cloudinary');

const uploadOnCloudinary = async (localFilePath) => {
  try {
    if (!localFilePath) return null;

    // Upload to Cloudinary
    const response = await cloudinary.uploader.upload(localFilePath, {
      resource_type: 'auto'
    });

    // Delete temporary local file
    await fs.unlink(localFilePath);

    return response;
  } catch (error) {
    // Delete temp file on error
    await fs.unlink(localFilePath).catch(() => {});
    throw error;
  }
};

module.exports = uploadOnCloudinary;
```

---

## 🎯 Next: Chapter 18 - User Registration

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 16: HTTP Fundamentals](./16_HTTP_FUNDAMENTALS.md) | [📖 Course Index](../README.md) | [Chapter 18: User Registration →](./18_USER_REGISTRATION.md) |
