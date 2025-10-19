# Chapter 23: User Account Management

## 23.1 Change Password

```javascript
exports.changePassword = asyncHandler(async (req, res) => {
  const { oldPassword, newPassword } = req.body;

  const user = await User.findById(req.user._id).select('+password');

  if (!(await user.isPasswordCorrect(oldPassword))) {
    throw new ApiError(400, 'Invalid old password');
  }

  user.password = newPassword;
  await user.save();

  return res
    .status(200)
    .json(new ApiResponse(200, {}, 'Password changed'));
});
```

## 23.2 Update Account Details

```javascript
exports.updateAccountDetails = asyncHandler(async (req, res) => {
  const { fullname, email } = req.body;

  const user = await User.findByIdAndUpdate(
    req.user._id,
    { $set: { fullname, email } },
    { new: true }
  ).select('-password');

  return res
    .status(200)
    .json(new ApiResponse(200, user, 'Account updated'));
});
```

## 23.3 Update Avatar

```javascript
exports.updateAvatar = asyncHandler(async (req, res) => {
  if (!req.file) {
    throw new ApiError(400, 'Avatar file required');
  }

  const result = await uploadOnCloudinary(req.file.path);

  const user = await User.findByIdAndUpdate(
    req.user._id,
    { $set: { avatar: result.secure_url } },
    { new: true }
  ).select('-password');

  return res
    .status(200)
    .json(new ApiResponse(200, user, 'Avatar updated'));
});
```

---

## 🎯 Next: Chapter 24 - MongoDB Aggregation
