# Module 10 Assignment: Dockerize and Deploy Express.js App with Docker Compose + Nginx

## Overview

This project containerizes the Express.js app from [roy35-909/Module-3-deployment](https://github.com/roy35-909/Module-3-deployment) and runs it alongside Nginx using Docker Compose, exposed on port **8080**.

---

## Project Structure

After completing this assignment, your repository should look like this:

```
Module-3-deployment/
├── src/
│   └── server.js
├── test/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── package.json
└── README.md
```

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/roy35-909/Module-3-deployment.git
cd Module-3-deployment
```

> 📸 **Screenshot required:** Terminal showing the `git clone` command and the cloned directory listing (`ls -la`).

---

## Step 2: Create the Dockerfile

Create a file named `Dockerfile` in the root of the project:

```dockerfile
# Use official Node.js 22 image as base
FROM node:22-alpine

# Set working directory inside the container
WORKDIR /app

# Copy package files first (for layer caching)
COPY package*.json ./

# Install production dependencies only
RUN npm install --omit=dev

# Copy the rest of the application source
COPY . .

# Expose the app port
EXPOSE 3000

# Start the Express server
CMD ["node", "src/server.js"]
```

> 📸 **Screenshot required:** The `Dockerfile` open in your editor or shown via `cat Dockerfile`.

---

## Step 3: Create the Nginx Configuration

Create a file named `nginx.conf` in the root of the project:

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 8080;

        location / {
            proxy_pass http://app:3000;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

> 📸 **Screenshot required:** The `nginx.conf` file open in your editor or shown via `cat nginx.conf`.

---

## Step 4: Create the docker-compose.yml

Create a file named `docker-compose.yml` in the root of the project:

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:8080"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: express-app
    expose:
      - "3000"
    restart: unless-stopped
```

**Key points:**
- `nginx` uses the official `nginx:alpine` image (just runs it, no build).
- `app` is built from our `Dockerfile`.
- `app` starts before `nginx` is considered ready (`depends_on`).
- Only port `8080` is exposed to the host — through Nginx.

> 📸 **Screenshot required:** The `docker-compose.yml` file open in your editor or shown via `cat docker-compose.yml`.

---

## Step 5: Build and Push the Docker Image to DockerHub

### 5.1 Log in to DockerHub

```bash
docker login
```

Enter your DockerHub username and password when prompted.

> 📸 **Screenshot required:** Terminal showing successful `docker login` — `Login Succeeded`.

### 5.2 Build the image with your DockerHub username

Replace `<your-dockerhub-username>` with your actual DockerHub username:

```bash
docker build -t <your-dockerhub-username>/module10-express-app:latest .
```

Example:
```bash
docker build -t asraful/module10-express-app:latest .
```

> 📸 **Screenshot required:** Terminal showing the build output with all steps completing successfully.

### 5.3 Push the image to DockerHub

```bash
docker push <your-dockerhub-username>/module10-express-app:latest
```

> 📸 **Screenshot required:** Terminal showing the push output — layers being pushed and the final digest line.

### 5.4 Verify on DockerHub

Go to `https://hub.docker.com/r/<your-dockerhub-username>/module10-express-app` and confirm the image is listed.

> 📸 **Screenshot required:** DockerHub web page showing your pushed image with its tag and push timestamp.

---

## Step 6: Run Everything with Docker Compose

```bash
docker compose up --build
```

You should see both containers starting — Nginx and the Express app.

> 📸 **Screenshot required:** Terminal showing both services starting up, with logs like `nginx  | ...` and `express-app | Server is running...` (or similar).

To run in detached (background) mode:

```bash
docker compose up --build -d
```

Check running containers:

```bash
docker compose ps
```

> 📸 **Screenshot required:** Output of `docker compose ps` showing both `nginx` and `express-app` containers with status `Up`.

---

## Step 7: Verify in the Browser

Open your browser and go to:

- **Home route:** `http://localhost:8080/` → Should display a Hello World page
- **API route:** `http://localhost:8080/api` → Should return a JSON response

> 📸 **Screenshot required (x2):**
> 1. Browser showing `http://localhost:8080/` with the Hello World page.
> 2. Browser showing `http://localhost:8080/api` with the JSON response.

---

## Step 8: Verify with curl (optional but recommended)

```bash
curl http://localhost:8080/
curl http://localhost:8080/api
```

> 📸 **Screenshot required:** Terminal showing the curl output for both routes.

---

## Step 9: Commit and Push Files to GitHub

```bash
git add Dockerfile docker-compose.yml nginx.conf README.md
git commit -m "Add Dockerfile, docker-compose.yml, and nginx config for Module 10"
git push origin main
```

> 📸 **Screenshot required:** Terminal showing the `git push` completing successfully, and the GitHub repository page showing the new files.

---

## Step 10: Stop the Containers

```bash
docker compose down
```

> 📸 **Screenshot required:** Terminal showing the containers stopping.

---

## Submission Checklist

| Item | Details |
|------|---------|
| ✅ Dockerfile | Added to repository root |
| ✅ docker-compose.yml | Added to repository root |
| ✅ nginx.conf | Added to repository root |
| ✅ DockerHub image pushed | `docker.io/<your-username>/module10-express-app:latest` |
| ✅ App accessible on port 8080 | Via Nginx reverse proxy |
| ✅ Express app starts after Nginx | Controlled by `depends_on` |
| ✅ Screenshots compiled into PDF | All steps documented |
| ✅ GitHub repo link shared | Public repository |

---

## DockerHub Image Link

```
https://hub.docker.com/r/<your-dockerhub-username>/module10-express-app
```

> Replace `<your-dockerhub-username>` with your actual DockerHub username before submitting.

---

## Architecture Summary

```
Browser (port 8080)
       │
       ▼
  ┌─────────┐
  │  Nginx  │  ← nginx:alpine (official image, no build)
  │  :8080  │
  └────┬────┘
       │ proxy_pass http://app:3000
       ▼
  ┌─────────┐
  │  Express│  ← Built from Dockerfile
  │   :3000 │
  └─────────┘
```

- Nginx acts as a reverse proxy, receiving traffic on port 8080 and forwarding it to the Express app on port 3000.
- The Express app is **not** directly exposed to the host — all traffic goes through Nginx.
- Docker Compose manages both services and their startup order.