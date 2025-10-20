# Project 35: Profile Picture Upload

**Difficulty:** 🟡 Medium  
**Estimated Time:** 2-3 hours  
**Related Chapter:** [B11 - File Uploads](../chapters/B11_FILE_UPLOADS.md)

---

## 📝 Description

Implement profile picture upload with cropping, validation, and cloud storage.

**What You'll Learn:**
- Avatar upload
- Image cropping
- File validation
- CDN storage
- Previous image cleanup

---

## 🎯 Project Goals

- ✅ Upload profile picture
- ✅ Image validation and size limits
- ✅ Auto-crop to square
- ✅ Delete old avatar
- ✅ React avatar upload UI with preview

---

## 🗺️ Roadmap

### Step 1: Avatar Upload Route
**Todo:**
- [ ] `POST /api/users/avatar`
  - Accept single image file
  - Validate: jpg, png only
  - Max size: 2MB
  - Crop to square (400x400)
  - Upload to Cloudinary
  - Update user's avatar URL
  - Delete old avatar if exists

---

### Step 2: Image Validation
**Todo:**
- [ ] Check file type
- [ ] Check file size
- [ ] Check dimensions (min 200x200)
- [ ] Return errors if invalid

---

### Step 3: Image Processing
**Todo:**
- [ ] Crop to square (center crop)
- [ ] Resize to 400x400
- [ ] Optimize quality
- [ ] Convert to WebP (optional)

**Use:** Sharp package for processing
```javascript
const sharp = require('sharp');
await sharp(file.path)
  .resize(400, 400, { fit: 'cover' })
  .toFile(outputPath);
```

---

### Step 4: Delete Old Avatar
**Todo:**
- [ ] Before uploading new, delete old from Cloudinary
- [ ] Extract publicId from old URL
- [ ] Call cloudinary.uploader.destroy()

---

### Step 5: React Frontend
**Todo:**
- [ ] Avatar display (circular)
- [ ] Click to upload
- [ ] File input (hidden)
- [ ] Image preview before upload
- [ ] Crop interface (optional)
- [ ] Upload button

---

## 💡 Hints & Tips

### Multer for Avatar:
```javascript
const upload = multer({
  limits: { fileSize: 2 * 1024 * 1024 }, // 2MB
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png'];
    if (allowed.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Only JPG/PNG allowed'));
    }
  }
});

router.post('/avatar', upload.single('avatar'), uploadAvatar);
```

### Crop to Square:
```javascript
const result = await cloudinary.uploader.upload(file.path, {
  folder: 'avatars',
  width: 400,
  height: 400,
  crop: 'fill',
  gravity: 'face' // focus on face if detected
});
```

---

## 📚 Concepts

- Profile picture upload
- Image cropping
- File validation
- Cloud storage cleanup

---

**Next:** [Project 36: Document Manager API](36-document-manager.md)
