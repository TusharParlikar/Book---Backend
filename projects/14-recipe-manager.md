# Project 14: Recipe Manager API

**Difficulty:** 🟡 Medium  
**Estimated Time:** 3-4 hours  
**Related Chapter:** [B04 - MongoDB Basics](../chapters/B04_MONGODB_BASICS.md), [B05 - Mongoose Schemas](../chapters/B05_MONGOOSE_SCHEMAS.md)

---

## 📝 Description

Build a recipe management API with ingredients, instructions, ratings, and meal planning.

**What You'll Learn:**
- Complex data schemas
- Nested arrays (ingredients, steps)
- Rating systems
- Search and filtering
- Meal planning logic

---

## 🎯 Project Goals

- ✅ Store recipes with ingredients and steps
- ✅ Rate and review recipes
- ✅ Search by ingredients
- ✅ Filter by cuisine, diet, time
- ✅ React recipe app with search and filters

---

## 🗺️ Roadmap

### Step 1: Recipe Model
**Todo:**
- [ ] Create recipe structure:
  - title, description, image
  - prepTime, cookTime, servings
  - difficulty (easy, medium, hard)
  - cuisine, diet (vegetarian, vegan, etc.)
  - ingredients array:
    - name, quantity, unit
  - instructions array (step-by-step)
  - tags, ratings, reviews

---

### Step 2: Recipe CRUD
**Todo:**
- [ ] `POST /api/recipes` - Create recipe
- [ ] `GET /api/recipes` - Get all
- [ ] `GET /api/recipes/:id` - Get single
- [ ] `PUT /api/recipes/:id` - Update
- [ ] `DELETE /api/recipes/:id` - Delete

---

### Step 3: Search & Filter
**Todo:**
- [ ] `GET /api/recipes?search=pasta` - Search in title/description
- [ ] `GET /api/recipes?cuisine=Italian`
- [ ] `GET /api/recipes?diet=vegan`
- [ ] `GET /api/recipes?maxTime=30` - Quick recipes
- [ ] `GET /api/recipes?difficulty=easy`

---

### Step 4: Ingredient-Based Search
**Todo:**
- [ ] `POST /api/recipes/search-by-ingredients`
  - Body: { ingredients: ['chicken', 'rice'] }
- [ ] Return recipes that contain all OR any of these ingredients
- [ ] Sort by number of matching ingredients

**Algorithm:**
```
Loop through recipes
Count how many ingredients match
Sort by match count
Return sorted list
```

---

### Step 5: Rating System
**Todo:**
- [ ] `POST /api/recipes/:id/rate`
  - Body: { rating: 4.5, review: 'Delicious!' }
- [ ] Calculate average rating
- [ ] Store all reviews
- [ ] `GET /api/recipes/:id/reviews` - Get all reviews

**Average Calculation:**
```
totalRating = sum of all ratings
avgRating = totalRating / numberOfRatings
```

---

### Step 6: Meal Planning
**Todo:**
- [ ] `POST /api/meal-plan` - Create weekly plan
  - Body: { monday: [recipeIds], tuesday: [...], ... }
- [ ] `GET /api/meal-plan/shopping-list` - Generate list from week's recipes
- [ ] Combine ingredients from multiple recipes

---

### Step 7: React Frontend
**Todo:**
- [ ] Recipe card grid
- [ ] Recipe detail page
- [ ] Add recipe form
- [ ] Search bar
- [ ] Filter sidebar (cuisine, diet, time)
- [ ] Rating stars
- [ ] Meal planner calendar

---

## 💡 Hints & Tips

### Ingredient Search:
```javascript
const hasIngredient = recipe.ingredients.some(ing => 
  ing.name.toLowerCase().includes(searchTerm.toLowerCase())
);
```

### Shopping List Generation:
```javascript
// Combine ingredients from multiple recipes
// Group by ingredient name
// Sum quantities
```

### Recipe Time Filter:
```javascript
const totalTime = recipe.prepTime + recipe.cookTime;
if (totalTime <= maxTime) { /* include */ }
```

---

## 📚 Concepts

- Nested arrays in schemas
- Complex searching
- Rating calculations
- Data aggregation
- Meal planning logic

---

**Next:** [Project 15: Task Scheduler API](15-task-scheduler.md)
