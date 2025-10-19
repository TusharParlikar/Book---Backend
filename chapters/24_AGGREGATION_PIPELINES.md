# Chapter 24: MongoDB Aggregation Pipelines

## 24.1 Understanding Aggregation

Aggregation pipelines process data in stages:

```
Database
  ↓ $match (filter)
  ↓ $lookup (join)
  ↓ $addFields (compute)
  ↓ $project (select fields)
  ↓ $sort (order)
Result
```

## 24.2 Common Aggregation Stages

### $match (Filtering)
```javascript
db.users.aggregate([
  { $match: { role: 'admin' } }
]);
```

### $lookup (Join)
```javascript
db.posts.aggregate([
  {
    $lookup: {
      from: 'users',
      localField: 'userId',
      foreignField: '_id',
      as: 'author'
    }
  }
]);
```

### $addFields (Compute)
```javascript
db.users.aggregate([
  {
    $addFields: {
      fullName: { $concat: ['$firstName', ' ', '$lastName'] }
    }
  }
]);
```

### $project (Select Fields)
```javascript
db.users.aggregate([
  {
    $project: {
      name: 1,
      email: 1,
      password: 0  // Exclude password
    }
  }
]);
```

## 24.3 Mongoose Aggregation

```javascript
// Get users with post count

const result = await User.aggregate([
  { $match: { _id: new mongoose.Types.ObjectId(userId) } },
  {
    $lookup: {
      from: 'posts',
      localField: '_id',
      foreignField: 'userId',
      as: 'posts'
    }
  },
  {
    $addFields: {
      postCount: { $size: '$posts' }
    }
  },
  {
    $project: {
      name: 1,
      email: 1,
      postCount: 1,
      posts: 0
    }
  }
]);
```

---

## 🎯 Next: Chapter 25 - Channel Profile Implementation
