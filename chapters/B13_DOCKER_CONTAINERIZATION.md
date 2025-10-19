# B13: Docker & Containerization 🐳

> **Package Application: Run Anywhere with Docker**

**Chapter Level:** Advanced  
**Time Required:** 2-3 hours  
**Prerequisites:** Completed B01-B12  

---

## What is Docker?

```
PROBLEM: "Works on my machine"
- Different computers have different setups
- Environment differences cause bugs
- Hard to deploy consistently

SOLUTION: Docker
- Package entire app with dependencies
- Same container runs everywhere
- Consistent environment
```

---

## Setup

```bash
# Download Docker Desktop
# https://www.docker.com/products/docker-desktop

# Verify installation
docker --version
```

---

## Implementation

```dockerfile
# ============================================================
# FILE: Dockerfile
# PURPOSE: Define how to build Docker image
# ============================================================

# BASE IMAGE: Start with Node.js image
FROM node:18-alpine

# WORKDIR: Set working directory inside container
WORKDIR /app

# COPY: Copy package.json and package-lock.json
COPY package*.json ./

# RUN: Install dependencies
RUN npm install

# COPY: Copy entire app code
COPY . .

# EXPOSE: Expose port
EXPOSE 5000

# ENV: Set environment variables
ENV NODE_ENV=production

# CMD: Command to start app
CMD ["node", "server.js"]
```

---

## Docker Compose

```yaml
# ============================================================
# FILE: docker-compose.yml
# PURPOSE: Run multiple containers (app + database)
# ============================================================

version: '3.8'

services:
  # MongoDB Database
  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongo_data:/data/db

  # Node.js Backend
  backend:
    build: .
    ports:
      - "5000:5000"
    environment:
      MONGO_URI: mongodb://root:password@mongodb:27017/myapp
      NODE_ENV: production
    depends_on:
      - mongodb
    volumes:
      - .:/app
      - /app/node_modules

volumes:
  mongo_data:
```

---

## Usage

```bash
# BUILD IMAGE
docker build -t myapp:1.0 .

# RUN CONTAINER
docker run -p 5000:5000 myapp:1.0

# RUN WITH DOCKER COMPOSE
docker-compose up

# STOP CONTAINER
docker-compose down

# VIEW LOGS
docker logs container_id

# CONNECT TO CONTAINER
docker exec -it container_id bash
```

---

| Previous | Index | Next |
|----------|-------|------|
| [B12: Complete Project](B12_COMPLETE_PROJECT.md) | [BEGINNER_INDEX.md](../BEGINNER_INDEX.md) | [B14: Testing & QA](B14_TESTING_AND_QA.md) |
