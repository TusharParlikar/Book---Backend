# Chapter 21: JWT Verification Middleware

## 21.1 Purpose of verifyJWT

Protects routes - only authenticated users can access.

## 21.2 JWT Middleware

```javascript
// middlewares/auth.js

const jwt = require('jsonwebtoken');
const ApiError = require('../utils/ApiError');
const asyncHandler = require('../utils/asyncHandler');
const User = require('../models/User');

exports.verifyJWT = asyncHandler(async (req, res, next) => {
  // Extract token from cookies or Authorization header
  const token =
    req.cookies?.accessToken ||
    req.header('Authorization')?.replace('Bearer ', '');

  if (!token) {
    throw new ApiError(401, 'Unauthorized');
  }

  try {
    // Verify and decode token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Get user from database
    const user = await User.findById(decoded._id);

    if (!user) {
      throw new ApiError(401, 'User not found');
    }

    // Attach user to request
    req.user = user;
    next();
  } catch (error) {
    throw new ApiError(401, 'Invalid token');
  }
});
```

## 21.3 Using Protected Routes

```javascript
// routes/userRoutes.js

const { verifyJWT } = require('../middlewares/auth');

router.get('/profile', verifyJWT, userController.getProfile);
router.put('/profile', verifyJWT, userController.updateProfile);
router.delete('/profile', verifyJWT, userController.deleteProfile);
```

---

## 🎯 Next: Chapter 22 - Token Refresh

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 20: User Logout](./20_USER_LOGOUT.md) | [📖 Course Index](../INDEX.md) | [Chapter 22: Token Refresh →](./22_TOKEN_REFRESH.md) |
