# Project 34: Image Gallery API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B11 - File Uploads](../chapters/B11_FILE_UPLOADS.md)

---

## 📝 Description

Build an image hosting and gallery system with upload, storage, and organization features.

**What You'll Learn:**
- Image upload with Multer
- Cloud storage (Cloudinary)
- Image optimization
- Album/collection management
- Image metadata

---

## 🎯 Project Goals

- ✅ Upload images to cloud
- ✅ Organize in albums
- ✅ Image thumbnails and optimization
- ✅ Search by tags
- ✅ React image gallery UI

---

## 🗺️ Roadmap

### Step 1: Setup Cloudinary
**Todo:**
- [ ] Sign up at cloudinary.com
- [ ] Get API credentials
- [ ] Install: `cloudinary`, `multer`
- [ ] Configure in backend

---

### Step 2: Image Model
**Todo:**
- [ ] Image structure:
  - userId, title, description
  - url (Cloudinary URL)
  - thumbnailUrl
  - publicId (for deletion)
  - size, format, dimensions
  - albumId, tags array
  - uploadedAt

---

### Step 3: Upload Image
**Todo:**
- [ ] `POST /api/images/upload`
  - Use Multer to accept file
  - Validate: image only (jpg, png, gif)
  - Max size: 5MB
  - Upload to Cloudinary
  - Generate thumbnail (200x200)
  - Save metadata to DB
  - Return image URL

**Multer Config:**
```javascript
const upload = multer({
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('Only images allowed'));
    }
  }
});
```

---

### Step 4: Get Images
**Todo:**
- [ ] `GET /api/images` - All user's images
- [ ] `GET /api/images/:id` - Single image
- [ ] Pagination (24 per page)
- [ ] Sort by: newest, most viewed

---

### Step 5: Album Management
**Todo:**
- [ ] Album model:
  - name, description, coverImage
  - userId, imagesCount
- [ ] `POST /api/albums` - Create album
- [ ] `GET /api/albums` - All albums
- [ ] `POST /api/images/:id/add-to-album`
- [ ] `GET /api/albums/:id/images` - Images in album

---

### Step 6: Delete Image
**Todo:**
- [ ] `DELETE /api/images/:id`
  - Delete from Cloudinary using publicId
  - Delete from database
  - Return success

**Cloudinary Delete:**
```javascript
await cloudinary.uploader.destroy(image.publicId);
```

---

### Step 7: Image Optimization
**Todo:**
- [ ] Auto-compress images
- [ ] Generate multiple sizes:
  - Thumbnail (200x200)
  - Medium (800x600)
  - Original
- [ ] Serve responsive images

---

### Step 8: Search & Tags
**Todo:**
- [ ] `GET /api/images/search?q=sunset`
- [ ] `GET /api/images/tags/:tag`
- [ ] Auto-tag with AI (optional)

---

### Step 9: React Frontend
**Todo:**
- [ ] Upload form (drag & drop)
- [ ] Image grid gallery
- [ ] Lightbox for full view
- [ ] Albums page
- [ ] Tag filtering
- [ ] Search bar

---

## 💡 Hints & Tips

### Cloudinary Upload:
```javascript
const cloudinary = require('cloudinary').v2;

const result = await cloudinary.uploader.upload(file.path, {
  folder: 'gallery',
  transformation: [{ quality: 'auto', fetch_format: 'auto' }]
});

const imageUrl = result.secure_url;
const publicId = result.public_id;
```

### Generate Thumbnail URL:
```javascript
const thumbnailUrl = cloudinary.url(publicId, {
  width: 200,
  height: 200,
  crop: 'fill'
});
```

---

## 📚 Concepts

- File uploads
- Cloud storage
- Image optimization
- Gallery management
- Responsive images

---

**Next:** [Project 35: Profile Picture Upload](35-profile-picture-upload.md)
