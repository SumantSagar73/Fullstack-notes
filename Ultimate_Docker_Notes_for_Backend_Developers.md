# Ultimate Docker Notes for Backend Developers

> **Who is this for?** Backend developers who want to understand Docker deeply — not become Docker specialists, but use Docker confidently in day-to-day work, deployments, and interviews.

---

## Table of Contents

1. [What Problem Does Docker Solve?](#1-what-problem-does-docker-solve)
2. [Core Concepts — The Mental Model](#2-core-concepts--the-mental-model)
3. [Installing Docker](#3-installing-docker)
4. [Your First Container — Hands-On](#4-your-first-container--hands-on)
5. [Images — The Blueprint](#5-images--the-blueprint)
6. [Dockerfile — Writing Your Own Blueprint](#6-dockerfile--writing-your-own-blueprint)
7. [Containers — The Running Application](#7-containers--the-running-application)
8. [Docker Networking — How Containers Talk](#8-docker-networking--how-containers-talk)
9. [Docker Volumes — Keeping Your Data Safe](#9-docker-volumes--keeping-your-data-safe)
10. [Docker Compose — Running Multi-Container Apps](#10-docker-compose--running-multi-container-apps)
11. [Environment Variables & Secrets](#11-environment-variables--secrets)
12. [Dockerizing Real Backend Projects](#12-dockerizing-real-backend-projects)
13. [Docker in Production — What You Need to Know](#13-docker-in-production--what-you-need-to-know)
14. [Multi-Stage Builds — Smaller, Safer Images](#14-multi-stage-builds--smaller-safer-images)
15. [Docker Best Practices for Backend Developers](#15-docker-best-practices-for-backend-developers)
16. [Common Docker Commands Cheat Sheet](#16-common-docker-commands-cheat-sheet)
17. [Debugging & Troubleshooting](#17-debugging--troubleshooting)
18. [Docker Interview Questions (70+ Questions)](#18-docker-interview-questions-70-questions)

---

## 1. What Problem Does Docker Solve?

### The "It Works on My Machine" Problem

Imagine you're a chef. You've perfected a recipe at home — your oven, your ingredients, your kitchen. Now your friend tries the same recipe in their kitchen with a different oven temperature, different brand of flour, and slightly different water. The dish tastes completely different.

**That's what happens in software without Docker.** Your Node.js app works perfectly on your laptop with Node 18, a specific version of MongoDB, and certain environment variables. But when your teammate pulls the code, they have Node 20, a different MongoDB, and it crashes.

### Docker's Solution: Ship the Entire Kitchen

Docker lets you pack your application along with everything it needs — the exact operating system, the exact runtime, the exact libraries, the exact configuration — into a single portable box called a **container**. Anyone, anywhere, can run that box and get the exact same result.

### Real-Life Analogy: Shipping Containers

Before the 1950s, shipping goods internationally was chaotic. Every port had different loading methods, different trucks, different storage. Then someone invented the **standardised shipping container** — a metal box of fixed dimensions. It didn't matter what was inside (cars, bananas, electronics). Every ship, crane, truck, and warehouse could handle it the same way.

Docker containers are the same idea for software. It doesn't matter if your app is written in Python, Java, or Go. Docker wraps it in a standard format that any server can run.

---

## 2. Core Concepts — The Mental Model

### Image vs Container

| Concept | Real-Life Analogy | What It Is |
|---------|-------------------|------------|
| **Image** | A recipe card | A read-only template containing your app code, runtime, libraries, and OS. It's the blueprint. |
| **Container** | The dish you cooked from the recipe | A running instance of an image. You can run many containers from the same image. |
| **Dockerfile** | The instructions you write on the recipe card | A text file with step-by-step instructions to build an image. |
| **Registry** | A cookbook library (like Docker Hub) | A place where images are stored and shared. Docker Hub is the most popular public one. |
| **Volume** | A USB drive you plug into your kitchen appliance | Persistent storage that survives container restarts. |
| **Network** | The phone lines between rooms | How containers communicate with each other. |

### How They Connect

```
Dockerfile  →  (docker build)  →  Image  →  (docker run)  →  Container
                                    ↕
                              Docker Registry
                              (Docker Hub, ECR, GCR)
```

### Containers vs Virtual Machines

Think of a **VM** as renting an entire apartment — separate kitchen, bathroom, bedroom. You get full isolation but it's heavy and slow to set up.

Think of a **Container** as renting a room in a co-living space — you share the building's infrastructure (plumbing, electricity) but your room is private. It's lightweight and you can move in within seconds.

| Feature | Virtual Machine | Container |
|---------|----------------|-----------|
| Boot time | Minutes | Seconds |
| Size | Gigabytes | Megabytes |
| OS | Full guest OS per VM | Shares host OS kernel |
| Isolation | Complete hardware-level | Process-level |
| Performance | Near-native but heavier | Near-native and lighter |
| Use case | Running different OS types | Running microservices |

---

## 3. Installing Docker

### On macOS
1. Download Docker Desktop from docker.com.
2. Drag to Applications and open.
3. Verify: `docker --version`

### On Ubuntu/Linux
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify
docker --version

# (Optional) Run Docker without sudo
sudo usermod -aG docker $USER
# Log out and back in for this to take effect
```

### On Windows
1. Install WSL 2 (Windows Subsystem for Linux).
2. Download Docker Desktop.
3. Enable WSL 2 integration in Docker Desktop settings.
4. Verify: `docker --version`

### Verify Everything Works
```bash
docker run hello-world
```

If you see "Hello from Docker!" — you're all set.

---

## 4. Your First Container — Hands-On

### Running Your First Container

```bash
# Run an Nginx web server
docker run -d -p 8080:80 --name my-website nginx
```

Now open `http://localhost:8080` in your browser. You'll see the Nginx welcome page.

**What just happened?**

| Flag | What It Does | Analogy |
|------|-------------|---------|
| `docker run` | Creates and starts a container | "Start cooking" |
| `-d` | Run in the background (detached) | "Put it in the oven and walk away" |
| `-p 8080:80` | Map port 8080 on your machine to port 80 inside the container | "Connect doorbell on the front door to the intercom inside" |
| `--name my-website` | Give it a friendly name | "Label the container box" |
| `nginx` | The image to use | "The recipe to follow" |

### Seeing What's Running

```bash
# List running containers
docker ps

# List ALL containers (including stopped)
docker ps -a
```

### Interacting with a Container

```bash
# See the logs (what the container is printing)
docker logs my-website

# Follow logs in real-time (like tail -f)
docker logs -f my-website

# Jump INSIDE the container (like SSH-ing into a server)
docker exec -it my-website bash

# Once inside, you can explore
ls /usr/share/nginx/html/
cat /etc/nginx/nginx.conf
exit
```

**Analogy:** `docker exec -it` is like walking into a room and looking around. You're inside the container's filesystem now. When you `exit`, you're back on your host machine.

### Stopping and Removing

```bash
# Stop the container (it still exists, just not running)
docker stop my-website

# Start it again
docker start my-website

# Remove it completely (must be stopped first, or use -f)
docker rm my-website

# Stop AND remove in one go
docker rm -f my-website
```

---

## 5. Images — The Blueprint

### What's Inside an Image?

An image is made up of **layers**. Each instruction in your Dockerfile creates a new layer. Layers are cached, so rebuilding is fast if only later layers change.

**Analogy:** Think of a layered cake. The bottom layer is the OS, the next layer adds the runtime, the next adds your dependencies, and the top layer is your application code. If you change only the frosting (your code), you don't have to rebake the whole cake.

```
┌─────────────────────────┐
│   Your app code         │  ← Changes often (top layer)
├─────────────────────────┤
│   npm install output    │  ← Changes when deps change
├─────────────────────────┤
│   Node.js runtime       │  ← Rarely changes
├─────────────────────────┤
│   Alpine Linux OS       │  ← Almost never changes (base layer)
└─────────────────────────┘
```

### Pulling Images

```bash
# Pull an image from Docker Hub
docker pull node:20-alpine

# List all images on your machine
docker images

# Remove an image
docker rmi node:20-alpine
```

### Image Tags

Tags are version labels. Think of them like git tags or npm versions.

```bash
# Specific version (recommended for production)
docker pull node:20.11-alpine

# Major version (gets latest minor/patch)
docker pull node:20-alpine

# "latest" tag (dangerous — could be anything)
docker pull node:latest

# Using a specific OS variant
docker pull node:20-bullseye     # Debian Bullseye
docker pull node:20-alpine       # Alpine Linux (much smaller)
```

**Rule of thumb for backend devs:** Always use a specific tag in production. Never use `latest` — it can change under you without warning. It's like importing `*` — you don't know what you're getting.

### Image Size Matters

```bash
# Compare image sizes
docker images | grep node
```

| Image | Size |
|-------|------|
| `node:20` | ~1 GB |
| `node:20-slim` | ~200 MB |
| `node:20-alpine` | ~130 MB |

**Why care?** Smaller images mean faster builds, faster deployments, faster scaling, and smaller attack surface. For backend services, Alpine-based images are usually the best choice.

---

## 6. Dockerfile — Writing Your Own Blueprint

### Anatomy of a Dockerfile

A Dockerfile is like writing cooking instructions — each line is one step.

```dockerfile
# Step 1: Start with a base image (choose your kitchen)
FROM node:20-alpine

# Step 2: Set the working directory inside the container
WORKDIR /app

# Step 3: Copy dependency files first (for caching)
COPY package.json package-lock.json ./

# Step 4: Install dependencies
RUN npm ci --only=production

# Step 5: Copy the rest of your application code
COPY . .

# Step 6: Tell Docker which port your app uses
EXPOSE 3000

# Step 7: Define the command to start your app
CMD ["node", "server.js"]
```

### Every Instruction Explained

#### FROM — Choose Your Starting Point
```dockerfile
# Start from an official Node.js image on Alpine Linux
FROM node:20-alpine
```
This is like choosing which kitchen to cook in. You pick a base that has what you need already installed.

#### WORKDIR — Set Your Workspace
```dockerfile
# All future commands run from this directory
WORKDIR /app
```
Think of this as walking into a specific room. All your files will be placed here. If the directory doesn't exist, Docker creates it.

#### COPY — Bring Files In
```dockerfile
# Copy from host → container
COPY package.json .          # single file
COPY src/ ./src/             # entire directory
COPY . .                     # everything (respect .dockerignore!)
```
Think of it as moving boxes from your car into your new apartment.

#### RUN — Execute a Command During Build
```dockerfile
# Install dependencies
RUN npm ci --only=production

# You can chain commands
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```
RUN executes while the image is being built. The result becomes a layer. Think of it as installing furniture before you move in — it's done once and baked into the apartment.

#### EXPOSE — Document Which Port Your App Uses
```dockerfile
EXPOSE 3000
```
This doesn't actually open the port — it's documentation. It tells other developers "this container expects traffic on port 3000." You still need `-p` when running.

**Analogy:** Putting a sign on the door saying "reception is on floor 3." The sign doesn't build the elevator — you still need `-p` to create the actual connection.

#### CMD — Default Startup Command
```dockerfile
# Use exec form (recommended — proper signal handling)
CMD ["node", "server.js"]

# Shell form (runs inside /bin/sh -c)
CMD node server.js
```
This is the instruction that runs when the container starts. There can be only one CMD per Dockerfile. Think of it as the "Open for business" sign.

#### ENTRYPOINT — The Fixed Part of the Command
```dockerfile
ENTRYPOINT ["node"]
CMD ["server.js"]

# docker run myapp              → runs: node server.js
# docker run myapp worker.js    → runs: node worker.js
```
ENTRYPOINT is the fixed part, CMD is the default argument that can be overridden. Think of ENTRYPOINT as "we're definitely using a car" and CMD as "by default, we drive to the office."

#### ENV — Set Environment Variables
```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```
Like setting the thermostat before you move in — it affects everything inside.

#### ARG — Build-Time Variables
```dockerfile
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}-alpine

ARG BUILD_DATE
RUN echo "Built on ${BUILD_DATE}"
```
ARG only exists during build. ENV persists into the running container. ARG is like the construction blueprint notes — workers see them during building, but tenants don't see them after moving in.

#### USER — Don't Run as Root
```dockerfile
# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to that user
USER appuser
```
Running as root inside a container is like leaving your house keys under the doormat. If someone breaks in, they own everything. Always run your app as a non-root user.

### The .dockerignore File

Just like `.gitignore`, this tells Docker which files to skip when copying.

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
Dockerfile
docker-compose.yml
README.md
.vscode
coverage
tests
```

**Why is this important?** Without it, `COPY . .` would copy your entire `node_modules` folder (which you're about to reinstall anyway), your `.git` history, your secret `.env` files, and more. Your image becomes bloated and potentially insecure.

### Building Your Image

```bash
# Basic build
docker build -t my-api:1.0 .

# Breaking that down:
# -t my-api:1.0   → Tag the image as "my-api" version "1.0"
# .                → Use the current directory as build context (where to find the Dockerfile)

# Build with a specific Dockerfile
docker build -t my-api:1.0 -f Dockerfile.production .

# Build with build arguments
docker build --build-arg NODE_VERSION=20 -t my-api:1.0 .

# See build history / layers
docker history my-api:1.0
```

---

## 7. Containers — The Running Application

### Container Lifecycle

```
Created  →  Running  →  Paused  →  Stopped  →  Removed
  ↑            ↓                      ↑
  └── docker create    docker stop ───┘
       docker run       docker kill
```

**Analogy:** A container is like a restaurant.
- **Created:** Kitchen is set up but doors aren't open.
- **Running:** Open for business, serving customers.
- **Paused:** Lunch break — everything frozen in place.
- **Stopped:** Closed for the day — lights off, but everything is still inside.
- **Removed:** Demolished — everything is gone.

### Running Containers — All the Options

```bash
# Basic run
docker run my-api:1.0

# Detached (background)
docker run -d my-api:1.0

# With port mapping
docker run -d -p 3000:3000 my-api:1.0

# With environment variables
docker run -d -p 3000:3000 -e DATABASE_URL=mongodb://db:27017/myapp my-api:1.0

# With a volume (persist data)
docker run -d -p 3000:3000 -v mydata:/app/data my-api:1.0

# With auto-restart
docker run -d --restart unless-stopped my-api:1.0

# With memory and CPU limits
docker run -d --memory 512m --cpus 1.5 my-api:1.0

# Interactive mode (for debugging)
docker run -it my-api:1.0 sh
```

### Resource Limits — Don't Let Containers Eat Everything

```bash
# Limit memory to 512MB
docker run -d --memory 512m my-api:1.0

# Limit to 1.5 CPU cores
docker run -d --cpus 1.5 my-api:1.0

# Both
docker run -d --memory 512m --cpus 1.5 my-api:1.0
```

**Why does this matter for backend devs?** Without limits, a memory leak in one container can crash the entire host machine. In production (Kubernetes, ECS), you always set resource limits. It's like giving each tenant in an apartment building a water meter — one person's leak doesn't flood the whole building.

### Restart Policies

```bash
# Never restart (default)
docker run -d --restart no my-api:1.0

# Always restart (even if you manually stop it)
docker run -d --restart always my-api:1.0

# Restart unless you explicitly stopped it
docker run -d --restart unless-stopped my-api:1.0

# Restart on failure (with max retry count)
docker run -d --restart on-failure:5 my-api:1.0
```

**For backend services:** `unless-stopped` is usually what you want. Your API server comes back up after a crash or host reboot, but you can still manually stop it for maintenance.

---

## 8. Docker Networking — How Containers Talk

### The Problem

You have a Node.js API container and a MongoDB container. They need to talk to each other. But `localhost` inside a container refers to that container itself — not your host machine, not other containers.

**Analogy:** You and your neighbour live in separate apartments. You can't just yell through the wall. You need either a shared hallway, a phone line, or a doorbell system.

### Network Types

| Type | Analogy | Use Case |
|------|---------|----------|
| **bridge** (default) | Private hallway between apartments | Containers on the same host talk to each other |
| **host** | No walls — you ARE the building | Container uses host's network directly (Linux only) |
| **none** | Soundproof room | Total network isolation |
| **overlay** | Phone system across buildings | Containers on different hosts (Docker Swarm) |

### Practical: Connecting API + Database

```bash
# Step 1: Create a custom network
docker network create my-app-network

# Step 2: Run MongoDB on that network
docker run -d \
  --name mongo \
  --network my-app-network \
  -v mongo-data:/data/db \
  mongo:7

# Step 3: Run your API on the same network
docker run -d \
  --name api \
  --network my-app-network \
  -p 3000:3000 \
  -e DATABASE_URL=mongodb://mongo:27017/myapp \
  my-api:1.0
```

**The magic:** On a custom bridge network, containers can reach each other by name. Your API connects to `mongodb://mongo:27017/myapp` — Docker resolves `mongo` to the MongoDB container's IP address automatically.

**Why not use the default bridge network?** The default bridge network doesn't support DNS name resolution. You'd have to use IP addresses, which change every time you recreate containers.

### Useful Network Commands

```bash
# List all networks
docker network ls

# Inspect a network (see which containers are connected)
docker network inspect my-app-network

# Connect a running container to a network
docker network connect my-app-network my-container

# Disconnect
docker network disconnect my-app-network my-container

# Remove a network
docker network rm my-app-network
```

---

## 9. Docker Volumes — Keeping Your Data Safe

### The Problem

Containers are **ephemeral** — when you remove a container, everything inside it is gone. Your database data, uploaded files, logs — all wiped.

**Analogy:** Imagine writing a book on a whiteboard. When someone erases the whiteboard (removes the container), your book is gone. A volume is like keeping your book on a USB drive that you plug into the whiteboard. Even if the whiteboard is erased, your book is safe on the USB drive.

### Three Ways to Persist Data

#### 1. Named Volumes (Recommended)
```bash
# Docker manages the storage location
docker run -d -v my-db-data:/data/db mongo:7
```
Docker stores the data somewhere on the host (you don't need to know where). It's like cloud storage — you don't manage the physical disk, just the data.

#### 2. Bind Mounts (For Development)
```bash
# You control the exact path on your host
docker run -d -v $(pwd)/src:/app/src my-api:1.0
```
Your host directory is mounted directly into the container. Changes on either side are reflected immediately. Perfect for development — edit code on your laptop, and the container sees the changes instantly.

#### 3. tmpfs Mounts (Temporary, In-Memory)
```bash
# Data stored in memory, never written to disk
docker run -d --tmpfs /app/tmp my-api:1.0
```
Fast but temporary — data disappears when the container stops. Good for sensitive data that shouldn't be written to disk.

### Practical: Development Setup with Live Reloading

```bash
# Mount your source code into the container for live reloading
docker run -d \
  --name dev-api \
  -p 3000:3000 \
  -v $(pwd):/app \
  -v /app/node_modules \
  my-api:dev
```

**What's that `-v /app/node_modules` line?** It creates an anonymous volume for `node_modules`, preventing your host's `node_modules` (which might be for a different OS) from overriding the container's `node_modules`. Think of it as saying "share everything except this one folder."

### Volume Commands

```bash
# List all volumes
docker volume ls

# Create a volume explicitly
docker volume create my-data

# Inspect a volume (see where it's stored)
docker volume inspect my-data

# Remove a volume
docker volume rm my-data

# Remove ALL unused volumes (careful!)
docker volume prune
```

---

## 10. Docker Compose — Running Multi-Container Apps

### The Problem

A typical backend application has multiple services: API server, database, cache, message queue, maybe a worker process. Running each one with a long `docker run` command is tedious and error-prone.

**Analogy:** Imagine hiring a band. You could call each musician individually, tell them the time, the venue, the songs, the volume. Or you could write one sheet with all the details and hand it to everyone. Docker Compose is that sheet.

### docker-compose.yml — The Master Plan

```yaml
# docker-compose.yml
version: "3.8"

services:
  # Your API server
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=mongodb://mongo:27017/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - mongo
      - redis
    restart: unless-stopped

  # MongoDB database
  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=secret

  # Redis cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

# Named volumes (declared at the bottom)
volumes:
  mongo-data:
  redis-data:
```

### Compose Commands

```bash
# Start all services (in foreground)
docker compose up

# Start in background
docker compose up -d

# Start and rebuild images
docker compose up -d --build

# Stop all services
docker compose down

# Stop and remove volumes too (wipes data!)
docker compose down -v

# See logs
docker compose logs

# Follow logs for a specific service
docker compose logs -f api

# See running services
docker compose ps

# Run a one-off command in a service
docker compose exec api sh

# Restart a single service
docker compose restart api

# Scale a service (run multiple instances)
docker compose up -d --scale api=3
```

### depends_on — Order Matters, But Not How You Think

```yaml
services:
  api:
    depends_on:
      - mongo
```

This only ensures MongoDB **starts** before your API. It does NOT wait for MongoDB to be **ready**. Your app might crash because it tries to connect to MongoDB before MongoDB has finished initializing.

**The Fix: Healthchecks**

```yaml
services:
  mongo:
    image: mongo:7
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    build: .
    depends_on:
      mongo:
        condition: service_healthy
```

Now Docker waits until MongoDB's healthcheck passes before starting the API. It's like waiting until the restaurant kitchen confirms they're ready before seating guests.

### Compose Profiles — Optional Services

```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"

  mongo:
    image: mongo:7

  # Only start these when explicitly requested
  mongo-express:
    image: mongo-express
    profiles:
      - debug
    ports:
      - "8081:8081"

  mailhog:
    image: mailhog/mailhog
    profiles:
      - debug
    ports:
      - "8025:8025"
```

```bash
# Normal start (api + mongo only)
docker compose up -d

# Include debug tools
docker compose --profile debug up -d
```

---

## 11. Environment Variables & Secrets

### Passing Environment Variables

```bash
# Method 1: Inline
docker run -e DATABASE_URL=mongodb://localhost:27017/myapp my-api

# Method 2: From a file
docker run --env-file .env my-api

# Method 3: In docker-compose.yml
services:
  api:
    environment:
      - DATABASE_URL=mongodb://mongo:27017/myapp
      - NODE_ENV=production

# Method 4: env_file in docker-compose.yml
services:
  api:
    env_file:
      - .env
      - .env.production
```

### The .env File

```bash
# .env (NEVER commit this to git)
DATABASE_URL=mongodb://mongo:27017/myapp
JWT_SECRET=super-secret-key-change-in-production
REDIS_URL=redis://redis:6379
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI...
```

**Critical rule:** Add `.env` to your `.gitignore` AND your `.dockerignore`. Secrets in images are secrets leaked. Anyone who can pull your image can extract your credentials.

### Docker Secrets (for Swarm/Production)

For production, environment variables for sensitive data aren't ideal — they show up in `docker inspect`, process listings, and logs. Docker Secrets mounts sensitive data as files.

```yaml
# docker-compose.yml (Swarm mode)
services:
  api:
    image: my-api:1.0
    secrets:
      - db_password
      - jwt_secret

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

Your app reads the secret from `/run/secrets/db_password` inside the container.

---

## 12. Dockerizing Real Backend Projects

### Example 1: Node.js Express API

```
my-express-api/
├── src/
│   ├── server.js
│   ├── routes/
│   └── models/
├── package.json
├── package-lock.json
├── Dockerfile
├── .dockerignore
└── docker-compose.yml
```

**Dockerfile:**
```dockerfile
FROM node:20-alpine

# Security: don't run as root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy dependency files first (caching layer)
COPY package.json package-lock.json ./

# Install production dependencies only
RUN npm ci --only=production

# Copy source code
COPY src/ ./src/

# Switch to non-root user
USER appuser

EXPOSE 3000

# Use node directly (not npm start) for proper signal handling
CMD ["node", "src/server.js"]
```

**docker-compose.yml:**
```yaml
version: "3.8"

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - MONGO_URI=mongodb://mongo:27017/express-api
    volumes:
      - ./src:/app/src
    depends_on:
      mongo:
        condition: service_healthy

  mongo:
    image: mongo:7
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongo-data:
```

### Example 2: Python FastAPI

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Install dependencies first (caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN adduser --disabled-password --gecos "" appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Important:** `--host 0.0.0.0` is critical. Without it, uvicorn binds to `127.0.0.1` (localhost), which means only the container itself can access it. With `0.0.0.0`, it listens on all interfaces, allowing traffic from outside the container.

### Example 3: Java Spring Boot

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### Example 4: Go API

```dockerfile
# Build stage
FROM golang:1.22-alpine AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

# Runtime stage — scratch means "empty image, nothing at all"
FROM scratch
COPY --from=build /app/server /server
EXPOSE 8080
CMD ["/server"]
```

Go's compiled binary needs no runtime, so you can use `scratch` (an empty image). Your final image is just a few megabytes. It's like shipping just the cake, not the entire bakery.

---

## 13. Docker in Production — What You Need to Know

### You Probably Won't Use Docker Alone in Production

As a backend developer, you should know that in production environments, Docker containers are typically managed by an **orchestrator**:

| Tool | What It Is | Analogy |
|------|-----------|---------|
| **Kubernetes (K8s)** | Container orchestration platform | An entire airport management system — scheduling, routing, scaling |
| **AWS ECS/Fargate** | AWS-managed container service | Hiring a logistics company to manage your shipping containers |
| **Docker Swarm** | Docker's built-in orchestrator | A simpler, smaller airport |

You don't need to master Kubernetes to be a good backend developer, but you should understand why it exists and the basics of how your containers get deployed.

### Health Checks — Is Your App Actually Working?

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

In your app, create a `/health` endpoint:
```javascript
// Express.js example
app.get('/health', (req, res) => {
  // Check database connection, etc.
  res.status(200).json({ status: 'healthy', uptime: process.uptime() });
});
```

**Why this matters:** Orchestrators use health checks to decide if your container is alive. If the health check fails, the orchestrator kills the container and starts a new one. It's like a lifeguard checking if swimmers are okay — no response means rescue is needed.

### Logging — stdout and stderr

**Rule:** Never write logs to files inside a container. Write to stdout and stderr.

```javascript
// Good — writes to stdout
console.log('Server started on port 3000');

// Bad — writes to a file inside the container
fs.appendFileSync('/var/log/app.log', 'Server started\n');
```

Docker captures stdout/stderr automatically. Orchestrators and log aggregation tools (ELK Stack, Datadog, CloudWatch) can collect them from there. Writing to files means the logs die with the container and are harder to aggregate.

### Graceful Shutdown

When Docker stops a container, it sends `SIGTERM`, waits 10 seconds, then sends `SIGKILL`. Your app should handle `SIGTERM` gracefully:

```javascript
// Graceful shutdown in Node.js
const server = app.listen(3000);

process.on('SIGTERM', () => {
  console.log('SIGTERM received. Shutting down gracefully...');
  server.close(() => {
    // Close database connections
    mongoose.connection.close(false, () => {
      console.log('MongoDB connection closed.');
      process.exit(0);
    });
  });
});
```

**Why this matters:** Without graceful shutdown, in-flight requests get dropped, database writes get corrupted, and connections leak. It's like a restaurant closing — you don't throw diners out mid-meal. You stop accepting new diners, serve the ones who are eating, and then close.

**Important:** Use `CMD ["node", "server.js"]` (exec form), NOT `CMD node server.js` (shell form). The shell form wraps your app in `/bin/sh`, which doesn't forward signals to your Node process. Your app never receives `SIGTERM` and gets killed after 10 seconds.

---

## 14. Multi-Stage Builds — Smaller, Safer Images

### The Problem

Your build tools (compilers, dev dependencies, test frameworks) are huge and have no business being in your production image.

**Analogy:** When you're building a house, you need cranes, scaffolding, concrete mixers, and construction workers. But once the house is built, you don't leave the crane in the living room. Multi-stage builds let you use the crane during construction and ship only the finished house.

### Node.js Multi-Stage Build

```dockerfile
# ============ Stage 1: Build ============
FROM node:20-alpine AS builder

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build     # Compile TypeScript, bundle, etc.
RUN npm prune --production   # Remove dev dependencies

# ============ Stage 2: Production ============
FROM node:20-alpine

WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy ONLY what we need from the builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

USER appuser
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### What Changes?

| Without Multi-Stage | With Multi-Stage |
|---------------------|------------------|
| TypeScript compiler in production | Only compiled JavaScript |
| Dev dependencies (nodemon, jest, etc.) | Only production dependencies |
| Source `.ts` files | Only `.js` output |
| ~500 MB image | ~150 MB image |

---

## 15. Docker Best Practices for Backend Developers

### 1. Order Dockerfile Instructions by Change Frequency

Put things that change rarely at the top, things that change often at the bottom. Docker caches each layer — if a layer hasn't changed, Docker reuses it.

```dockerfile
# GOOD: Dependencies change less often than code
FROM node:20-alpine
WORKDIR /app
COPY package.json package-lock.json ./   # Changes rarely
RUN npm ci                                # Cached unless package.json changed
COPY . .                                  # Changes every commit (last = fast rebuild)
```

```dockerfile
# BAD: Every code change invalidates the npm install cache
FROM node:20-alpine
WORKDIR /app
COPY . .                                  # Changes every commit
RUN npm ci                                # Re-runs every time because COPY above changed
```

### 2. Use .dockerignore Religiously

Every file you exclude is a file that doesn't need to be sent to the Docker daemon during build, doesn't bloat your image, and can't leak secrets.

### 3. One Process per Container

Run one service per container. Don't run your API, database, and Redis all in one container.

**Why?** Each service can scale independently, restart independently, and update independently. It's like having one employee per role — the cashier doesn't have to stop when the cook takes a break.

### 4. Pin Your Base Image Versions

```dockerfile
# GOOD: Specific and reproducible
FROM node:20.11.1-alpine3.19

# OKAY: Tracks minor updates
FROM node:20-alpine

# BAD: Could be anything tomorrow
FROM node:latest
```

### 5. Minimize Layers

```dockerfile
# BAD: 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# GOOD: 1 layer
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

### 6. Don't Store Secrets in Images

```dockerfile
# NEVER do this
ENV DATABASE_PASSWORD=mysecretpassword
COPY .env .

# Pass secrets at runtime instead
# docker run -e DATABASE_PASSWORD=mysecretpassword my-api
```

### 7. Use COPY, Not ADD

`ADD` has extra features (auto-extracting tar files, downloading URLs) that you almost never need and can be surprising. `COPY` is explicit and predictable.

### 8. Scan Your Images for Vulnerabilities

```bash
# Docker's built-in scanner
docker scout cves my-api:1.0

# Trivy (popular open-source scanner)
trivy image my-api:1.0
```

---

## 16. Common Docker Commands Cheat Sheet

### Images

```bash
docker build -t name:tag .           # Build an image
docker images                         # List images
docker rmi image_name                 # Remove an image
docker pull image_name:tag            # Pull from registry
docker push image_name:tag            # Push to registry
docker tag old_name new_name          # Rename/retag an image
docker image prune                    # Remove dangling images
docker image prune -a                 # Remove ALL unused images
```

### Containers

```bash
docker run -d -p 3000:3000 image     # Run detached with port mapping
docker ps                             # List running containers
docker ps -a                          # List all containers
docker stop container_name            # Stop gracefully
docker start container_name           # Start stopped container
docker restart container_name         # Restart
docker rm container_name              # Remove (must be stopped)
docker rm -f container_name           # Force remove
docker logs container_name            # View logs
docker logs -f container_name         # Follow logs
docker exec -it container_name sh     # Shell into container
docker inspect container_name         # Detailed info (JSON)
docker stats                          # Live resource usage
docker cp file container:/path        # Copy file into container
docker cp container:/path file        # Copy file from container
```

### Volumes

```bash
docker volume create name             # Create a volume
docker volume ls                      # List volumes
docker volume inspect name            # Volume details
docker volume rm name                 # Remove a volume
docker volume prune                   # Remove unused volumes
```

### Networks

```bash
docker network create name            # Create a network
docker network ls                     # List networks
docker network inspect name           # Network details
docker network connect net container  # Connect container to network
docker network rm name                # Remove a network
```

### Docker Compose

```bash
docker compose up -d                  # Start all services
docker compose up -d --build          # Start and rebuild
docker compose down                   # Stop and remove
docker compose down -v                # Stop, remove, delete volumes
docker compose ps                     # List services
docker compose logs -f service        # Follow service logs
docker compose exec service sh        # Shell into service
docker compose restart service        # Restart one service
```

### Cleanup

```bash
docker system df                      # Show disk usage
docker system prune                   # Remove unused data
docker system prune -a --volumes      # Nuclear option: remove EVERYTHING unused
```

---

## 17. Debugging & Troubleshooting

### Container Won't Start

```bash
# Check the logs — 90% of issues are here
docker logs container_name

# Check the container's exit code
docker inspect container_name --format='{{.State.ExitCode}}'
# Exit code 0 = clean exit
# Exit code 1 = application error
# Exit code 137 = killed (OOM or docker kill)
# Exit code 139 = segfault
# Exit code 143 = SIGTERM (graceful stop)
```

### Can't Connect to Container

```bash
# Is the container running?
docker ps

# Are the ports mapped?
docker port container_name

# Is the app listening on the right interface?
# Inside the container:
docker exec -it container_name sh
netstat -tlnp    # or: ss -tlnp

# Common mistake: app listens on 127.0.0.1 instead of 0.0.0.0
```

### Container Runs Out of Memory

```bash
# Check current memory usage
docker stats container_name

# Check if the container was OOM-killed
docker inspect container_name --format='{{.State.OOMKilled}}'
```

### Debugging Inside a Container

```bash
# If the container has a shell
docker exec -it container_name sh

# If the container doesn't have a shell (like scratch-based images)
# Run a debug container that shares the network namespace
docker run -it --network container:container_name alpine sh
```

### Build Taking Forever

```bash
# Check what's being sent as build context
# If it says "Sending build context to Docker daemon  500MB", your .dockerignore is wrong

# Build with progress output
docker build --progress=plain -t my-api .

# Check layer sizes
docker history my-api:1.0
```

### Container Networking Issues

```bash
# Can container A reach container B?
docker exec -it container_a ping container_b

# Check DNS resolution
docker exec -it container_a nslookup container_b

# Are they on the same network?
docker network inspect my-network
```

---

## 18. Docker Interview Questions (70+ Questions)

### Beginner Level — Concepts

**Q1: What is Docker and why do we use it?**
Docker is a platform that lets you package your application along with all its dependencies into a standardised unit called a container. We use it to ensure consistency across environments (dev, staging, production), simplify deployments, and make applications portable. For a backend developer, it means your API runs the same way on your laptop as it does on a production server in AWS.

**Q2: What is the difference between a Docker image and a container?**
An image is a read-only template — like a class in OOP. A container is a running instance of that image — like an object instantiated from a class. You can create many containers from one image, just like you can create many objects from one class.

**Q3: What is a Dockerfile?**
A text file with instructions to build a Docker image. Each instruction (FROM, COPY, RUN, CMD) creates a layer in the image. It's the recipe that Docker follows to build your application's image.

**Q4: What is Docker Hub?**
A public registry where Docker images are stored and shared. It's like npm for Docker images. You pull base images (node, python, postgres) from Docker Hub and can push your own images there.

**Q5: What is the difference between Docker and a Virtual Machine?**
A VM includes a full guest operating system and runs on a hypervisor — it virtualises hardware. Docker containers share the host OS kernel and virtualise only the application layer. Containers are lighter (MBs vs GBs), start faster (seconds vs minutes), and use fewer resources. However, VMs provide stronger isolation since each VM has its own kernel.

**Q6: Explain Docker's architecture.**
Docker uses a client-server architecture. The Docker Client (CLI) sends commands to the Docker Daemon (dockerd), which does the actual work of building, running, and managing containers. They communicate via a REST API. The daemon manages images, containers, networks, and volumes. A Docker Registry (like Docker Hub) stores images that the daemon pulls from and pushes to.

**Q7: What does `docker run` actually do?**
It combines several steps: pulls the image if not available locally, creates a new container from the image, allocates a filesystem and network interface, and starts the container by executing the CMD/ENTRYPOINT instruction. It's shorthand for `docker create` + `docker start`.

**Q8: What is the difference between COPY and ADD in a Dockerfile?**
Both copy files from host to image. ADD has two extra features: it can auto-extract tar archives and download files from URLs. But these extras can cause unexpected behaviour. Best practice is to always use COPY unless you specifically need tar extraction. If you need to download files, use RUN with curl or wget instead — it's more explicit.

**Q9: What is a Docker volume? Why do we need it?**
A volume is a persistent storage mechanism that lives outside the container's filesystem. Containers are ephemeral — when removed, their data is gone. Volumes let you persist data (database files, uploaded files) across container restarts and removal. They're managed by Docker and are the recommended way to persist data.

**Q10: What does `-p 8080:80` mean?**
It maps port 8080 on the host machine to port 80 inside the container. Traffic hitting localhost:8080 on your machine gets forwarded to port 80 in the container. The format is `host_port:container_port`.

---

### Intermediate Level — Practical

**Q11: What is the difference between CMD and ENTRYPOINT?**
CMD provides the default command that runs when a container starts. It can be overridden entirely from the command line. ENTRYPOINT sets a fixed command that always runs — arguments from the command line are appended to it. Use ENTRYPOINT when the container should always run the same executable, and CMD for default arguments that users might want to change.

**Q12: What is the difference between RUN, CMD, and ENTRYPOINT?**
RUN executes during image build time and creates a new layer (installing packages, compiling code). CMD runs when the container starts — it's the default command. ENTRYPOINT also runs when the container starts but is harder to override — it's the fixed executable. In a Dockerfile, you can have many RUN instructions but only one CMD and one ENTRYPOINT (the last one wins).

**Q13: What is layer caching and how do you optimise it?**
Docker caches each layer (instruction result). If a layer and everything before it hasn't changed, Docker reuses the cache. To optimise: put instructions that change rarely (installing OS packages) at the top and instructions that change often (copying application code) at the bottom. The classic example is copying `package.json` and running `npm install` before copying the rest of the code.

**Q14: What is a multi-stage build? Why would you use it?**
A multi-stage build uses multiple FROM statements in a single Dockerfile. You build your application in one stage (with compilers, dev tools) and copy only the built output to a final lightweight stage. Benefits: much smaller production images, no build tools or source code in production, better security. Essential for compiled languages (Go, Java, TypeScript).

**Q15: How do containers communicate with each other?**
Containers on the same Docker network can communicate using container names as hostnames (DNS resolution). You create a custom bridge network with `docker network create`, put both containers on it, and one container connects to the other using its name (e.g., `mongodb://mongo:27017`). The default bridge network doesn't support automatic DNS resolution.

**Q16: What is Docker Compose and when would you use it?**
Docker Compose is a tool for defining and running multi-container applications using a YAML file. You use it when your backend application needs multiple services (API + database + cache + queue). Instead of running multiple `docker run` commands with complex flags, you define everything declaratively and run `docker compose up`.

**Q17: What happens when you run `docker compose up`?**
Docker Compose reads docker-compose.yml, creates a default network for the project, builds images if build context is specified, creates and starts containers for each service in dependency order, and attaches the containers to the project network. All services can reach each other by their service name.

**Q18: Explain the difference between bind mounts and volumes.**
Volumes are managed by Docker, stored in Docker's storage area, and work on all platforms. They're the preferred mechanism for persistent data. Bind mounts map a specific host path directly into the container — they depend on the host's directory structure and work differently across OS. Use volumes for persistent data (databases) and bind mounts for development (live-reloading source code).

**Q19: What is the .dockerignore file?**
Like .gitignore for Docker builds. It lists files and directories that should NOT be sent to the Docker daemon during build. This reduces build context size (faster builds), prevents sensitive files (.env, secrets) from being included in images, and avoids copying unnecessary files (node_modules, .git, tests).

**Q20: How do you pass configuration to a Docker container?**
Several ways: environment variables via `-e` flag or env_file, Docker configs, Docker secrets (for sensitive data in Swarm mode), config files mounted via volumes, and command-line arguments. For backend apps, environment variables are the most common method, following the 12-factor app methodology.

**Q21: What is a dangling image?**
An image that is no longer tagged and not referenced by any container. This happens when you rebuild an image with the same tag — the old image becomes dangling. Clean them up with `docker image prune`. They waste disk space.

**Q22: How do you view logs from a Docker container?**
`docker logs container_name` for all logs, `docker logs -f container_name` to follow in real-time, `docker logs --tail 100 container_name` for the last 100 lines, and `docker logs --since 2h container_name` for logs from the last 2 hours. In production, you typically use a logging driver to send logs to a centralised system.

**Q23: What is the purpose of `EXPOSE` in a Dockerfile?**
EXPOSE is documentation — it tells users which port the container is expected to listen on. It does NOT actually publish the port or make it accessible from outside. You still need to use `-p` when running the container. Think of it as a comment with semantic meaning.

**Q24: What does `docker exec -it container sh` do?**
It runs a new process inside a running container. `-i` keeps stdin open (interactive), `-t` allocates a pseudo-TTY (so you get a terminal prompt), and `sh` is the command to execute. It's used for debugging — looking at files, checking processes, testing connectivity inside the container.

**Q25: What restart policy would you use for a production backend service?**
`unless-stopped`. It restarts the container automatically if it crashes or if the Docker daemon restarts (e.g., after a server reboot), but doesn't restart it if you explicitly stopped it with `docker stop`. `always` would restart even after manual stops, which is annoying during maintenance.

---

### Advanced Level — Production & Architecture

**Q26: What is the difference between `docker stop` and `docker kill`?**
`docker stop` sends SIGTERM, waits for a grace period (default 10 seconds), then sends SIGKILL if the container hasn't stopped. `docker kill` sends SIGKILL immediately. Always prefer `docker stop` in production because it allows your application to shut down gracefully — close database connections, finish in-flight requests, flush buffers.

**Q27: Why should you not run containers as root?**
If an attacker exploits a vulnerability in your application, they'll have root access inside the container. While container isolation prevents direct host access, there are known container escape vulnerabilities. Running as a non-root user follows the principle of least privilege and adds a defense layer. Always use the USER instruction in your Dockerfile.

**Q28: How do you handle graceful shutdown in a Docker container?**
Your application should listen for the SIGTERM signal, stop accepting new requests, finish processing in-flight requests, close database connections and file handles, then exit with code 0. Use the exec form of CMD (`CMD ["node", "server.js"]`) so your process receives signals directly. The shell form wraps your process in `/bin/sh`, which doesn't forward signals.

**Q29: What's the difference between exec form and shell form for CMD?**
Exec form (`CMD ["node", "server.js"]`) runs the process directly as PID 1 — it receives Unix signals properly and is recommended. Shell form (`CMD node server.js`) wraps the command in `/bin/sh -c`, meaning the shell is PID 1 and your process is a child. The shell doesn't forward signals like SIGTERM to your app, so graceful shutdown doesn't work.

**Q30: How does Docker networking work internally?**
Docker creates a virtual bridge network (docker0) on the host. Each container gets a virtual ethernet interface (veth) connected to this bridge. Docker manages iptables rules for port forwarding. Custom bridge networks also provide DNS resolution — an embedded DNS server resolves container names to IP addresses. Host networking skips all this and uses the host's network stack directly.

**Q31: Explain the Docker build cache. How can it break?**
Docker caches each build layer. For each instruction, Docker checks if that instruction and all preceding layers are identical to a cached version. If yes, it reuses the cache. The cache breaks (invalidates) when: the instruction text changes, files referenced by COPY/ADD change (checked by checksum), or any preceding layer was rebuilt. Once one layer's cache is invalidated, all subsequent layers are also rebuilt.

**Q32: What are Docker logging drivers?**
Logging drivers control where container logs go. The default is `json-file` (stored on the host). Others include `syslog`, `journald`, `awslogs` (CloudWatch), `gcplogs`, `fluentd`, and `splunk`. In production, you configure a logging driver to send logs to a centralised logging system rather than storing them on the host.

**Q33: How do you reduce Docker image size?**
Use minimal base images (Alpine), multi-stage builds, combine RUN instructions to reduce layers, clean up package manager caches in the same RUN instruction, use `.dockerignore` to exclude unnecessary files, install only production dependencies, avoid installing unnecessary packages, and use `--no-cache-dir` for pip installs. A smaller image means faster pulls, faster deploys, and a smaller attack surface.

**Q34: What happens when a Docker container runs out of memory?**
The Linux OOM (Out of Memory) killer terminates the container's main process. The container exits with code 137 (128 + 9, since SIGKILL is signal 9). Docker reports `OOMKilled: true` in the container inspection. To prevent this: set appropriate memory limits with `--memory`, optimise your application's memory usage, and monitor with `docker stats`.

**Q35: What is a container registry?**
A storage and distribution system for Docker images. Docker Hub is the default public registry. Private registries include AWS ECR, Google GCR, Azure ACR, and self-hosted solutions like Harbor. In a CI/CD pipeline, you typically build an image, push it to a registry, then pull it on production servers.

**Q36: How would you debug a container that keeps crashing?**
Check logs with `docker logs container_name`. Check the exit code with `docker inspect`. Override the CMD to keep the container alive: `docker run -it image sh` (interactive shell instead of starting the app). Check resource limits with `docker stats`. Inspect the container's configuration with `docker inspect`. If it's an OOM kill, `docker inspect --format='{{.State.OOMKilled}}'`.

**Q37: What is Docker's layered filesystem?**
Docker images are built from layers stacked on top of each other. Each Dockerfile instruction creates a new read-only layer. When a container runs, Docker adds a thin writable layer on top (the container layer). This uses a copy-on-write strategy — files in the image layers are only copied to the writable layer when they're modified. This is why images are space-efficient — multiple containers from the same image share the read-only layers.

**Q38: How do you implement health checks for a backend API?**
Add a HEALTHCHECK instruction in the Dockerfile that hits a health endpoint, and create a `/health` endpoint in your API that checks critical dependencies (database connectivity, cache connectivity, disk space). The health check should be lightweight and fast. Orchestrators use this to determine if the container should receive traffic or be replaced.

**Q39: Explain how Docker handles environment variables across build and runtime.**
ARG variables exist only during build — they're used to parameterise the build process (choosing a base image version, setting build flags). ENV variables are set during build AND persist into the running container. Variables passed via `-e` at runtime override ENV values from the Dockerfile. Sensitive values should never be set with ARG or ENV (they're stored in image layers); use runtime `-e` flags or Docker secrets instead.

**Q40: What is Docker context?**
The build context is the set of files at the path or URL you pass to `docker build`. Docker sends this entire context to the daemon before building. A large build context (with node_modules, .git, etc.) makes builds slow. The `.dockerignore` file controls what's excluded from the context. Best practice is to keep the build context as small as possible.

---

### Scenario-Based Questions

**Q41: Your Node.js Docker container works locally but crashes in production. How do you troubleshoot?**
First, check logs (`docker logs`). Compare environment variables between local and production. Verify the image version is the same (use specific tags, not `latest`). Check resource limits — production might have memory/CPU constraints you don't have locally. Verify network connectivity — can the container reach the database, cache, external APIs? Check if there are filesystem/permission differences (especially if running as non-root in production but root locally).

**Q42: Your Docker build is taking 10 minutes. How do you speed it up?**
Review the Dockerfile instruction order — ensure dependency installation happens before code copying (layer caching). Check `.dockerignore` for missing exclusions (large build context). Use multi-stage builds so you're not rebuilding everything. Use BuildKit (`DOCKER_BUILDKIT=1`). Consider caching package manager downloads across builds with cache mounts. Split independent operations where possible. Use lighter base images.

**Q43: You need to run your backend API, PostgreSQL, Redis, and RabbitMQ for local development. How?**
Write a docker-compose.yml defining all four services. Use named volumes for PostgreSQL and Redis data. Use health checks to ensure PostgreSQL and Redis are ready before starting the API. Mount your source code via bind mount for live reloading. Use environment variables to configure connection strings, with service names as hostnames. Add profiles for optional tools like pgAdmin or Redis Commander.

**Q44: How do you handle database migrations in a Docker environment?**
Several approaches: Run migrations as part of the container startup script (before the main app starts), use an init container or entrypoint script that runs migrations then starts the app, or run migrations as a separate one-off container (`docker compose run api npm run migrate`). In production with orchestrators like Kubernetes, use init containers or migration jobs that run before the main deployment rolls out.

**Q45: Your container keeps getting OOM-killed. What do you do?**
Profile your application's memory usage to identify leaks. Check if the memory limit is too low for your workload. For Node.js, set `--max-old-space-size` to match your container's memory limit. Monitor with `docker stats`. Implement streaming/pagination instead of loading large datasets into memory. Consider horizontal scaling (more containers with reasonable limits) instead of vertical scaling (one container with huge memory).

**Q46: How do you handle secrets in a Docker-based microservices setup?**
Never bake secrets into images (via ENV in Dockerfile or COPY .env). Options from least to most secure: environment variables at runtime (visible in `docker inspect`), Docker Secrets in Swarm mode (mounted as files), external secret managers (AWS Secrets Manager, HashiCorp Vault) accessed at runtime, and sidecar containers that fetch and inject secrets. For local development, use env_file pointing to a .env file that's in .gitignore and .dockerignore.

**Q47: You need to deploy a new version of your API with zero downtime. How?**
Use an orchestrator (Kubernetes, ECS) with rolling updates. The orchestrator starts new containers with the new version, waits for health checks to pass, then stops old containers. Docker alone doesn't provide zero-downtime deployments — you need either an orchestrator or a reverse proxy (like Nginx or Traefik) that routes traffic and manages blue-green or canary deployments.

**Q48: Your Docker image contains a security vulnerability. What's your process?**
Scan the image with `docker scout cves` or Trivy. Identify which layer introduced the vulnerability (base image, OS package, or app dependency). Update the base image to the latest patched version. Update the vulnerable dependency. Rebuild and rescan. In CI/CD, add image scanning as a gate — fail the pipeline if critical vulnerabilities are found. Consider using distroless or minimal images to reduce the attack surface.

**Q49: How would you set up CI/CD for a Dockerized backend application?**
In the CI pipeline: run tests inside a container (for consistency), build the Docker image with a tag based on git commit SHA or semantic version, scan the image for vulnerabilities, push to a container registry (ECR, GCR). In the CD pipeline: pull the new image on the target environment, use rolling updates via the orchestrator to deploy. Add automated rollback if health checks fail after deployment.

**Q50: Your team has multiple microservices. How do you manage Docker images across them?**
Use a monorepo or consistent Dockerfile templates. Create a base image with common dependencies and build service-specific images on top. Use a private registry with clear naming conventions (e.g., `company/service-name:version`). Automate builds in CI/CD. Tag images with git commit SHA for traceability. Implement a cleanup policy to remove old images from the registry. Use Docker Compose for local development with all services.

---

### Quick-Fire / True-False / Conceptual

**Q51: Can a container modify its image?**
No. Images are read-only. When a container writes to the filesystem, it writes to a writable layer on top of the image layers (copy-on-write). The underlying image is never modified.

**Q52: What happens to data in a container when it stops?**
The data is preserved — a stopped container still has its writable layer. You can restart it with `docker start` and the data is still there. Data is only lost when the container is removed with `docker rm`.

**Q53: Can two containers use the same port?**
Inside their own namespace, yes — each can listen on port 3000. But you can't map the same host port to two containers. You'd need different host ports: `-p 3001:3000` and `-p 3002:3000`.

**Q54: What is the PID 1 problem in Docker?**
The first process in a container runs as PID 1. In Linux, PID 1 has special responsibilities — it should reap zombie processes and handle signals. If your application isn't designed for PID 1 (most aren't), zombie processes can accumulate. Solutions: use `--init` flag (runs tini as PID 1), or use a process manager. The exec form of CMD ensures your app is PID 1.

**Q55: What is `docker system prune` and when should you use it?**
It removes all unused containers, networks, and dangling images. With `-a`, it also removes unused images. With `--volumes`, it removes unused volumes. Use it when running low on disk space. Be careful in shared environments — it might remove things other developers are using.

**Q56: What is BuildKit?**
BuildKit is Docker's modern build engine. It offers parallel build stages, better caching (including cache mounts for package managers), secret mounts (pass secrets during build without baking them into layers), and faster builds overall. Enable it with `DOCKER_BUILDKIT=1` environment variable or set it as default in Docker's config.

**Q57: What is the difference between `docker compose up` and `docker compose run`?**
`docker compose up` starts all services defined in the compose file. `docker compose run` starts a single service and runs a one-off command — it doesn't start dependent services by default (use `--deps` if needed), doesn't map ports by default, and is useful for running migrations, seeds, tests, or any one-time command.

**Q58: What are Docker labels?**
Key-value metadata attached to images, containers, volumes, or networks. They're used for organisation, automation, and tooling. For example, labeling an image with its git commit, build date, or maintainer. Orchestrators and monitoring tools use labels to filter and manage resources.

**Q59: What is a Docker overlay network?**
A multi-host network that allows containers running on different Docker daemons to communicate as if they're on the same network. Used in Docker Swarm and Kubernetes. It uses VXLAN technology to encapsulate container traffic across hosts.

**Q60: How does Docker handle DNS resolution?**
On custom bridge networks, Docker runs an embedded DNS server at 127.0.0.11. When a container tries to resolve another container's name, the DNS server returns that container's IP address. This is why service names in docker-compose.yml work as hostnames. The default bridge network doesn't have this — you'd need to use `--link` (deprecated) or IP addresses.

---

### Backend-Specific Docker Questions

**Q61: How do you handle database connection pooling in a Dockerized backend?**
Connection pooling should be handled at the application level or with a dedicated pooler (like PgBouncer for PostgreSQL). In Docker, connection limits matter because containers can be scaled horizontally. If you run 5 API containers with 20 connections each, your database sees 100 connections. Set pool sizes based on total containers × pool size ≤ database max connections. Consider using a connection pooler as a separate container in front of the database.

**Q62: How do you run database migrations in a containerised setup?**
Create a migration command (e.g., `npm run migrate`) and run it as a one-off container: `docker compose run --rm api npm run migrate`. In CI/CD, run migrations as a separate step before deploying new containers. In Kubernetes, use init containers or Jobs. Never run migrations automatically on container startup in production — if multiple containers start simultaneously, you get race conditions.

**Q63: How do you handle file uploads in a Dockerized backend?**
Don't store uploads inside the container — they'd be lost when the container is replaced. Options: mount a volume for the upload directory, use a shared volume (NFS) if running multiple containers, or use object storage (S3, GCS) which is the recommended approach. With object storage, your container is truly stateless and can scale freely.

**Q64: How do you handle cron jobs in Docker?**
Don't install cron inside your API container (violates "one process per container"). Instead: run a separate container dedicated to cron jobs, use your orchestrator's scheduling feature (Kubernetes CronJobs, ECS Scheduled Tasks), or use a job queue (Bull, Celery) with a dedicated worker container. The cron container runs the same image but with a different CMD.

**Q65: How do you implement a development vs production Dockerfile?**
Use multi-stage builds and build targets: define a `development` stage with dev dependencies, hot reloading, and debugging tools, and a `production` stage with only production dependencies and optimised settings. Build with `docker build --target development` or `docker build --target production`. In docker-compose.yml, override the target per environment.

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package.json package-lock.json ./

FROM base AS development
RUN npm install
COPY . .
CMD ["npx", "nodemon", "src/server.js"]

FROM base AS production
RUN npm ci --only=production
COPY . .
USER appuser
CMD ["node", "src/server.js"]
```

**Q66: How do you handle logging in a Dockerized microservices architecture?**
Each service writes structured logs (JSON) to stdout/stderr. Docker captures these via the logging driver. Use a log aggregation stack — the EFK stack (Elasticsearch, Fluentd, Kibana) or a managed service (Datadog, CloudWatch). Include correlation IDs in logs to trace requests across services. Configure log rotation to prevent disk space issues (Docker's json-file driver has `max-size` and `max-file` options).

**Q67: How do you handle WebSocket connections with Docker?**
WebSocket connections are long-lived and stateful — you can't just move a client to a different container. Use sticky sessions (session affinity) in your load balancer so a client always connects to the same container. Alternatively, use Redis Pub/Sub or a message broker so any container can broadcast to WebSocket clients connected to other containers.

**Q68: What's the difference between `docker compose down` and `docker compose stop`?**
`docker compose stop` stops the containers but leaves them and the networks intact — you can `docker compose start` to resume. `docker compose down` stops and removes containers, removes networks, but keeps volumes and images. Adding `-v` to `down` also removes volumes (wipes data). Adding `--rmi all` also removes images.

**Q69: How do you optimise Node.js for running inside a Docker container?**
Set `NODE_ENV=production`. Set `--max-old-space-size` to about 75% of the container's memory limit (leave room for the OS and garbage collection). Handle SIGTERM for graceful shutdown. Use the exec form of CMD. Set the `UV_THREADPOOL_SIZE` if doing heavy I/O. Don't use PID file-based monitoring (use Docker health checks instead). Install production dependencies only.

**Q70: How do you monitor Docker containers in production?**
Use `docker stats` for basic real-time metrics. For production: deploy a monitoring stack with Prometheus (metrics collection), Grafana (visualisation), cAdvisor (container-specific metrics), and alerting. Alternatively, use managed solutions like Datadog, New Relic, or CloudWatch. Monitor CPU, memory, network I/O, disk I/O, container restart counts, and application-level metrics exposed via `/metrics` endpoint.

---

### Bonus: Rapid-Fire Round

**Q71: Default Docker network type?** — Bridge.

**Q72: How to see a container's IP address?** — `docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name`

**Q73: What does `-it` mean?** — `-i` (interactive, keep stdin open) + `-t` (allocate a TTY).

**Q74: What is `docker commit`?** — Creates an image from a running container's current state. Rarely used; Dockerfiles are preferred for reproducibility.

**Q75: What is the default restart policy?** — `no` (never restart).

**Q76: What is a bridge network?** — An internal private network on the host where containers can communicate. Containers on different bridge networks are isolated from each other.

**Q77: Can you map a UDP port?** — Yes: `-p 5000:5000/udp`.

**Q78: What does `docker cp` do?** — Copies files between a container and the host filesystem.

**Q79: What is `docker save` vs `docker export`?** — `docker save` saves an image (with layers and metadata) to a tar file. `docker export` exports a container's filesystem as a flat tar file (no layers, no metadata). Use `save/load` for images, `export/import` for container filesystems.

**Q80: What is `STOPSIGNAL` in a Dockerfile?** — Sets the system call signal that will be sent to the container to stop it. Default is SIGTERM. Some applications (like Nginx) need SIGQUIT for graceful shutdown.

---

> **Final Tip for Interviews:** As a backend developer, interviewers don't expect you to know Docker's source code. They want to know that you can containerise your applications, set up development environments with Compose, understand networking between services, handle data persistence, write efficient Dockerfiles, and debug container issues. Focus on practical scenarios over obscure trivia.
