# Project 12: Polling System API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Create a voting/polling system where users can create polls and vote on them.

**What You'll Learn:**
- Poll creation
- Vote counting
- Duplicate vote prevention
- Results calculation
- Real-time updates (optional)

---

## 🎯 Project Goals

- ✅ Create polls with multiple options
- ✅ Vote on polls
- ✅ Prevent duplicate voting (IP-based)
- ✅ Display results as percentages
- ✅ React UI with poll creation and voting

---

## 🗺️ Roadmap

### Step 1: Poll Model
**Todo:**
- [ ] Poll structure:
  - id, question, options array
  - createdBy, createdAt, expiresAt
  - allowMultipleVotes (boolean)
- [ ] Option structure:
  - id, text, votes (count)

---

### Step 2: Create Poll Routes
**Todo:**
- [ ] `POST /api/polls` - Create new poll
  - Body: { question, options: [text1, text2, ...] }
- [ ] `GET /api/polls` - Get all polls
- [ ] `GET /api/polls/:id` - Get single poll
- [ ] `DELETE /api/polls/:id` - Delete poll

---

### Step 3: Voting System
**Todo:**
- [ ] `POST /api/polls/:id/vote`
  - Body: { optionId }
- [ ] Increment vote count for chosen option
- [ ] Return updated poll with results

---

### Step 4: Duplicate Vote Prevention
**Todo:**
- [ ] Track voter IPs or user IDs
- [ ] Create votes collection: { pollId, optionId, voterIP }
- [ ] Before accepting vote, check if IP already voted
- [ ] Return error if already voted

**Hint:** Get IP from `req.ip`

---

### Step 5: Results Calculation
**Todo:**
- [ ] Calculate total votes
- [ ] Calculate percentage for each option
- [ ] Return results object
- [ ] Sort options by vote count

**Formula:**
```
percentage = (optionVotes / totalVotes) * 100
```

---

### Step 6: Poll Expiry
**Todo:**
- [ ] Add expiresAt field
- [ ] Check if poll expired before accepting votes
- [ ] Mark polls as closed
- [ ] `GET /api/polls/active` - Only active polls

---

### Step 7: React Frontend
**Todo:**
- [ ] Create poll form
- [ ] Add/remove options dynamically
- [ ] Display polls list
- [ ] Vote buttons for each option
- [ ] Results visualization (bar chart)
- [ ] Show percentages

---

## 💡 Hints & Tips

### Dynamic Options in React:
```javascript
const [options, setOptions] = useState(['', '']);
```

### Results Visualization:
Use percentage width for bars:
```jsx
<div style={{ width: `${percentage}%` }}>
```

### Preventing Duplicate Votes:
```javascript
const hasVoted = votes.some(v => 
  v.pollId === pollId && v.voterIP === req.ip
);
```

---

## 📚 Concepts

- Voting systems
- Duplicate prevention
- Percentage calculations
- Data aggregation
- Dynamic forms in React

---

**Next:** [Project 13: Expense Tracker API](13-expense-tracker.md)
