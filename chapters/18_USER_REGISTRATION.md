# Chapter 18: User Registration Implementation

## 18.1 Registration Flow Overview

```
React Form
  ↓ Submit
Express Route
  ↓ Validate
Controller
  ↓ Check existing user
  ↓ Upload avatar
  ↓ Hash password
MongoDB
  ↓ Save user
Response
  ↓ Return to React
```

## 18.2 Backend Registration

```javascript
// controllers/authController.js

const asyncHandler = require('../utils/asyncHandler');
const ApiError = require('../utils/ApiError');
const ApiResponse = require('../utils/ApiResponse');
const User = require('../models/User');
const uploadOnCloudinary = require('../utils/uploadOnCloudinary');

exports.register = asyncHandler(async (req, res) => {
  const { fullname, email, username, password } = req.body;

  // Validation
  if (!fullname || !email || !username || !password) {
    throw new ApiError(400, 'All fields required');
  }

  // Check if user exists
  const existingUser = await User.findOne({
    $or: [{ email }, { username }]
  });

  if (existingUser) {
    throw new ApiError(409, 'User already exists');
  }

  // Upload avatar if provided
  let avatarUrl = null;
  if (req.file) {
    const result = await uploadOnCloudinary(req.file.path);
    avatarUrl = result?.secure_url;
  }

  // Create user
  const user = await User.create({
    fullname,
    email,
    username,
    password,
    avatar: avatarUrl
  });

  // Return user without password
  const createdUser = await User.findById(user._id).select('-password');

  return res.status(201)
    .json(new ApiResponse(201, createdUser, 'User registered successfully'));
});
```

## 18.3 React Registration Form

```jsx
// components/Register.jsx

import { useState } from 'react';
import axios from 'axios';

export default function Register() {
  const [formData, setFormData] = useState({
    fullname: '',
    email: '',
    username: '',
    password: '',
    avatar: null
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleFileChange = (e) => {
    setFormData(prev => ({ ...prev, avatar: e.target.files[0] }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const form = new FormData();
    form.append('fullname', formData.fullname);
    form.append('email', formData.email);
    form.append('username', formData.username);
    form.append('password', formData.password);
    if (formData.avatar) {
      form.append('avatar', formData.avatar);
    }

    try {
      const response = await axios.post(
        'http://localhost:5000/api/auth/register',
        form,
        {
          headers: { 'Content-Type': 'multipart/form-data' }
        }
      );
      console.log('Registered:', response.data);
    } catch (error) {
      console.error('Error:', error.response.data);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="fullname" onChange={handleChange} placeholder="Full name" />
      <input name="email" onChange={handleChange} placeholder="Email" type="email" />
      <input name="username" onChange={handleChange} placeholder="Username" />
      <input name="password" onChange={handleChange} placeholder="Password" type="password" />
      <input type="file" onChange={handleFileChange} />
      <button type="submit">Register</button>
    </form>
  );
}
```

---

## 🎯 Next: Chapter 19 - User Login

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 17: File Uploads](./17_FILE_UPLOADS.md) | [📖 Course Index](../README.md) | [Chapter 19: User Login →](./19_USER_LOGIN.md) |
