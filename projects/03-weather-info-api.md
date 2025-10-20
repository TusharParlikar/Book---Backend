# Project 3: Weather Information API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 2-3 hours  
**Related Chapter:** [B02 - Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

---

## 📝 Description

Build a weather API that serves weather information for different cities. Learn to work with external APIs, environment variables, and data transformation.

**What You'll Learn:**
- Calling external APIs (OpenWeatherMap)
- Environment variables for API keys
- Data transformation
- Error handling for external services
- Caching responses

---

## 🎯 Project Goals

- ✅ Fetch weather from OpenWeatherMap API
- ✅ Transform data to clean format
- ✅ Cache responses for 10 minutes
- ✅ Handle errors gracefully
- ✅ React app shows weather with icons

---

## 🗺️ Roadmap

### Step 1: Setup & Get API Key
**Todo:**
- [ ] Sign up at [OpenWeatherMap](https://openweathermap.org/api)
- [ ] Get free API key
- [ ] Store in `.env` file: `WEATHER_API_KEY=your_key`
- [ ] Install: `axios` or `node-fetch` for API calls
- [ ] Install: `dotenv`

**Hint:** Never commit API keys to GitHub. Always use `.env` and add it to `.gitignore`

---

### Step 2: Create Weather Service
**Todo:**
- [ ] Create `services/weatherService.js`
- [ ] Write function to call OpenWeatherMap API
- [ ] Function accepts city name
- [ ] Returns weather data or error
- [ ] Test with console.log

**API Endpoint Structure:**
```
https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric
```

**Hint:** Use axios.get() or fetch() to make the request

---

### Step 3: Create Routes
**Todo:**
- [ ] `GET /api/weather/:city` - Get weather for specific city
- [ ] `GET /api/weather/current?lat=&lon=` - Get by coordinates
- [ ] `GET /api/weather/forecast/:city` - 5-day forecast

**Hint:** Extract city from `req.params.city`

---

### Step 4: Transform Data
**Todo:**
- [ ] Extract only useful information:
  - Temperature (current, feels like, min, max)
  - Weather description
  - Humidity
  - Wind speed
  - City name
  - Country
  - Icon code
- [ ] Create clean response object
- [ ] Remove unnecessary fields from external API

**Hint:** OpenWeatherMap response is huge. You only need a few fields.

---

### Step 5: Add Caching
**Todo:**
- [ ] Create simple cache object in memory
- [ ] Cache format: `{ cityName: { data, timestamp } }`
- [ ] Before API call, check cache
- [ ] If data exists and < 10 minutes old, return cached data
- [ ] Otherwise, fetch fresh data and update cache

**Hint:** Use `Date.now()` to compare timestamps

**Why cache?** API has rate limits. Caching reduces API calls and speeds up response.

---

### Step 6: Error Handling
**Todo:**
- [ ] Handle city not found (404)
- [ ] Handle API key invalid
- [ ] Handle network errors
- [ ] Return proper status codes and messages
- [ ] Log errors for debugging

**Common Errors:**
- City name spelling wrong
- API key expired
- Rate limit exceeded
- No internet connection

---

### Step 7: React Frontend
**Todo:**
- [ ] Create search bar for city input
- [ ] Fetch weather on search
- [ ] Display:
  - City name and country
  - Weather icon
  - Temperature
  - Description
  - Humidity and wind
- [ ] Add loading spinner
- [ ] Show error messages nicely
- [ ] Save recent searches

**Hint:** OpenWeatherMap provides icon codes. Use them:
```
http://openweathermap.org/img/wn/{icon}@2x.png
```

---

## 💡 Hints & Tips

### Making External API Calls:
```javascript
const axios = require('axios');
const response = await axios.get(API_URL);
const data = response.data;
```

### Caching Logic:
```
Check if city exists in cache
  → Check if timestamp is < 10 minutes old
    → Return cached data
  → Else fetch new data
```

### Temperature Conversion:
- API gives Celsius by default (units=metric)
- For Fahrenheit: units=imperial
- For Kelvin: don't specify units

### React Search:
- Use controlled input (value + onChange)
- Fetch on form submit, not on every keystroke
- Show recent searches from localStorage

---

## 🎨 Enhancement Ideas

**Easy:**
- Add popular cities quick search
- Show weather emoji based on conditions
- Add "feels like" temperature

**Medium:**
- 5-day forecast display
- Hourly forecast
- Weather alerts
- Multiple cities comparison

**Advanced:**
- Geolocation to auto-detect city
- Weather maps
- Historical weather data
- Weather-based recommendations

---

## ✅ Completion Checklist

**Backend:**
- [ ] External API integration working
- [ ] Environment variables secure
- [ ] Caching implemented
- [ ] Error handling complete
- [ ] Clean data transformation
- [ ] Rate limiting considered

**Frontend:**
- [ ] Search functionality works
- [ ] Weather displays correctly
- [ ] Icons show properly
- [ ] Loading states
- [ ] Error messages clear
- [ ] Responsive design

---

## 📚 Concepts You'll Practice

- External API integration
- Environment variables
- Data transformation
- Caching strategies
- Error handling
- Async/await
- React form handling
- localStorage

---

## 🔗 Related Resources

**Chapters:**
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md)

**External:**
- [OpenWeatherMap API Docs](https://openweathermap.org/api)
- [Axios Documentation](https://axios-http.com/)
- [dotenv Package](https://www.npmjs.com/package/dotenv)

---

**Next:** [Project 4: Todo List API](04-todo-list-api.md)
