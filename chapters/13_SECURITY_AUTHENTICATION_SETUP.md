# Chapter 13: Security and Authentication Setup

## 13.1 Password Hashing with bcrypt

### Install bcrypt

```bash
npm install bcrypt
```

### Hash Password Before Save

```javascript
const userSchema = new mongoose.Schema({
  password: {
    type: String,
    required: true
  }
});

// Pre-save hook: runs before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();

  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Method: Compare password
userSchema.methods.comparePassword = async function(enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};
```

## 13.2 JWT (JSON Web Tokens)

### Install jsonwebtoken

```bash
npm install jsonwebtoken
```

### Access vs Refresh Tokens

```javascript
userSchema.methods.generateAccessToken = function() {
  return jwt.sign(
    { _id: this._id, email: this.email },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }  // Expires in 15 minutes
  );
};

userSchema.methods.generateRefreshToken = function() {
  return jwt.sign(
    { _id: this._id },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }  // Expires in 7 days
  );
};
```

## 13.3 Custom Schema Methods

```javascript
userSchema.methods.isPasswordCorrect = async function(password) {
  return await bcrypt.compare(password, this.password);
};

userSchema.methods.hidePassword = function() {
  this.password = undefined;
  return this;
};
```

---

## 🎯 Next: Chapter 14 - Professional Project Setup
