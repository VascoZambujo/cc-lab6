# Cloud Computing – Lab 04: Edge Routing and API Gateways
**Master in Computer Engineering - Mobile Computing (2026)**

**Date:** [Insert Date]  
**Objective:** In this lab, you are acting as a Platform Engineer. The software development team has provided you with the source code for a new microservice architecture. Your mission is to containerize the applications, orchestrate them alongside a database, and secure the entire ecosystem behind an **API Gateway** using public wildcard domains (`*.meicm.pt`).

**Prerequisites:** Visual Studio Code, Docker CLI, and the provided `lab04-source.zip` file.

---

### Phase 1: Environment Setup & Architecture Review
The development team has provided you with `lab04-source.zip`. Extract it to your workspace. 

You should see two directories:
1. `/frontend`: Contains a static `index.html` dashboard.
2. `/api`: Contains a Node.js Express application that requires a Redis database.

**Your Architectural & Networking Constraints:**
* The API must connect to a Redis database using the `REDIS_URL` environment variable.
* **The Zero-Trust Network Rule:** The Frontend, API, and Database containers **must not** expose any ports to the host machine. 
* All external traffic must flow through a single Reverse Proxy / API Gateway (e.g., Nginx Proxy Manager).
* The Gateway must be the *only* container allowed to bind to the host's ports (80 for HTTP, 81 for Admin UI).

---

### Phase 2: The Containerization Challenge (30 mins)
Your first task is to autonomously write the OCI-compliant `Dockerfile`s for the provided source code.

**Task 2.1: The Frontend Container**
Inside the `/frontend` directory, create a `Dockerfile` that uses an `nginx:alpine` base image to serve the `index.html` file.

**Task 2.2: The API Container**
Inside the `/api` directory, create a highly optimized, **Multi-Stage `Dockerfile`** for the Node.js application. 
* *Requirement 1:* Utilize Layer Caching for the `package.json` dependencies.
* *Requirement 2:* Use `node:18-alpine` for both the build and production stages.
* *Requirement 3:* The production container must run as the non-root `node` user.

---

### Phase 3: The Network & Orchestration Challenge (40 mins)
Now that your containers can be built, you must orchestrate the ecosystem and design the network topology. 

Create a `docker-compose.yml` file in the root directory. You must define **four** services: your frontend, your API, your Redis cache, and an API Gateway (we recommend using the `jc21/nginx-proxy-manager:latest` image).

**Focus on Docker Networking:**
In a Compose environment, Docker automatically creates a custom bridge network for your stack. This provides internal DNS resolution. 
* Your API Gateway container will not be able to route traffic using `localhost`, because `localhost` inside the Gateway container refers to itself! 
* To route traffic to the API or Frontend, the Gateway must use the exact **service names** you define in this YAML file. 
* Ensure your database uses a Docker Volume to persist data across restarts.

Once your YAML file is complete, boot the architecture in the background:
```bash
docker compose up -d --build