# DSO101 — Assignment 1: CI/CD with Docker and Render

**Student Name:** Samuel Tamang  
**Student ID:** 02240335  
**Course:** DSO101 — Continuous Integration and Continuous Deployment  
**Program:** Bachelor's of Engineering in Software Engineering  
**GitHub Repository:** https://github.com/Waiba334/A1_DSO101

---

## Table of Contents

1. [Application Overview](#application-overview)
2. [Step 0 — Building the Todo Application](#step-0--building-the-todo-application)
3. [Part A — Docker Hub and Render Deployment](#part-a--docker-hub-and-render-deployment)
4. [Part B — Automated Deployment from GitHub](#part-b--automated-deployment-from-github)
5. [Live URLs](#live-urls)

---

## Application Overview

This project is a full-stack Todo List web application built with the following tech stack:

- **Frontend:** React (Vite) — UI for adding, editing, deleting, and completing tasks
- **Backend:** Node.js + Express — REST API with CRUD operations
- **Database:** PostgreSQL — persistent storage for todos

The application supports:
- Adding tasks with a title and optional description
- Marking tasks as complete/incomplete
- Editing existing tasks
- Deleting tasks
- Displaying total, done, and pending task counts

---

## Step 0 — Building the Todo Application

### Project Structure

```
todo-app/
├── frontend/
│   ├── src/
│   │   └── App.jsx
│   ├── Dockerfile
│   ├── .env
│   ├── .gitignore
│   └── vite.config.js
├── backend/
│   ├── server.js
│   ├── Dockerfile
│   ├── .env
│   └── .gitignore
└── render.yaml
```

### Backend — Environment Variables (.env)

```
DB_HOST=your-db-host
DB_PORT=5432
DB_USER=your-db-user
DB_PASSWORD=your-db-password
DB_NAME=your-db-name
PORT=5000
```

### Frontend — Environment Variables (.env)

```
VITE_API_URL=http://localhost:5000
```

---

### Screenshot 1 — Project Structure in VS Code

```
[ INSERT SCREENSHOT: VS Code showing the full folder structure with backend/, frontend/, render.yaml ]
```

---

### Screenshot 2 — Application Running Locally (Frontend)

![Screenshot: Browser showing the Todo app UI](./images/Screenshot%202026-04-29%20at%209.10.42%20PM.png)

---

### Screenshot 3 — Backend Running Locally

![Backend Running Locally](./images/Screenshot%202026-04-29%20at%209.26.06%20PM.png)

---

## Part A — Docker Hub and Render Deployment

### Step 1 — Writing the Dockerfiles

**Backend Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
```

**Frontend Dockerfile:**
```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV VITE_API_URL=https://be-todo-02240335.onrender.com
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### Screenshot 4 — Backend Dockerfile

![Backend Dockerfile](<images/Screenshot 2026-04-29 at 9.29.50 PM.png>)

---

### Screenshot 5 — Frontend Dockerfile

![Frontend Dockerfile](<images/Screenshot 2026-04-29 at 9.31.10 PM.png>)
---

### Step 2 — Building Docker Images

```bash
# Backend
cd backend
docker build --platform linux/amd64 -t bishalwaiba/be-todo:02240335 .

# Frontend
cd frontend
docker build --platform linux/amd64 -t bishalwaiba/fe-todo:02240335 .
```

---

### Screenshot 6 — Backend Docker Build Success

![Backend Docker Build](<images/Screenshot 2026-04-15 at 9.56.19 PM.png>)
---

### Screenshot 7 — Frontend Docker Build Success

![Frontend Docker Build](<images/Screenshot 2026-04-15 at 9.55.09 PM.png>)
---

### Step 3 — Pushing Images to Docker Hub

```bash
docker login
docker push bishalwaiba/be-todo:02240335
docker push bishalwaiba/fe-todo:02240335
```

---


### Screenshot 8 — Backend Image Pushed

![Backend Image Pushed](<images/Screenshot 2026-04-15 at 9.56.04 PM.png>)
---

### Screenshot 9 — Frontend Image Pushed

![Frontend Image Pushed](<images/Screenshot 2026-04-15 at 9.55.30 PM.png>)
---

### Screenshot 10 — Docker Hub Repositories

![Docker Hub Repo](<images/Screenshot 2026-03-27 at 2.48.34 PM.png>)
---


### Step 4 — Setting Up PostgreSQL on Render

1. Logged into Render.com
2. New → PostgreSQL → named `todo-db`
3. Copied credentials from Connections tab

---

### Screenshot 11 — Render PostgreSQL Created

![Render](<images/Screenshot 2026-04-29 at 9.43.25 PM.png>)
---

### Screenshot 12 — Render PostgreSQL Connection Details

![Render psql](<images/Screenshot 2026-04-29 at 9.43.54 PM.png>)
---

### Step 5 — Deploying Backend on Render

1. New → Web Service → Existing Image
2. Image URL: `bishalwaiba/be-todo:02240335`
3. Added all DB environment variables

**Environment variables configured:**

| Key | Value |
|-----|-------|
| DB_HOST | dpg-d7mb3868bjmc7388gpj0-a |
| DB_PORT | 5432 |
| DB_USER | todo_db_d50l_user |
| DB_PASSWORD | (hidden) |
| DB_NAME | todo_db_d50l |
| PORT | 5000 |

---


### Screenshot 14 — Backend Environment Variables on Render

![Backend Env](<images/Screenshot 2026-04-29 at 9.48.37 PM.png>)

---


### Screenshot 19 — Backend API Working in Browser

![alt text](<images/Screenshot 2026-04-29 at 3.40.11 PM.png>)

---

### Step 6 — Deploying Frontend on Render

1. New → Web Service → Existing Image
2. Image URL: `bishalwaiba/fe-todo:latest`
3. Added VITE_API_URL environment variable

---


### Screenshot 20 — Frontend Environment Variables on Render

![Frontend Env](<images/Screenshot 2026-04-29 at 9.52.21 PM.png>)

---

### Screenshot 21 — Deployed Successfully

![Deployed Successfully](<images/Screenshot 2026-04-29 at 9.53.05 PM.png>)

---


### Screenshot 22 — Live Frontend Application

![alt text](<images/Screenshot 2026-04-29 at 9.55.34 PM.png>)

---


### Screenshot 23 — Task Confirmed in Backend API

![alt text](<images/Screenshot 2026-04-29 at 9.56.24 PM.png>)

---

## Part B — Automated Deployment from GitHub

### Step 1 — render.yaml Configuration

```yaml
services:
  - type: web
    name: be-todo
    env: docker
    dockerfilePath: ./backend/Dockerfile
    envVars:
      - key: DB_HOST
        value: dpg-d7mb3868bjmc7388gpj0-a
      - key: DB_PORT
        value: 5432
      - key: DB_USER
        value: todo_db_d50l_user
      - key: DB_PASSWORD
        value: ohxtdmfkDOYdK8Lv6QAd8lv1Rh5LIwHi
      - key: DB_NAME
        value: todo_db_d50l
      - key: PORT
        value: 5000

  - type: web
    name: fe-todo
    env: docker
    dockerfilePath: ./frontend/Dockerfile
    envVars:
      - key: VITE_API_URL
        value: https://be-todo-02240335.onrender.com
```

---

### Screenshot 24 — render.yaml in VS Code

![alt text](<images/Screenshot 2026-04-29 at 9.57.05 PM.png>)

---

### Step 2 — Pushing Code to GitHub

```bash
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/Waiba334/A1_DSO101.git
git push -u origin main
```

---

### Screenshot 25 — GitHub Repository with All Files

![Github Repo](<images/Screenshot 2026-04-29 at 9.58.11 PM.png>)

---

### Step 3 — Connecting Render Blueprint to GitHub

1. Render → New → Blueprint
2. Connected GitHub account
3. Selected A1_DSO101 repository
4. Render auto-detected render.yaml

---

### Screenshot 26 — Render Blueprint Connecting to GitHub

![alt text](<images/Screenshot 2026-05-01 at 10.30.09 AM.png>)

![alt text](<images/Screenshot 2026-05-01 at 10.32.52 AM.png>)

---

### Screenshot 27 — Auto Deploy Triggered by Git Push

![alt text](<images/Screenshot 2026-05-01 at 10.39.36 AM.png>)

![alt text](<images/Screenshot 2026-05-01 at 10.39.48 AM.png>)

---

### Screenshot 28 — All Services Auto Deployed via Blueprint

![alt text](<images/Screenshot 2026-05-01 at 10.37.23 AM.png>)

---

## Live URLs

| Service | URL |
|---------|-----|
| Frontend | https://fe-todo-latest.onrender.com |
| Backend API | https://be-todo-02240335.onrender.com/todos |
| Backend /todos endpoint | https://be-todo-02240335.onrender.com/todos |
| Docker Hub | https://hub.docker.com/u/bishalwaiba |
| GitHub Repo | https://github.com/Waiba334/A1_DSO101 |
