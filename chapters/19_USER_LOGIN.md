# Chapter 19: User Authentication - Login

## 19.1 Login Flow

```
React Form (email, password)
  ↓ Submit
Express Route
  ↓ Find user
  ↓ Verify password
  ↓ Generate tokens
  ↓ Set cookies
Response (tokens)
  ↓ Save in localStorage
React Dashboard
```

## 19.2 Backend Login

```javascript
// controllers/authController.js

exports.login = asyncHandler(async (req, res) => {
  const { email, password } = req.body;

  if (!email || !password) {
    throw new ApiError(400, 'Email and password required');
  }

  // Find user (include password field)
  const user = await User.findOne({ email }).select('+password');

  if (!user) {
    throw new ApiError(401, 'Invalid credentials');
  }

  // Verify password
  const isPasswordCorrect = await user.isPasswordCorrect(password);

  if (!isPasswordCorrect) {
    throw new ApiError(401, 'Invalid credentials');
  }

  // Generate tokens
  const accessToken = user.generateAccessToken();
  const refreshToken = user.generateRefreshToken();

  // Save refresh token to database
  user.refreshToken = refreshToken;
  await user.save({ validateBeforeSave: false });

  // Send tokens as secure HTTP-only cookies
  const options = {
    httpOnly: true,
    secure: true
  };

  return res
    .status(200)
    .cookie('accessToken', accessToken, options)
    .cookie('refreshToken', refreshToken, options)
    .json(new ApiResponse(200, { user }, 'Login successful'));
});
```

## 19.3 React Login

```jsx
// components/Login.jsx

import { useState } from 'react';
import axios from 'axios';

export default function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async (e) => {
    e.preventDefault();

    try {
      const response = await axios.post(
        'http://localhost:5000/api/auth/login',
        { email, password }
      );

      // Save token to localStorage
      localStorage.setItem('authToken', response.data.data.accessToken);

      console.log('Logged in successfully');
      // Redirect to dashboard
    } catch (error) {
      console.error('Login failed:', error.response.data);
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" />
      <input value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" type="password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## 🎯 Next: Chapter 20 - User Logout
