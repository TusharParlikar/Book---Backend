# Project 13: Expense Tracker API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md), [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md)

---

## 📝 Description

Build a personal finance API to track income and expenses with categories, budgets, and analytics.

**What You'll Learn:**
- Financial data modeling
- Date-based queries
- Aggregation calculations
- Budget tracking
- Charts and analytics

---

## 🎯 Project Goals

- ✅ Track expenses and income
- ✅ Categorize transactions
- ✅ Set monthly budgets
- ✅ Generate spending reports
- ✅ React dashboard with charts

---

## 🗺️ Roadmap

### Step 1: Transaction Model
**Todo:**
- [ ] Create transaction structure:
  - id, type (income/expense)
  - amount, category, description
  - date, paymentMethod
  - tags, receipt (optional)

---

### Step 2: Categories System
**Todo:**
- [ ] Predefined categories: Food, Transport, Shopping, Bills, Salary, etc.
- [ ] CRUD for custom categories
- [ ] Each category has: name, color, icon, type (income/expense)

---

### Step 3: Transactions CRUD
**Todo:**
- [ ] `POST /api/transactions` - Add transaction
- [ ] `GET /api/transactions` - Get all
- [ ] `GET /api/transactions/:id` - Get one
- [ ] `PUT /api/transactions/:id` - Update
- [ ] `DELETE /api/transactions/:id` - Delete

---

### Step 4: Filtering & Queries
**Todo:**
- [ ] `GET /api/transactions?type=expense`
- [ ] `GET /api/transactions?category=Food`
- [ ] `GET /api/transactions?startDate=2025-01-01&endDate=2025-01-31`
- [ ] `GET /api/transactions?month=1&year=2025`

---

### Step 5: Analytics & Reports
**Todo:**
- [ ] `GET /api/analytics/summary` - Total income, expenses, balance
- [ ] `GET /api/analytics/by-category` - Spending per category
- [ ] `GET /api/analytics/monthly` - Month-wise breakdown
- [ ] `GET /api/analytics/trends` - Spending trends over time

**Calculations:**
- Total income
- Total expenses
- Current balance
- Average monthly expense
- Biggest expense category

---

### Step 6: Budget System
**Todo:**
- [ ] Set monthly budget per category
- [ ] `POST /api/budgets` - Create budget
- [ ] `GET /api/budgets/status` - Check if over budget
- [ ] Alert when 80% of budget used

**Budget Model:**
- category, month, year, limit
- spent (calculated from transactions)
- remaining

---

### Step 7: React Dashboard
**Todo:**
- [ ] Add transaction form
- [ ] Transactions list (latest first)
- [ ] Filter by date range, category, type
- [ ] Summary cards (income, expense, balance)
- [ ] Pie chart: expenses by category
- [ ] Line chart: monthly trends
- [ ] Budget progress bars

---

## 💡 Hints & Tips

### Date Filtering:
```javascript
transactions.filter(t => 
  new Date(t.date) >= startDate && 
  new Date(t.date) <= endDate
)
```

### Category Totals:
```javascript
const categoryTotals = transactions.reduce((acc, t) => {
  acc[t.category] = (acc[t.category] || 0) + t.amount;
  return acc;
}, {});
```

### Budget Check:
```javascript
const spent = transactions
  .filter(t => t.category === category && t.type === 'expense')
  .reduce((sum, t) => sum + t.amount, 0);
const percentage = (spent / budgetLimit) * 100;
```

### React Charts:
Use `recharts` or `chart.js` library

---

## 📚 Concepts

- Financial calculations
- Date manipulation
- Data aggregation
- Budget tracking
- Chart visualization

---

**Next:** [Project 14: Recipe Manager API](14-recipe-manager.md)
