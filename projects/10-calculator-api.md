# Project 10: Calculator API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 1-2 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build a REST API for mathematical calculations with history tracking.

**What You'll Learn:**
- Expression parsing
- Math operations API
- Input validation
- Calculation history
- Error handling

---

## 🎯 Project Goals

- ✅ Basic operations (+ - × ÷)
- ✅ Advanced operations (power, square root, percentage)
- ✅ Expression evaluation
- ✅ History tracking
- ✅ React calculator UI

---

## 🗺️ Roadmap

### Step 1: Basic Operations
**Todo:**
- [ ] `GET /api/calc/add?a=5&b=3` → 8
- [ ] `GET /api/calc/subtract?a=5&b=3` → 2
- [ ] `GET /api/calc/multiply?a=5&b=3` → 15
- [ ] `GET /api/calc/divide?a=6&b=3` → 2
- [ ] Validate: numbers only, no division by zero

---

### Step 2: Advanced Operations
**Todo:**
- [ ] Power: `a^b`
- [ ] Square root: `√a`
- [ ] Percentage: `a% of b`
- [ ] Modulo: `a % b`

---

### Step 3: Expression Evaluation
**Todo:**
- [ ] `POST /api/calc/evaluate`
- [ ] Body: `{ expression: "2 + 3 * 4" }`
- [ ] Parse and evaluate correctly
- [ ] Handle parentheses
- [ ] Return result

**Hint:** Use `eval()` carefully (security risk) or write a parser

---

### Step 4: History System
**Todo:**
- [ ] Save all calculations
- [ ] `GET /api/calc/history` - Get all
- [ ] `DELETE /api/calc/history` - Clear history
- [ ] Include: expression, result, timestamp

---

### Step 5: React Calculator
**Todo:**
- [ ] Calculator button UI
- [ ] Display screen
- [ ] Number buttons (0-9)
- [ ] Operation buttons
- [ ] Equals button
- [ ] Clear button
- [ ] History panel

---

## 💡 Hints & Tips

### Safe Expression Evaluation:
Use `math.js` library instead of `eval()`

### Division by Zero:
```javascript
if (b === 0) {
  return res.status(400).json({ error: 'Division by zero' });
}
```

---

**Next:** [Project 11: Random User Generator](11-random-user-generator.md)
