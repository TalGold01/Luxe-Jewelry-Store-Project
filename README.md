# 💎 Luxe Jewelry Store - Full-Stack E-Commerce Application
> **A Containerized Microservices E-Commerce Platform**

![React](https://img.shields.io/badge/Frontend-React_18-blue)
![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688)
![Docker](https://img.shields.io/badge/Infrastructure-Docker-2496ED)
![CI/CD](https://img.shields.io/badge/Pipeline-Jenkins_%7C_GitHub_Actions-black)

## 📖 Project Overview
This project is a complete e-commerce jewelry store application built with a modern microservices architecture. Designed to demonstrate full-stack development, decoupled API design, secure user authentication, and production-grade containerization practices.

## 🏗️ Architecture
The system is decoupled into three primary microservices:
1. **React Frontend (Port 3000):** User interface, product catalog, and shopping cart experience.
2. **Main API Service (Port 8000):** Product catalog management and session/authenticated cart state.
3. **Authentication Service (Port 8001):** User registration, secure login, and JWT-based profile management.

## 🛠️ Technology Stack
* **Frontend:** React 18, CSS3 (Glassmorphism), Fetch API, LocalStorage
* **Backend:** Python 3.8+, FastAPI, Pydantic, JWT (JSON Web Tokens), Bcrypt, HTTPX, Uvicorn
* **DevOps & Infrastructure:** Docker, Docker Compose
* **CI/CD Pipelines:** Jenkins, GitHub Actions, Docker Hub

---

## 🐳 Quick Start: Docker Deployment (Recommended)

The entire application stack is containerized. Docker Compose is configured to spin up all microservices within an isolated bridge network, handling internal routing automatically.

**Prerequisites:** [Docker](https://www.docker.com/products/docker-desktop/) and Docker Compose installed.

```bash
# Clone the repository and build the cluster
git clone [https://github.com/TalGold01/Luxe-Jewelry-Store-Project.git](https://github.com/TalGold01/Luxe-Jewelry-Store-Project.git)
cd Luxe-Jewelry-Store-Project
docker-compose up --build

```markdown
Services Started:

    Frontend UI: http://localhost:3000

    Main API Backend: http://localhost:8000

    Authentication Service: http://localhost:8001

    💻 Local Development Setup (Manual)

If you wish to run the application locally outside of Docker for active development, follow these steps.

Prerequisites: Python 3.8+ and Node.js 14+
Step 1: Authentication Service

```bash
cd src/auth-service
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Secure configuration: copy the template and inject your local secrets
cp .env.example .env

# Launch the service
uvicorn main:app --reload --port 8001

```markdown
Step 2: Main API Service

```bash
cd ../backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Secure configuration: copy the template
# Note: Ensure the JWT_SECRET_KEY matches the one you set in the auth-service
cp .env.example .env

# Launch the service
uvicorn main:app --reload --port 8000

```markdown
Step 3: React Frontend

```bash

cd ../jewelry-store
npm install

# Setup local routing variables mapping to localhost ports
cp .env.example .env

# Launch the React dev server
npm start

```markdown
Step 4: Verify Deployment

Open your browser at http://localhost:3000 and test the following application flows:

    Browse the dynamic product catalog.

    Add items to the session cart.

    Register a new user profile and log in securely.

📡 API Documentation & Endpoints

Both backend services utilize FastAPI's automatic OpenAPI documentation. Once the application is running, you can interact with the APIs directly via the Swagger UI:

    Main API Docs: http://localhost:8000/docs

    Auth Service Docs: http://localhost:8001/docs

Core Data Flow

    Authentication: User logs in via the Frontend -> Auth Service validates and returns a JWT -> Frontend stores the token in LocalStorage.

    State Management: Anonymous users utilize a session ID for cart storage. Authenticated users pass the JWT to the Main API, which verifies it against the Auth Service to persist cart data across devices.