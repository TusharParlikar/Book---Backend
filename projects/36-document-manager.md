# Project 36: Document Manager API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4 hours  
**Related Chapter:** [B11 - File Uploads](../chapters/B11_FILE_UPLOADS.md)

---

## 📝 Description

Build a file management system for documents with folders, search, and sharing.

**What You'll Learn:**
- File upload (PDF, DOCX, etc.)
- Folder hierarchy
- File sharing
- Download links
- Storage management

---

## 🎯 Project Goals

- ✅ Upload documents
- ✅ Organize in folders
- ✅ Share files with others
- ✅ Search documents
- ✅ React file manager UI

---

## 🗺️ Roadmap

### Step 1: File Model
**Todo:**
- [ ] name, originalName, size, type
- [ ] url (cloud URL), folderId
- [ ] userId, uploadedAt
- [ ] isPublic, sharedWith (array)

### Step 2: Upload File
**Todo:**
- [ ] `POST /api/files/upload`
- [ ] Accept: PDF, DOCX, TXT, ZIP
- [ ] Max: 10MB
- [ ] Upload to Cloudinary or S3
- [ ] Save metadata

### Step 3: Folder System
**Todo:**
- [ ] `POST /api/folders` - Create folder
- [ ] `GET /api/folders` - All folders
- [ ] `GET /api/folders/:id/files` - Files in folder

### Step 4: Share File
**Todo:**
- [ ] `POST /api/files/:id/share`
- [ ] Body: { email }
- [ ] Generate share link
- [ ] Send email with link

### Step 5: React UI
**Todo:**
- [ ] Folder tree sidebar
- [ ] File list view
- [ ] Upload button
- [ ] Share modal

---

**Next:** [Project 37: Real-Time Chat](37-realtime-chat.md)
