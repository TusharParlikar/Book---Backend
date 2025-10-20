# Project 17: Course Management System

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B06 - Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md)

---

## 📝 Description

Build an online learning platform API with courses, modules, lessons, and student enrollment.

**What You'll Learn:**
- Hierarchical data (courses → modules → lessons)
- Many-to-many relationships (students ↔ courses)
- Progress tracking
- Completion certificates
- Enrollment management

---

## 🎯 Project Goals

- ✅ Manage courses, modules, lessons
- ✅ Student enrollment system
- ✅ Track progress and completion
- ✅ Quizzes and assessments
- ✅ React learning platform UI

---

## 🗺️ Roadmap

### Step 1: Data Models

**Courses:**
- [ ] title, description, instructor
- [ ] duration, level, price
- [ ] thumbnail, category
- [ ] modules (array of module IDs)

**Modules:**
- [ ] title, description, order
- [ ] courseId
- [ ] lessons (array of lesson IDs)

**Lessons:**
- [ ] title, content, videoUrl
- [ ] duration, moduleId
- [ ] resources (PDF, links)

**Enrollments:**
- [ ] studentId, courseId
- [ ] enrolledDate, progress (%)
- [ ] completedLessons (array)
- [ ] certificateIssued (boolean)

---

### Step 2: Course Management
**Todo:**
- [ ] `POST /api/courses` - Create course
- [ ] `GET /api/courses` - All courses
- [ ] `GET /api/courses/:id` - Full course with modules
- [ ] `PUT /api/courses/:id` - Update
- [ ] `DELETE /api/courses/:id` - Delete

---

### Step 3: Module & Lesson Management
**Todo:**
- [ ] `POST /api/courses/:courseId/modules` - Add module
- [ ] `POST /api/modules/:moduleId/lessons` - Add lesson
- [ ] `GET /api/courses/:id/content` - Full hierarchy
- [ ] Update, delete modules/lessons

---

### Step 4: Enrollment System
**Todo:**
- [ ] `POST /api/enrollments` - Enroll student
  - Body: { studentId, courseId }
  - Check: not already enrolled
  - Initialize progress to 0%
- [ ] `GET /api/students/:id/courses` - Enrolled courses
- [ ] `DELETE /api/enrollments/:id` - Unenroll

---

### Step 5: Progress Tracking
**Todo:**
- [ ] `POST /api/enrollments/:id/complete-lesson`
  - Body: { lessonId }
  - Add to completedLessons
  - Recalculate progress percentage
- [ ] `GET /api/enrollments/:id/progress` - Progress report

**Progress Calculation:**
```
totalLessons = count all lessons in course
completedCount = completedLessons.length
progress = (completedCount / totalLessons) * 100
```

---

### Step 6: Certificates
**Todo:**
- [ ] When progress reaches 100%, auto-generate certificate
- [ ] `GET /api/enrollments/:id/certificate`
- [ ] Include: student name, course title, completion date
- [ ] Generate PDF (use package like `pdfkit`)

---

### Step 7: Search & Filter
**Todo:**
- [ ] `GET /api/courses?category=Programming`
- [ ] `GET /api/courses?level=beginner`
- [ ] `GET /api/courses?search=javascript`
- [ ] Sort by: popularity, rating, newest

---

### Step 8: React Frontend
**Todo:**
- [ ] Course catalog
- [ ] Course detail page
- [ ] Enroll button
- [ ] Student dashboard (enrolled courses)
- [ ] Course player (video + progress)
- [ ] Certificate download

---

## 💡 Hints & Tips

### Getting Full Course Hierarchy:
Populate modules and lessons when fetching course

### Progress Calculation:
Track completed lessons, calculate percentage dynamically

### Certificate Generation:
Use library or simple JSON object with completion data

---

## 📚 Concepts

- Hierarchical data
- Many-to-many relationships
- Progress tracking
- Enrollment logic
- Certificate generation

---

**Next:** [Project 18: Job Board API](18-job-board.md)
