# B11: File Uploads & Cloud Storage ☁️

> **Handle Images: Upload, Store, Serve**

**Chapter Level:** Intermediate  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B10  

---

## Setup

```bash
npm install multer cloudinary
```

---

## Complete Implementation

```javascript
// ============================================================
// FILE: config/multer.js (Handle file upload)
// ============================================================

const multer = require('multer');
const path = require('path');

// STORAGE: How to store uploaded files temporarily
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname);
    cb(null, Date.now() + ext);
  }
});

// FILTER: Only allow images
const fileFilter = (req, file, cb) => {
  if (['image/jpeg', 'image/png', 'image/gif'].includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Only images allowed'));
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }  // 5MB max
});

module.exports = upload;


// ============================================================
// FILE: config/cloudinary.js (Cloud storage)
// ============================================================

const cloudinary = require('cloudinary').v2;

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_NAME,
  api_key: process.env.CLOUDINARY_KEY,
  api_secret: process.env.CLOUDINARY_SECRET
});

const uploadToCloudinary = async (filePath) => {
  try {
    const result = await cloudinary.uploader.upload(filePath);
    return {
      url: result.secure_url,
      publicId: result.public_id
    };
  } catch (error) {
    console.error('Cloudinary upload error:', error);
    throw new Error('Error uploading to Cloudinary');
  }
};

module.exports = { uploadToCloudinary };


// ============================================================
// FILE: controllers/uploadController.js
// ============================================================

const fs = require('fs');
const { uploadToCloudinary } = require('../config/cloudinary');

const uploadImage = async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ success: false, message: 'No file uploaded' });
    }

    // UPLOAD to Cloudinary
    const { url, publicId } = await uploadToCloudinary(req.file.path);

    // DELETE temporary file asynchronously
    fs.unlink(req.file.path, (err) => {
      if (err) {
        console.error('Failed to delete temporary file:', req.file.path, err);
      }
    });

    res.json({
      success: true,
      data: {
        url,
        publicId,
        filename: req.file.originalname
      }
    });

  } catch (error) {
    // If file exists, try to delete it on error
    if (req.file && req.file.path) {
      fs.unlink(req.file.path, (err) => {
        if (err) console.error('Failed to delete temporary file on error:', req.file.path, err);
      });
    }
    res.status(500).json({ success: false, error: 'File upload failed' });
  }
};

module.exports = { uploadImage };


// ============================================================
// FILE: routes/uploadRoutes.js
// ============================================================

const express = require('express');
const upload = require('../config/multer');
const uploadController = require('../controllers/uploadController');
const authMiddleware = require('../middleware/auth');

const router = express.Router();

router.post(
  '/image',
  authMiddleware,
  upload.single('image'),
  uploadController.uploadImage
);

module.exports = router;


// ============================================================
// FILE: models/Profile.js (Store image URL)
// ============================================================

const mongoose = require('mongoose');

const profileSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  avatar: String,
  bio: String,
  website: String
});

module.exports = mongoose.model('Profile', profileSchema);
```

---

## React Frontend

```javascript
// ============================================================
// FILE: src/components/ImageUpload.jsx
// ============================================================

import React, { useState } from 'react';
import axios from 'axios';

function ImageUpload() {
  const [file, setFile] = useState(null);
  const [preview, setPreview] = useState('');
  const [uploadedUrl, setUploadedUrl] = useState('');

  const handleFileSelect = (e) => {
    const selectedFile = e.target.files[0];
    setFile(selectedFile);
    setPreview(URL.createObjectURL(selectedFile));
  };

  const handleUpload = async () => {
    if (!file) {
      alert('Select a file first');
      return;
    }

    const formData = new FormData();
    formData.append('image', file);

    try {
      const token = localStorage.getItem('token');
      const response = await axios.post(
        'http://localhost:5000/api/upload/image',
        formData,
        {
          headers: {
            Authorization: `Bearer ${token}`,
            'Content-Type': 'multipart/form-data'
          }
        }
      );

      setUploadedUrl(response.data.data.url);
      alert('✅ Uploaded!');
    } catch (error) {
      alert('❌ Upload failed: ' + error.message);
    }
  };

  return (
    <div>
      <input type="file" accept="image/*" onChange={handleFileSelect} />
      {preview && <img src={preview} alt="Preview" style={{ maxWidth: '200px' }} />}
      <button onClick={handleUpload}>Upload</button>
      {uploadedUrl && (
        <div>
          <p>✅ Uploaded:</p>
          <img src={uploadedUrl} alt="Uploaded" style={{ maxWidth: '300px' }} />
        </div>
      )}
    </div>
  );
}

export default ImageUpload;
```

---

## 🎯 Practice Projects

Implement file uploads in these projects:

**Recommended Projects:**
- [Project #34: Image Gallery](../projects/34-image-gallery.md) - Cloudinary upload ⭐
- [Project #35: Profile Picture Upload](../projects/35-profile-picture-upload.md) - Avatar cropping
- [Project #36: Document Manager](../projects/36-document-manager.md) - File management system

**Time Required:** 5-7 hours per project

---

| Previous | Index | Next |
|----------|-------|------|
| [B10: E-Commerce Backend](B10_ECOMMERCE_BACKEND.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B12: Complete Project](B12_COMPLETE_PROJECT.md) |
