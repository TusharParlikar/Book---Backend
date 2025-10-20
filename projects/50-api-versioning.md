# Project 50: API Versioning & Deprecation

**Difficulty:** 🔴 Advanced  
**Estimated Time:** 4 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Implement API versioning to maintain backward compatibility while introducing new features.

**What You'll Learn:**
- API versioning strategies
- Version management
- Deprecation workflows
- Breaking change handling
- Client migration

---

## 🎯 Project Goals

- ✅ Support multiple API versions
- ✅ Gradual deprecation strategy
- ✅ Version routing
- ✅ Documentation for each version
- ✅ React version selector

---

## 🗺️ Roadmap

### Step 1: Choose Versioning Strategy
**Todo:**
- [ ] **URL Versioning:** `/api/v1/users`, `/api/v2/users`
- [ ] **Header Versioning:** `Accept: application/vnd.api.v1+json`
- [ ] **Query Param:** `/api/users?version=1`

**Recommended:** URL versioning (most common)

---

### Step 2: Setup Version Routes
**Todo:**
- [ ] Create folder structure:
  ```
  routes/
    v1/
      users.js
      posts.js
    v2/
      users.js
      posts.js
  ```
- [ ] Mount routes:
  ```javascript
  app.use('/api/v1', v1Routes);
  app.use('/api/v2', v2Routes);
  ```

---

### Step 3: Shared Logic
**Todo:**
- [ ] Reuse controllers for unchanged endpoints
- [ ] Only create new versions for changed endpoints
- [ ] Share models across versions

---

### Step 4: Deprecation Warnings
**Todo:**
- [ ] Add headers to deprecated versions:
  ```javascript
  res.set('Warning', '299 - "API v1 is deprecated. Migrate to v2 by 2025-12-31"');
  res.set('Sunset', 'Wed, 31 Dec 2025 23:59:59 GMT');
  ```
- [ ] Log usage of deprecated endpoints
- [ ] Send email to users still using old version

---

### Step 5: Version Documentation
**Todo:**
- [ ] Document changes between versions
- [ ] Migration guide
- [ ] Changelog
- [ ] Breaking changes highlighted

**Changelog Example:**
```markdown
## V2 (Current)
- New: Added pagination to /users
- Changed: /posts response format (array → object with metadata)
- Removed: /users/search (merged into /users?q=)

## V1 (Deprecated - Sunset: 2025-12-31)
- Original API
```

---

### Step 6: Default Version
**Todo:**
- [ ] `/api/users` → redirects to latest version
- [ ] Or return error asking to specify version
- [ ] Set default in header if not specified

---

### Step 7: React Version Handling
**Todo:**
- [ ] API client with configurable version
- [ ] Environment variable for API version
- [ ] Show upgrade prompt if using deprecated version

**React Client:**
```javascript
const API_VERSION = process.env.REACT_APP_API_VERSION || 'v2';
const baseURL = `/api/${API_VERSION}`;
```

---

### Step 8: Sunset Old Versions
**Todo:**
- [ ] Monitor usage of old versions
- [ ] Send migration warnings 3 months before
- [ ] Final warning 1 month before
- [ ] Remove old version on sunset date
- [ ] Return 410 Gone for removed versions

---

## 💡 Hints & Tips

### Version Router:
```javascript
// app.js
app.use('/api/v1', require('./routes/v1'));
app.use('/api/v2', require('./routes/v2'));

// Default to latest
app.use('/api', require('./routes/v2'));
```

### Deprecation Middleware:
```javascript
function deprecationWarning(version, sunsetDate) {
  return (req, res, next) => {
    res.set('Warning', `299 - "API ${version} is deprecated. Migrate by ${sunsetDate}"`);
    res.set('Sunset', sunsetDate);
    next();
  };
}

app.use('/api/v1', deprecationWarning('v1', '2025-12-31'));
```

### Track Version Usage:
```javascript
// Log which version is being used
logger.info(`API ${version} called: ${req.method} ${req.path}`);
```

---

## 📚 Concepts

- API versioning
- Backward compatibility
- Deprecation strategies
- Breaking changes
- Client migration
- Sunset policies

---

## ✅ Completion Checklist

**Backend:**
- [ ] Multiple versions supported
- [ ] Deprecation headers set
- [ ] Usage tracking implemented
- [ ] Documentation updated
- [ ] Migration guide created

**Frontend:**
- [ ] Configurable API version
- [ ] Deprecation warnings displayed
- [ ] Easy version switching

---

## 🎉 Congratulations!

You've completed all 50 backend projects! You've learned:

- **Beginner (1-15):** Core concepts, CRUD, APIs, routing
- **Intermediate (16-30):** Databases, auth, file uploads, e-commerce
- **Advanced (31-45):** Social media, algorithms, real-time, DevOps
- **Expert (46-50):** Performance, scaling, monitoring, best practices

**Next Steps:**
1. Deploy your projects to production
2. Add them to your portfolio
3. Contribute to open source
4. Build your own original project
5. Apply for backend developer positions

**You're now a backend developer! 🚀**

---

## 🔗 Related Resources

**All Chapters:**
- [B01-B15](../chapters/) - Complete course content
- [README](../README.md) - Course overview
- [INDEX](../INDEX.md) - Chapter navigation

**External:**
- [Express.js Best Practices](https://expressjs.com/en/advanced/best-practice-performance.html)
- [REST API Design Guidelines](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
- [Semantic Versioning](https://semver.org/)

---

**Made with ❤️ by Tushar Parlikar**
