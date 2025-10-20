# Project 43: Leaderboard System

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md)

---

## 📝 Description

Build a ranking system with scores, achievements, and real-time leaderboards.

**What You'll Learn:**
- Score tracking
- Ranking algorithms
- Real-time updates
- Achievement systems
- Position calculation

---

## 🎯 Project Goals

- ✅ Track user scores
- ✅ Calculate rankings
- ✅ Weekly/monthly/all-time leaderboards
- ✅ Achievement badges
- ✅ React leaderboard UI

---

## 🗺️ Roadmap

### Step 1: Score Model
**Todo:**
- [ ] userId, score, timestamp
- [ ] gameId or category
- [ ] metadata (level, duration)

### Step 2: Update Score
**Todo:**
- [ ] `POST /api/scores`
- [ ] Body: { score }
- [ ] Add to user's total
- [ ] Or keep highest score only

### Step 3: Get Leaderboard
**Todo:**
- [ ] `GET /api/leaderboard?period=weekly`
- [ ] Sort by score (descending)
- [ ] Return top 100
- [ ] Include rank position

### Step 4: Calculate Rank
**Todo:**
- [ ] Count users with higher score
- [ ] rank = count + 1
- [ ] Or use dense ranking (tie handling)

### Step 5: Achievements
**Todo:**
- [ ] Award badges for milestones
- [ ] First place badge
- [ ] Top 10 badge
- [ ] Streak badges

### Step 6: React UI
**Todo:**
- [ ] Leaderboard table
- [ ] Current user's rank highlighted
- [ ] Tabs for different periods
- [ ] Achievements display

---

**Next:** [Project 44: Webhook System](44-webhook-system.md)
