# Chapter 22: Token Refresh Implementation

## 22.1 Why Refresh Tokens?

- Access tokens are short-lived (15 mins)
- Refresh tokens are long-lived (7 days)
- When access token expires, use refresh token to get new one

## 22.2 Refresh Token Endpoint

```javascript
// controllers/authController.js

exports.refreshAccessToken = asyncHandler(async (req, res) => {
  const incomingRefreshToken =
    req.cookies.refreshToken ||
    req.body.refreshToken;

  if (!incomingRefreshToken) {
    throw new ApiError(401, 'Refresh token required');
  }

  try {
    // Verify refresh token
    const decoded = jwt.verify(
      incomingRefreshToken,
      process.env.JWT_REFRESH_SECRET
    );

    // Get user
    const user = await User.findById(decoded._id);

    if (!user || user.refreshToken !== incomingRefreshToken) {
      throw new ApiError(401, 'Invalid refresh token');
    }

    // Generate new tokens
    const newAccessToken = user.generateAccessToken();
    const newRefreshToken = user.generateRefreshToken();

    user.refreshToken = newRefreshToken;
    await user.save({ validateBeforeSave: false });

    const options = {
      httpOnly: true,
      secure: true
    };

    return res
      .status(200)
      .cookie('accessToken', newAccessToken, options)
      .cookie('refreshToken', newRefreshToken, options)
      .json(new ApiResponse(200, {}, 'Token refreshed'));
  } catch (error) {
    throw new ApiError(401, 'Failed to refresh token');
  }
});
```

---

## 🎯 Next: Chapter 23 - User Account Management
