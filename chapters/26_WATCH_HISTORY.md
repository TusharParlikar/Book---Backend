# Chapter 26: Watch History Implementation

## 26.1 Get Watch History

```javascript
exports.getWatchHistory = asyncHandler(async (req, res) => {
  const userId = new mongoose.Types.ObjectId(req.user._id);

  const watchHistory = await User.aggregate([
    { $match: { _id: userId } },
    {
      $lookup: {
        from: 'videos',
        localField: 'watchHistory',
        foreignField: '_id',
        as: 'watchHistory',
        pipeline: [
          {
            $lookup: {
              from: 'users',
              localField: 'owner',
              foreignField: '_id',
              as: 'owner',
              pipeline: [
                {
                  $project: {
                    username: 1,
                    avatar: 1
                  }
                }
              ]
            }
          },
          {
            $addFields: {
              owner: { $first: '$owner' }
            }
          }
        ]
      }
    }
  ]);

  return res
    .status(200)
    .json(new ApiResponse(200, watchHistory[0].watchHistory, 'Watch history'));
});
```

---

## 🎯 Next: Chapter 27 - Course Summary
