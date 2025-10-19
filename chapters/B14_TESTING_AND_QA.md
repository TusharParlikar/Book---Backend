# B14: Testing & Quality Assurance ✅

> **Write Tests: Unit, Integration, and API Tests**

**Chapter Level:** Advanced  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B12  

---

## Setup

```bash
npm install jest supertest --save-dev
```

---

## Unit Tests

```javascript
// ============================================================
// FILE: tests/user.test.js
// ============================================================

const mongoose = require('mongoose');
const User = require('../models/User');

describe('User Model', () => {
  beforeAll(async () => {
    await mongoose.connect('mongodb://localhost:27017/test');
  });

  afterAll(async () => {
    await mongoose.disconnect();
  });

  test('Should create user with valid data', async () => {
    const user = await User.create({
      name: 'John',
      email: 'john@test.com',
      password: 'password123'
    });

    expect(user._id).toBeDefined();
    expect(user.name).toBe('John');
  });

  test('Should hash password', async () => {
    const user = await User.create({
      name: 'Jane',
      email: 'jane@test.com',
      password: 'password123'
    });

    expect(user.password).not.toBe('password123');
  });

  test('Should compare password correctly', async () => {
    const user = await User.create({
      name: 'Bob',
      email: 'bob@test.com',
      password: 'password123'
    });

    const isValid = await user.comparePassword('password123');
    expect(isValid).toBe(true);

    const isInvalid = await user.comparePassword('wrong');
    expect(isInvalid).toBe(false);
  });
});
```

---

## API Tests

```javascript
// ============================================================
// FILE: tests/api.test.js
// ============================================================

const request = require('supertest');
const app = require('../server');
const User = require('../models/User');

describe('Auth API', () => {
  test('POST /api/auth/register - Should register user', async () => {
    const response = await request(app)
      .post('/api/auth/register')
      .send({
        name: 'Test User',
        email: 'test@test.com',
        password: 'password123'
      });

    expect(response.status).toBe(201);
    expect(response.body.success).toBe(true);
    expect(response.body.data.email).toBe('test@test.com');
  });

  test('POST /api/auth/login - Should return token', async () => {
    // First create user
    await User.create({
      name: 'Login Test',
      email: 'login@test.com',
      password: 'password123'
    });

    // Then test login
    const response = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'login@test.com',
        password: 'password123'
      });

    expect(response.status).toBe(200);
    expect(response.body.data.token).toBeDefined();
  });

  test('POST /api/auth/login - Should fail with wrong password', async () => {
    const response = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'login@test.com',
        password: 'wrongpassword'
      });

    expect(response.status).toBe(401);
    expect(response.body.success).toBe(false);
  });
});
```

---

## Run Tests

```bash
# package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}

# Run tests
npm test

# Watch mode
npm run test:watch

# See coverage
npm run test:coverage
```

---

| Previous | Index | Next |
|----------|-------|------|
| [B13: Docker & Containerization](B13_DOCKER_CONTAINERIZATION.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B15: CI/CD & GitHub Actions](B15_CICD_GITHUB_ACTIONS.md) |
