# Chapter 14: Professional Project Setup

## 14.1 Project Initialization and Git Setup

```bash
# Create project
mkdir professional-backend
cd professional-backend
git init
npm init -y
```

## 14.2 .gitignore Configuration

```
node_modules/
.env
.DS_Store
dist/
*.log
npm-debug.log*
```

## 14.3 Tracking Empty Folders

```bash
touch db/.gitkeep
touch models/.gitkeep
touch controllers/.gitkeep
touch routes/.gitkeep
touch middlewares/.gitkeep
touch utils/.gitkeep
touch public/temp/.gitkeep
```

## 14.4 Environment Variables Management

Create `.env`:
```
MONGODB_URI=mongodb://localhost:27017
PORT=5000
NODE_ENV=development
JWT_SECRET=your-secret-key
```

## 14.5 Code Quality Tools

### Install nodemon

```bash
npm install --save-dev nodemon
```

Update package.json:
```json
{
  "scripts": {
    "dev": "nodemon server.js"
  }
}
```

### Install prettier

```bash
npm install --save-dev prettier
```

Create `.prettierrc`:
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2
}
```

---

## 🎯 Next: Chapter 15 - Error Handling and API Responses
