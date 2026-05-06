# Reflection — DSO101 Assignment 1

**Student Name:** Samuel Tamang  
**Student ID:** 02240335  
**Course:** DSO101 — Continuous Integration and Continuous Deployment  
**Program:** Bachelor's of Engineering in Software Engineering  

---

## 1. What I Did

In this assignment, I built and deployed a full-stack Todo List application using Docker and Render.com. The application consists of three components — a React frontend, a Node.js/Express backend, and a PostgreSQL database. I completed two parts:

- **Part A:** Manually built Docker images, pushed them to Docker Hub, and deployed each service separately on Render using the pre-built images.
- **Part B:** Set up automated deployment by connecting Render to my GitHub repository using a `render.yaml` blueprint file, so every git push triggers a new deployment automatically.

---

## 2. What I Learned

### Docker and Containerization
I learned how to write Dockerfiles for both frontend and backend services. For the frontend I used a multi-stage build — first building the Vite app with Node, then serving the static files with Nginx. This reduced the final image size significantly. I also learned the importance of specifying `--platform linux/amd64` when building on an Apple Silicon Mac, because Render only accepts `linux/amd64` images.

### Environment Variables
I learned the difference between how Create React App and Vite handle environment variables. Vite requires the `VITE_` prefix and bakes the values into the build at compile time, not runtime. This meant I had to pass the API URL as a build argument or hardcode it in the Dockerfile before running `npm run build`.

### Render.com Deployment
I learned how to deploy Docker images on Render using both the manual image approach (Part A) and the Git-connected blueprint approach (Part B). The `render.yaml` file acts like a `docker-compose.yml` for Render, allowing multiple services to be defined and deployed together.

### CORS
I learned about Cross-Origin Resource Sharing (CORS) and why the backend needs to explicitly allow requests from the frontend domain. Without configuring CORS properly, the browser blocks API calls even if both services are running correctly.

### Debugging
I learned to debug deployment issues methodically — testing the backend API directly with `curl`, running the Docker image locally before deploying, and checking Render logs to identify the root cause of failures.

---

## 3. Challenges I Faced

### Challenge 1 — Platform Mismatch (linux/arm64 vs linux/amd64)
Since I am using a Mac with Apple Silicon (M-series chip), Docker builds images for `linux/arm64` by default. Render requires `linux/amd64`. This caused my images to be rejected when I tried to deploy them. I fixed this by adding the `--platform linux/amd64` flag to every `docker build` command.

### Challenge 2 — Vite Environment Variables Not Working
My frontend was not connecting to the backend even after setting `VITE_API_URL` on Render. I discovered that Vite bakes environment variables into the JavaScript bundle at build time. Since the Docker image was already built without the correct URL, setting the variable on Render had no effect at runtime. I fixed this by hardcoding the backend URL directly in the Dockerfile using `ENV VITE_API_URL=https://be-todo-02240335.onrender.com` before the `npm run build` step.

### Challenge 3 — Wrong Database Name
The initial error was `database "todo-db" does not exist`. Render auto-generates a database name different from what I expected. I had to go to the Render PostgreSQL dashboard, copy the exact database name (`todo_db_d50l`), and update the `DB_NAME` environment variable in the backend service.

### Challenge 4 — Node.js Version Too Old for Vite
My original Dockerfile used `node:18-alpine` but the version of Vite I installed requires Node.js 20+. The build failed with `Vite requires Node.js version 20.19+ or 22.12+`. I fixed this by changing the base image to `node:20-alpine`.

### Challenge 5 — Express Route Error with app.options
When I added `app.options('*', cors())` to fix CORS, the backend crashed with `PathError: Missing parameter name`. This was because the `'*'` wildcard is not valid as a route path in newer versions of Express. I removed that line and kept only `app.use(cors({...}))` which resolved the issue.

---

## 4. How I Solved the Problems

| Problem | Solution |
|---------|----------|
| Platform mismatch on Mac | Added `--platform linux/amd64` to docker build |
| Vite env vars not working | Hardcoded URL in Dockerfile with ENV before npm run build |
| Wrong database name | Copied exact DB_NAME from Render PostgreSQL dashboard |
| Node.js too old for Vite | Changed base image from node:18-alpine to node:20-alpine |
| Express route crash | Removed app.options line, kept only app.use(cors()) |
| .env committed to GitHub | Added .env to .gitignore and used git rm --cached |

---

## 5. What I Would Do Differently

- **Use Docker Compose locally** for development so frontend, backend, and database all run together with one command
- **Set up environment variables properly from the start** — understanding Vite's build-time variable baking before writing the Dockerfile would have saved a lot of debugging time
- **Test Docker images locally** before pushing to Docker Hub and deploying on Render — running `docker run` locally first would have caught most issues earlier
- **Use secrets management** instead of hardcoding database credentials in render.yaml

---

## 6. Key Takeaways

This assignment taught me that containerization is not just about writing a Dockerfile — it requires understanding the entire deployment pipeline from local development to production. The relationship between build-time and runtime environment variables, platform compatibility, and network configuration between services are all critical factors that must be considered when containerizing and deploying a real application.

The automated deployment in Part B demonstrated the power of CI/CD — by connecting GitHub to Render, every code change is automatically built and deployed without any manual steps, which is the foundation of modern software delivery practices.

---
