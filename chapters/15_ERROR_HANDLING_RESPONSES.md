# Chapter 15: Error Handling and API Responses

## 15.1 Asynchronous Handler Utility

### asyncHandler Function

```javascript
// utils/asyncHandler.js

const asyncHandler = (fn) => async (req, res, next) => {
  try {
    await fn(req, res, next);
  } catch (error) {
    next(error);
  }
};

module.exports = asyncHandler;
```

### Usage

```javascript
// controllers/userController.js

const asyncHandler = require('../utils/asyncHandler');

exports.getUsers = asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json({ users });
});
```

## 15.2 Custom API Error Class

```javascript
// utils/ApiError.js

class ApiError extends Error {
  constructor(
    statusCode,
    message = 'Something went wrong',
    errors = [],
    stack = ''
  ) {
    super(message);
    this.statusCode = statusCode;
    this.data = null;
    this.message = message;
    this.success = false;
    this.errors = errors;

    if (stack) {
      this.stack = stack;
    } else {
      Error.captureStackTrace(this, this.constructor);
    }
  }
}

module.exports = ApiError;
```

### Usage

```javascript
exports.createUser = asyncHandler(async (req, res) => {
  if (!req.body.email) {
    throw new ApiError(400, 'Email is required');
  }
});
```

## 15.3 Standardized API Response Class

```javascript
// utils/ApiResponse.js

class ApiResponse {
  constructor(statusCode, data, message = 'Success') {
    this.statusCode = statusCode;
    this.data = data;
    this.message = message;
    this.success = statusCode < 400;
  }
}

module.exports = ApiResponse;
```

### Usage

```javascript
const ApiResponse = require('../utils/ApiResponse');

exports.getUsers = asyncHandler(async (req, res) => {
  const users = await User.find();
  return res
    .status(200)
    .json(new ApiResponse(200, users, 'Users fetched successfully'));
});
```

## 15.4 Global Error Handler

```javascript
// middlewares/errorHandler.js

const ApiError = require('../utils/ApiError');

const errorHandler = (err, req, res, next) => {
  if (err instanceof ApiError) {
    return res.status(err.statusCode).json({
      success: err.success,
      message: err.message,
      errors: err.errors
    });
  }

  return res.status(500).json({
    success: false,
    message: 'Internal Server Error'
  });
};

module.exports = errorHandler;
```

### Apply to Server

```javascript
// server.js

app.use(errorHandler);
```

---

## 🎯 Next: Chapter 16 - HTTP Fundamentals

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 14: Professional Project Setup](./14_PROFESSIONAL_PROJECT_SETUP.md) | [📖 Course Index](../README.md) | [Chapter 16: HTTP Fundamentals →](./16_HTTP_FUNDAMENTALS.md) |
