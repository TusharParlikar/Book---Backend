# Chapter 25: Channel Profile Implementation

## 25.1 Get User Channel Profile

```javascript
exports.getUserChannelProfile = asyncHandler(async (req, res) => {
  const { username } = req.params;

  if (!username?.trim()) {
    throw new ApiError(400, 'Username required');
  }

  const channel = await User.aggregate([
    { $match: { username: username.toLowerCase() } },
    {
      $lookup: {
        from: 'subscriptions',
        localField: '_id',
        foreignField: 'channel',
        as: 'subscribers'
      }
    },
    {
      $lookup: {
        from: 'subscriptions',
        localField: '_id',
        foreignField: 'subscriber',
        as: 'subscribedTo'
      }
    },
    {
      $addFields: {
        subscribersCount: { $size: '$subscribers' },
        channelsSubscribedToCount: { $size: '$subscribedTo' }
      }
    },
    {
      $project: {
        username: 1,
        fullname: 1,
        avatar: 1,
        subscribersCount: 1,
        channelsSubscribedToCount: 1
      }
    }
  ]);

  if (!channel?.length) {
    throw new ApiError(404, 'Channel not found');
  }

  return res
    .status(200)
    .json(new ApiResponse(200, channel[0], 'Channel profile fetched'));
});
```

---

## 🎯 Next: Chapter 26 - Watch History

---

## 📚 Chapter Navigation

| Previous | Index | Next |
|----------|-------|------|
| [← Chapter 24: Aggregation Pipelines](./24_AGGREGATION_PIPELINES.md) | [📖 Course Index](../INDEX.md) | [Chapter 26: Watch History →](./26_WATCH_HISTORY.md) |
