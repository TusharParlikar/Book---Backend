# Project 1: Personal Portfolio API

**Difficulty:** 🟢 Easy  
**Estimated Time:** 1-2 hours  
**Related Chapter:** [B01 - What is Backend?](../chapters/B01_WHAT_IS_BACKEND.md)

---

## 📝 Description

Build a backend API that serves your personal information, skills, projects, and experience. This API will power your portfolio website where React fetches and displays all your data dynamically.

**What You'll Learn:**
- Setting up Express server
- Creating GET routes
- Structuring JSON responses
- Handling CORS for React frontend
- Organizing data in objects

---

## 🎯 Project Goals

By the end of this project, you will have:
- ✅ A running Express API server
- ✅ Multiple GET endpoints returning your data
- ✅ A React app displaying the portfolio
- ✅ Clean JSON response structure
- ✅ Error handling for invalid routes

---

## 🗺️ Roadmap

### Step 1: Initialize Project
**Todo:**
- [ ] Create new folder `portfolio-api`
- [ ] Run `npm init -y`
- [ ] Install: `express`, `cors`, `dotenv`
- [ ] Create `server.js` file
- [ ] Create `.env` and `.gitignore` files

**Hint:** Start with basic Express setup - server listening on port 5000

---

### Step 2: Create Data Structure
**Todo:**
- [ ] Create `data` folder
- [ ] Create `portfolio.js` file inside data folder
- [ ] Structure your data with these objects:
  - Personal info (name, title, bio, email, location)
  - Skills array (name, level, years of experience)
  - Projects array (title, description, technologies, links)
  - Experience array (company, position, duration, description)
  - Social links (github, linkedin, twitter)

**Hint:** Use JavaScript objects and arrays. Think about what information a recruiter would want to see.

**Reference:** [JavaScript Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)

---

### Step 3: Create API Routes
**Todo:**
- [ ] Create `routes` folder
- [ ] Create `portfolio.js` route file
- [ ] Implement these endpoints:
  - `GET /api/portfolio` - Return all data
  - `GET /api/portfolio/personal` - Return personal info only
  - `GET /api/portfolio/skills` - Return skills array
  - `GET /api/portfolio/projects` - Return projects array
  - `GET /api/portfolio/projects/:id` - Return single project
  - `GET /api/portfolio/experience` - Return experience array
  - `GET /api/portfolio/social` - Return social links

**Hint:** Use `router.get()` for each endpoint. Import your data and send it in response.

**Reference:** [Express Routing](https://expressjs.com/en/guide/routing.html)

---

### Step 4: Add Middleware
**Todo:**
- [ ] Enable CORS (so React can fetch)
- [ ] Add `express.json()` middleware
- [ ] Create error handling middleware
- [ ] Add 404 handler for unknown routes

**Hint:** CORS is essential for React to communicate with your backend during development.

**Reference:** [Chapter B01 - Middleware Section](../chapters/B01_WHAT_IS_BACKEND.md)

---

### Step 5: Test API
**Todo:**
- [ ] Start server with `node server.js`
- [ ] Test all routes in browser
- [ ] Test with Postman (download if needed)
- [ ] Verify JSON structure is correct
- [ ] Check error handling works

**Hint:** Open `http://localhost:5000/api/portfolio` in browser to see response.

---

### Step 6: Build React Frontend
**Todo:**
- [ ] Create React app (`npx create-react-app portfolio-frontend`)
- [ ] Create component `Portfolio.jsx`
- [ ] Use `fetch()` to get data from API
- [ ] Display personal info (name, title, bio)
- [ ] Map through skills and display them
- [ ] Map through projects and display them
- [ ] Add loading state
- [ ] Add error handling

**Hint:** Use `useEffect` to fetch data when component mounts. Use `useState` for data storage.

**React Integration Example Structure:**
```
Component mounts 
  → fetch('/api/portfolio') 
  → Display loading
  → Data arrives
  → Update state
  → Render data
```

---

### Step 7: Style and Deploy
**Todo:**
- [ ] Add basic CSS styling
- [ ] Make it responsive
- [ ] Test on different screen sizes
- [ ] Add social media links
- [ ] Deploy backend (Railway/Render)
- [ ] Deploy frontend (Vercel/Netlify)

**Hint:** Keep it simple first, improve design later.

---

## 💡 Hints & Tips

### If you're stuck on Express setup:
- Remember: `const app = express()` creates the server
- `app.listen(PORT)` starts it
- `app.use()` adds middleware
- `app.get()` creates routes

### If React can't fetch data:
- Check CORS is enabled on backend
- Check backend URL is correct
- Open browser console for errors
- Verify backend is running

### If data isn't displaying:
- Console.log the fetched data
- Check if state is updating
- Verify you're mapping arrays correctly

---

## 🎨 Enhancement Ideas

Once you complete the basic project:

**Easy Additions:**
- Add education section
- Add certifications
- Add testimonials from clients
- Add blog posts preview

**Medium Additions:**
- Filter projects by technology
- Search functionality
- Dark mode toggle
- View counter for projects

**Advanced Additions:**
- Admin panel to update data
- Connect to MongoDB
- Add file upload for images
- Add contact form

---

## ✅ Completion Checklist

Mark these off as you complete them:

**Backend:**
- [ ] Express server running on port 5000
- [ ] All 7 GET endpoints working
- [ ] CORS enabled
- [ ] Returns proper JSON
- [ ] Error handling in place
- [ ] Tested in Postman

**Frontend:**
- [ ] React app created
- [ ] Fetches data from backend
- [ ] Displays personal info
- [ ] Displays skills list
- [ ] Displays projects list
- [ ] Shows loading state
- [ ] Shows errors if API fails
- [ ] Looks presentable

**Integration:**
- [ ] Frontend and backend communicate
- [ ] No CORS errors
- [ ] Data displays correctly
- [ ] No console errors

---

## 📚 Concepts You'll Practice

- Express server setup
- RESTful API design
- Route creation
- JSON responses
- CORS handling
- React data fetching
- useState and useEffect hooks
- Array mapping in React
- Error handling (both backend and frontend)

---

## 🔗 Related Resources

**Chapters:**
- [B01: What is Backend?](../chapters/B01_WHAT_IS_BACKEND.md) - Server basics
- [B02: Express & Routing](../chapters/B02_EXPRESS_AND_ROUTING.md) - Routing in detail

**External Resources:**
- [Express.js Docs](https://expressjs.com)
- [React Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [CORS Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

---

## 🚀 Next Steps

After completing this project:
1. Move to [Project 2: Quote of the Day API](02-quote-of-day-api.md)
2. Try deploying this API online
3. Add it to your actual portfolio website
4. Share it with potential employers!

---

**Ready to code? Follow the roadmap step-by-step and build your first backend API! 🎉**
