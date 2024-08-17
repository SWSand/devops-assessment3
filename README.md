# 🎥 Movie NodeApp - Dockerized Three-Tier Application

Welcome to the **Movie NodeApp** project! This repository contains a fully Dockerized three-tier application built with Node.js. The project aims to familiarize with containerization using Docker and demonstrates a CI/CD pipeline with GitHub Actions.

---

## 📜 Table of Contents

- [Project Overview](#project-overview)
- [Environment Variables](#environment-variables)
- [Docker Setup](#docker-setup)
  - [Choosing Base Images](#choosing-base-images)
  - [Dockerfiles](#dockerfiles)
  - [Docker Compose Configuration](#docker-compose-configuration)
- [Database Initialization](#database-initialization)
- [CI/CD Pipeline with GitHub Actions](#cicd-pipeline-with-github-actions)
- [DockerHub Deployment](#dockerhub-deployment)
- [Commands Summary](#commands-summary)
- [Conclusion](#conclusion)

---

## 📖 Project Overview

The **Movie NodeApp** is a three-tier application consisting of:

1. **Frontend**: A Node.js application that serves a user interface.
2. **Backend**: A Node.js application that provides a REST API to the frontend.
3. **Database**: A PostgreSQL database that stores movie data.

This project demonstrates the following critical concepts:

- Replacing hardcoded values with environment variables.
- Dockerizing frontend and backend components.
- Configuring Docker Compose to manage multi-container applications.
- Setting up a CI/CD pipeline using GitHub Actions.
- Pushing Docker images to DockerHub.

---

## ⚙️ Environment Variables

Replaced all hardcoded values in the frontend and backend with environment variables to make the application more configurable and secure.

Environment variables used:

```plaintext
DB_PORT=xxxx
POSTGRES_USER=xxxxxxx
POSTGRES_PASSWORD=xxxxxxxxxx
POSTGRES_DB=xxxxxxxxxxxxxxx

DATABASE_URL=xxxxxxxxxxxxxxxxxxx
BACKEND_PORT=xxxx

REST_API_URL=xxxxxxxxxxxxxxxxxx
FRONTEND_PORT=xxxx
```

---


# 🐳 Docker Setup
# 🔍 Choosing Base Images
Selected a Node.js image (node:latest) for both the frontend and backend since the apps are both index.js files using express.

# 📄 Dockerfiles
Created separate Dockerfiles for the frontend and backend services.

<details>
<summary><strong>Backend Dockerfile</strong></summary>

    FROM node:latest
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE BACKEND_PORT
    CMD ["node", "index.js"]
</details>

<details>
<summary>
<strong> Frontend Dockerfile</strong>
</summary>

    FROM node:latest
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE FRONTEND_PORT
    CMD ["node", "index.js"]
</details>

---

# 🛠 Docker Compose Configuration
The docker-compose.yml file orchestrates all three parts of the application: frontend, backend, and PostgreSQL database.

Key Features:

Network Creation: All services are connected via a custom Docker bridge network, allowing seamless communication beten containers.
Volume Configuration: Used Docker volumes to persist PostgreSQL data, ensuring data is not lost when containers are stopped or removed.
<details>
<summary><strong>Docker Compose File</strong></summary>

    services:
    db:
        image: postgres:latest
        container_name: postgres
        ports:
        - "${DB_PORT}:${DB_PORT}"
        volumes:
        - postgres_data:/var/lib/postgresql/data/
        - ./init_sql_scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
        environment:
        POSTGRES_USER: ${POSTGRES_USER}
        POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
        POSTGRES_DB: ${POSTGRES_DB}
        networks:
        - ass3_network

    backend:
        build:
        context: ./backend
        dockerfile: Dockerfile
        container_name: backend
        ports:
        - "${BACKEND_PORT}:${BACKEND_PORT}"
        volumes:
        - ./backend:/app
        - /app/node_modules
        depends_on:
        - db
        environment:
        DATABASE_URL: ${DATABASE_URL}
        BACKEND_PORT: ${BACKEND_PORT}
        networks:
        - ass3_network

    frontend:
        build:
        context: ./frontend
        dockerfile: Dockerfile
        container_name: frontend
        ports:
        - "${FRONTEND_PORT}:${FRONTEND_PORT}"
        volumes:
        - ./frontend:/app
        - /app/node_modules
        depends_on:
        - backend
        environment:
        REST_API_URL: ${REST_API_URL}
        FRONTEND_PORT: ${FRONTEND_PORT}
        networks:
        - ass3_network

    networks:
    ass3_network:
        driver: bridge

    volumes:
    postgres_data: {}
</details>

---


# 🔗 Port Mapping Explained
Port mapping allows us to expose the services running inside Docker containers to the host machine. For example:

    Backend: Mapped ${BACKEND_PORT} inside the container to ${BACKEND_PORT} on the host machine.
    Frontend: Mapped ${FRONTEND_PORT} inside the container to ${BACKEND_PORT} on the host machine.
    Database: Mapped ${DB_PORT} inside the container to ${BACKEND_PORT} on the host machine.

# 📊 Database Initialization
The PostgreSQL database is automatically initialized with a sample dataset using the init.sql script, which is mounted into the container through Docker Compose.

    volumes:
    - ./init_sql_scripts/init.sql:/docker-entrypoint-initdb.d/init.sql

This ensures that the database is preloaded with data when the container starts, making it ready for use by the backend.

# 🚀 CI/CD Pipeline with GitHub Actions
Set up a GitHub Actions workflow to automate the building and pushing of Docker images to DockerHub.

<details>
<summary><strong>GitHub Actions Workflow</strong></summary>

    name: CI

    on:
    push:
        branches:
        - main
        paths:
        - 'frontend/**'
        - 'backend/**'

    jobs:
    build_and_push:
        runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to DockerHub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push backend image
      uses: docker/build-push-action@v2
      with:
        context: ./backend
        push: true
        tags: swsand/movie-nodeapp-backend:latest

    - name: Build and push frontend image
      uses: docker/build-push-action@v2
      with:
        context: ./frontend
        push: true
        tags: swsand/movie-nodeapp-frontend:latest
</details>

---


# 🔑 Setting Up Secrets
Added DockerHub credentials as GitHub Secrets (DOCKER_USERNAME and DOCKER_PASSWORD) to securely handle authentication.

# 🐋 DockerHub Deployment
Once the images are built and tested locally, pushed them to DockerHub using the following commands:

    docker login
    docker tag <image_id> dockerhub-username/movie-nodeapp-backend:tag
    docker tag <image_id> dockerhub-username/movie-nodeapp-frontend:tag
    docker push dockerhub-username/movie-nodeapp-backend:tag
    docker push dockerhub-username/movie-nodeapp-frontend:tag

---

# 🛠 Commands Summary
Here’s a quick rundown of all the critical commands  ran:

Building and Running Containers:

    docker-compose build
    docker-compose up -d

Testing Connectivity and Logs:

    docker exec -it <container_name> /bin/bash
    docker logs <container_name>

Pushing to DockerHub:

    docker login
    docker tag <image_id> dockerhub-username/movie-nodeapp-backend:tag
    docker tag <image_id> dockerhub-username/movie-nodeapp-frontend:tag
    docker push dockerhub-username/movie-nodeapp-backend:tag
    docker push dockerhub-username/movie-nodeapp-frontend:tag

Setting Up GitHub Actions:

-Create .github/workflows/docker-image.yml and add DockerHub credentials as GitHub Secrets.