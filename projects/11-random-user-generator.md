# Project 11: Random User Generator API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 2 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Generate random user data for testing purposes. Useful for populating databases during development.

**What You'll Learn:**
- Random data generation
- Faker library usage
- Bulk data generation
- Custom data formats
- Seeding databases

---

## 🎯 Project Goals

- ✅ Generate random users with realistic data
- ✅ Customize number of users
- ✅ Filter by nationality, gender
- ✅ Generate avatars
- ✅ React UI to display generated users

---

## 🗺️ Roadmap

### Step 1: Setup Faker
**Todo:**
- [ ] Install `@faker-js/faker`
- [ ] Explore faker methods
- [ ] Create user generator function

**User Data to Generate:**
- Name (first, last, full)
- Email
- Phone
- Address (street, city, state, zip, country)
- Date of birth
- Avatar URL
- Username
- Password (hashed)
- Bio

---

### Step 2: Create Routes
**Todo:**
- [ ] `GET /api/users/random` - 1 random user
- [ ] `GET /api/users/random?count=10` - Multiple users
- [ ] `GET /api/users/random?gender=male`
- [ ] `GET /api/users/random?nat=US` - Filter by nationality

---

### Step 3: User Generator Function
**Todo:**
- [ ] Create `utils/userGenerator.js`
- [ ] Function: `generateUser(options)`
- [ ] Options: gender, nationality, age range
- [ ] Return formatted user object

**Hint:**
```javascript
const { faker } = require('@faker-js/faker');
const user = {
  name: faker.person.fullName(),
  email: faker.internet.email(),
  // ... more fields
};
```

---

### Step 4: Avatar Generation
**Todo:**
- [ ] Use service: ui-avatars.com or DiceBear
- [ ] Generate based on user name/email
- [ ] Return avatar URL in user object

**Avatar URL Example:**
```
https://ui-avatars.com/api/?name=John+Doe
```

---

### Step 5: Bulk Generation
**Todo:**
- [ ] `POST /api/users/generate`
- [ ] Body: `{ count: 100, gender: 'female' }`
- [ ] Return array of users
- [ ] Limit: max 1000 users per request

---

### Step 6: Export Formats
**Todo:**
- [ ] `GET /api/users/random?format=json`
- [ ] `GET /api/users/random?format=csv`
- [ ] `GET /api/users/random?format=sql`
- [ ] Generate appropriate format

---

### Step 7: React Frontend
**Todo:**
- [ ] User count selector (1-100)
- [ ] Gender filter (all, male, female)
- [ ] Nationality dropdown
- [ ] Generate button
- [ ] Display users as cards
- [ ] Export button (JSON/CSV)

---

## 💡 Hints & Tips

### Generating Multiple Users:
```javascript
function generateUsers(count) {
  return Array.from({ length: count }, () => generateUser());
}
```

### CSV Format:
Use `json2csv` package to convert

### Nationality Codes:
US, GB, FR, DE, IN, etc.

---

## 📚 Concepts

- Faker.js library
- Random data generation
- Data formatting
- CSV generation
- Array manipulation

---

**Next:** [Project 12: Polling System API](12-polling-system.md)
