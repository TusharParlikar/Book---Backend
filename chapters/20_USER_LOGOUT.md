# Chapter 20: User Authentication - Logout

## 20.1 Logout Implementation

```javascript
// controllers/authController.js

exports.logout = asyncHandler(async (req, res) => {
  // User ID comes from authentication middleware
  const userId = req.user._id;

  // Clear refresh token from database
  await User.findByIdAndUpdate(
    userId,
    { $unset: { refreshToken: 1 } },
    { new: true }
  );

  // Clear cookies
  const options = {
    httpOnly: true,
    secure: true
  };

  return res
    .status(200)
    .clearCookie('accessToken', options)
    .clearCookie('refreshToken', options)
    .json(new ApiResponse(200, {}, 'Logout successful'));
});
```

## 20.2 React Logout

```jsx
export function useLogout() {
  const handleLogout = async () => {
    try {
      await axios.post('http://localhost:5000/api/auth/logout');

      // Clear localStorage
      localStorage.removeItem('authToken');

      // Redirect to login
      navigate('/login');
    } catch (error) {
      console.error('Logout failed:', error);
    }
  };

  return { handleLogout };
}
```

---

## 🎯 Next: Chapter 21 - JWT Verification Middleware
