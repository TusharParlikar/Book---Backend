# Project 2: Quote of the Day API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 1-2 hours  
**Related Chapter:** [B01 - What is Backend?](../chapters/B01_WHAT_IS_BACKEND.md), [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Create an API that serves inspirational quotes. Users can get a random quote, quotes by category, or all quotes. This is perfect for learning random selection logic and query parameters.

**What You'll Learn:**
- Working with arrays of data
- Random selection algorithms
- Query parameters
- Filtering data
- Serving JSON responses

---

## 🎯 Project Goals

- ✅ Store 50+ quotes in different categories
- ✅ Return random quote
- ✅ Filter by category, author, or tags
- ✅ React frontend displays quotes beautifully
- ✅ Add "favorite" functionality

---

## 🗺️ Roadmap

### Step 1: Project Setup
**Todo:**
- [ ] Initialize npm project
- [ ] Install express, cors, dotenv
- [ ] Create folder structure: routes, data, utils
- [ ] Setup basic Express server

---

### Step 2: Create Quotes Database
**Todo:**
- [ ] Create `data/quotes.js`
- [ ] Structure each quote object:
  - id (unique number)
  - text (the quote)
  - author (who said it)
  - category (motivational, success, life, etc.)
  - tags (array of keywords)
- [ ] Add at least 50 quotes in 5 categories

**Hint:** Use unique IDs starting from 1. Categories: motivational, success, life, wisdom, humor

**Where to get quotes:** [Type.fit Quotes API](https://type.fit/api/quotes) for inspiration

---

### Step 3: Create Routes
**Todo:**
- [ ] `GET /api/quotes` - Return all quotes
- [ ] `GET /api/quotes/random` - Return 1 random quote
- [ ] `GET /api/quotes/daily` - Return same quote for entire day
- [ ] `GET /api/quotes/:id` - Return specific quote by ID
- [ ] `GET /api/quotes/category/:category` - Filter by category
- [ ] `GET /api/quotes/author/:author` - Filter by author
- [ ] `GET /api/quotes/search?q=keyword` - Search in text

**Hint:** For random quote, use `Math.random()` and array length

---

### Step 4: Implement Logic
**Todo:**
- [ ] Write function to get random quote
- [ ] Write function for "quote of the day" (consistent for 24 hours)
- [ ] Write filter function for categories
- [ ] Write search function (case-insensitive)
- [ ] Handle "not found" errors

**Hint for Daily Quote:** Use current date as seed for random selection. Same date = same quote.

**Algorithm Hint:**
```
Get today's date → Convert to number → Use as index (with modulo)
```

---

### Step 5: Add Query Parameters
**Todo:**
- [ ] Support `?limit=5` to limit results
- [ ] Support `?random=true` to randomize order
- [ ] Support `?category=motivational`
- [ ] Support multiple filters together

**Hint:** Access query params with `req.query.paramName`

**Reference:** [Express Query Parameters](https://expressjs.com/en/api.html#req.query)

---

### Step 6: React Frontend
**Todo:**
- [ ] Create React app
- [ ] Create `QuoteDisplay.jsx` component
- [ ] Fetch random quote on button click using `async/await`
- [ ] Display quote text and author
- [ ] Add category filter dropdown
- [ ] Add "Get New Quote" button
- [ ] Add loading and error states

**React Integration Example:**
```jsx
import React, { useState, useEffect } from 'react';

function QuoteDisplay() {
  const [quote, setQuote] = useState(null);
  const [loading, setLoading] = useState(true);

  const fetchRandomQuote = async () => {
    setLoading(true);
    try {
      const response = await fetch('http://localhost:5000/api/quotes/random');
      if (!response.ok) {
        throw new Error('Failed to fetch quote');
      }
      const data = await response.json();
      setQuote(data);
    } catch (error) {
      console.error(error);
      setQuote({ text: 'Could not fetch a quote. Please try again.', author: 'System' });
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchRandomQuote();
  }, []);

  return (
    <div>
      {loading ? (
        <p>Loading...</p>
      ) : (
        <blockquote>
          <p>"{quote.text}"</p>
          <footer>- {quote.author}</footer>
        </blockquote>
      )}
      <button onClick={fetchRandomQuote} disabled={loading}>
        Get New Quote
      </button>
    </div>
  );
}

export default QuoteDisplay;
```

---

### Step 7: Advanced Features
**Todo:**
- [ ] Add "Share on Twitter" button
- [ ] Add "Copy to Clipboard" button
- [ ] Add favorites (store in localStorage)
- [ ] Show quote count by category
- [ ] Add transition animation when quote changes

**Hint:** Twitter share URL: `https://twitter.com/intent/tweet?text=QUOTE_HERE`

---

## 💡 Hints & Tips

### Random Quote Logic:
```
randomIndex = Math.floor(Math.random() * quotes.length)
randomQuote = quotes[randomIndex]
```

### Quote of the Day Logic:
- Get today's date as string: "2025-01-26"
- Convert to number somehow (hash it)
- Use modulo operator: `index = hashValue % quotes.length`
- This ensures same quote all day

### Filtering by Category:
```
Filter quotes array where quote.category === requestedCategory
```

### Case-insensitive Search:
```
Convert both search term and quote text to lowercase before comparing
```

---

## 🎨 Enhancement Ideas

**Easy:**
- Add more categories
- Add quote ratings
- Count how many times each quote shown

**Medium:**
- User submissions (POST route)
- Vote up/down quotes
- Most popular quotes endpoint

**Advanced:**
- Connect to MongoDB
- User authentication
- Save favorite quotes per user
- Daily email with quote

---

## ✅ Completion Checklist

**Backend:**
- [ ] All routes working
- [ ] Random quote truly random
- [ ] Daily quote consistent
- [ ] Filters working correctly
- [ ] Search functionality works
- [ ] Proper error handling

**Frontend:**
- [ ] Displays quotes beautifully
- [ ] Random quote button works
- [ ] Category filter works
- [ ] Smooth transitions
- [ ] Share buttons work
- [ ] No console errors

---

## 📚 Concepts You'll Practice

- Array manipulation
- Random selection algorithms
- Date-based logic
- Query parameters
- Filtering and searching
- React state management
- localStorage usage

---

## 🔗 Related Resources

**Chapters:**
- [B01: What is Backend?](../chapters/B01_WHAT_IS_BACKEND.md)
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

**External:**
- [Array.filter()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
- [Math.random()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)

---

**Next:** [Project 3: Weather Info API](03-weather-info-api.md)
