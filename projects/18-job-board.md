# Project 18: Job Board API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 4-5 hours  
**Related Chapter:** [B06 - Database Relationships](../chapters/B06_DATABASE_RELATIONSHIPS.md), [B08 - JWT Authentication](../chapters/B08_JWT_AUTHENTICATION.md)

---

## 📝 Description

Create a job posting platform where companies post jobs and candidates apply.

**What You'll Learn:**
- Role-based access (employers vs candidates)
- Application management
- Search and filtering
- Email notifications
- Resume handling

---

## 🎯 Project Goals

- ✅ Company and candidate profiles
- ✅ Job posting and applications
- ✅ Search with multiple filters
- ✅ Application status tracking
- ✅ React job board UI

---

## 🗺️ Roadmap

### Step 1: User Models

**Employers:**
- [ ] companyName, email, password
- [ ] description, website, logo
- [ ] location, industry

**Candidates:**
- [ ] name, email, password
- [ ] resumeUrl, skills array
- [ ] experience, education

**Jobs:**
- [ ] title, description, requirements
- [ ] employerId, location, type (full-time/part-time)
- [ ] salary range, posted date
- [ ] status (open/closed)

**Applications:**
- [ ] jobId, candidateId
- [ ] appliedDate, status (pending/reviewed/rejected/accepted)
- [ ] coverLetter

---

### Step 2: Job Posting
**Todo:**
- [ ] `POST /api/jobs` - Create job (employer only)
- [ ] `GET /api/jobs` - All jobs (public)
- [ ] `GET /api/jobs/:id` - Job details
- [ ] `PUT /api/jobs/:id` - Update (employer only)
- [ ] `DELETE /api/jobs/:id` - Delete (employer only)

---

### Step 3: Search & Filters
**Todo:**
- [ ] `GET /api/jobs?search=developer`
- [ ] `GET /api/jobs?location=Remote`
- [ ] `GET /api/jobs?type=full-time`
- [ ] `GET /api/jobs?salary min=50000&salaryMax=100000`
- [ ] Sort by: newest, salary

---

### Step 4: Application System
**Todo:**
- [ ] `POST /api/applications`
  - Body: { jobId, coverLetter }
  - Check: not already applied
  - Auto-fill candidateId from auth token
- [ ] `GET /api/applications/my` - Candidate's applications
- [ ] `GET /api/jobs/:id/applications` - Employer views applications

---

### Step 5: Application Management
**Todo:**
- [ ] `PATCH /api/applications/:id/status`
  - Update: pending → reviewed → accepted/rejected
  - Only job employer can update
- [ ] Send email notification on status change

---

### Step 6: Recommendations
**Todo:**
- [ ] `GET /api/jobs/recommended` - Jobs matching candidate skills
- [ ] Match job requirements with candidate skills
- [ ] Sort by relevance

**Algorithm:**
```
Count matching skills
Calculate match percentage
Return sorted by percentage
```

---

### Step 7: React Frontend
**Todo:**
- [ ] Job listings page
- [ ] Job detail page
- [ ] Post job form (employer)
- [ ] Application form
- [ ] Employer dashboard (posted jobs + applications)
- [ ] Candidate dashboard (applied jobs + status)

---

## 💡 Hints & Tips

### Role-Based Auth:
```javascript
if (req.user.role !== 'employer') {
  return res.status(403).json({ error: 'Forbidden' });
}
```

### Skill Matching:
```javascript
const matchCount = job.requirements.filter(req => 
  candidate.skills.includes(req)
).length;
const percentage = (matchCount / job.requirements.length) * 100;
```

---

## 📚 Concepts

- Role-based access
- Application workflows
- Skill matching
- Email notifications
- Status management

---

**Next:** [Project 19: Real Estate Listing API](19-real-estate-listing.md)
