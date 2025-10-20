# B15: CI/CD & GitHub Actions 🚀

> **Automated Testing and Deployment: Deploy on Every Push**

**Chapter Level:** Advanced  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B14  

---

## What is CI/CD?

```
CI (Continuous Integration):
- Automatically run tests on every code push
- Catch bugs early
- Ensure code quality

CD (Continuous Deployment):
- Automatically deploy to production
- No manual deployment
- Faster releases
```

---

## GitHub Actions Workflow

```yaml
# ============================================================
# FILE: .github/workflows/test.yml
# PURPOSE: Run tests on every push
# ============================================================

name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]

    services:
      mongodb:
        image: mongo:latest
        options: >-
          --health-cmd mongosh
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 27017:27017

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
        env:
          MONGO_URI: mongodb://localhost:27017/test

      - name: Upload coverage
        uses: codecov/codecov-action@v2


# ============================================================
# FILE: .github/workflows/deploy.yml
# PURPOSE: Deploy on success
# ============================================================

name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Deploy to Railway
        run: |
          npm install -g @railway/cli
          railway login --token ${{ secrets.RAILWAY_TOKEN }}
          railway deploy
```

---

## Deployment Platforms

### Railway (Simple)

```bash
# Install CLI
npm install -g @railway/cli

# Login
railway login

# Deploy
railway deploy
```

<details>
<summary>💡 Expected `railway deploy` Output</summary>

```bash
✓ Building...
✓ Uploading...
✓ Deploying...
✓ Deployment successful!
URL: https://my-app-production.up.railway.app
```
</details>

```bash
# Set environment variables
railway variables
```

### Heroku

```bash
# Install CLI
npm install -g heroku

# Login
heroku login

# Create app
heroku create my-app

# Deploy
git push heroku main

# View logs
heroku logs --tail
```

---

## Production Checklist

```
✅ Environment variables configured (.env)
✅ Database connection secured
✅ Error handling in place
✅ Logging enabled
✅ Rate limiting set up
✅ CORS configured
✅ Tests passing
✅ Security headers set
✅ Input validation
✅ Database backups scheduled
```

---

## Complete Workflow

```
1. Developer pushes code to GitHub
2. GitHub Actions runs tests
3. If tests pass:
   - Build Docker image
   - Push to container registry
   - Deploy to production
4. If tests fail:
   - Notify developer
   - Block merge
5. Monitor in production
6. Auto-rollback if issues
```

---

| Previous | Index | Completed! |
|----------|-------|-----------|
| [B14: Testing & QA](B14_TESTING_AND_QA.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | 🎉 |

---

## 🎓 Congratulations!

You've completed the entire Backend Developer course (B01-B15)!

### What You've Learned:

**Foundation (B01-B03)**
- How backend works
- Express routing
- MVC architecture

**Database (B04-B06)**
- MongoDB basics
- Mongoose schemas
- Database relationships

**Authentication (B07-B09)**
- Password security
- JWT tokens
- Complete auth flow

**Advanced (B10-B12)**
- E-commerce backend
- File uploads
- Real projects

**Industry Standards (B13-B15)**
- Docker containerization
- Testing & QA
- CI/CD deployment

### What You Can Build:

- ✅ REST APIs
- ✅ Full-stack apps
- ✅ Real-time systems
- ✅ Production applications
- ✅ Scalable systems

### Next Steps:

1. **Build Real Projects** - Apply what you learned
2. **Join Communities** - Network with developers
3. **Keep Learning** - Advanced topics:
   - WebSockets (real-time)
   - Microservices
   - GraphQL
   - Machine Learning APIs
4. **Get Hired** - Your portfolio ready!

### Resources:

- [Node.js Docs](https://nodejs.org/docs/)
- [Express Docs](https://expressjs.com/)
- [MongoDB Docs](https://docs.mongodb.com/)
- [Docker Docs](https://docs.docker.com/)
- [GitHub Actions](https://github.com/features/actions)

---

**Author:** Tushar Parlikar  
**Last Updated:** October 20, 2025  
**Duration:** ~40-50 hours of comprehensive content  
**Level:** Beginner to Intermediate

---

🎉 **You're ready to become a backend developer!**

Go build something amazing! 🚀
