# 💎 Luxe Jewelry Store - Full-Stack E-Commerce Application

## 📖 Project Overview

This is a complete e-commerce jewelry store application built with modern microservices architecture. The application demonstrates full-stack development principles, API design, user authentication, and containerization practices.

### 🏗️ Architecture

The application consists of three main components:

1. **React Frontend** (Port 3000) - User interface and shopping experience
2. **Main API Service** (Port 8000) - Product catalog and shopping cart management
3. **Authentication Service** (Port 8001) - User registration, login, and profile management

## 🎯 DevOps Learning Objectives

This course focuses on **DevOps practices** and **microservices architecture**. You will learn:

### **Core DevOps Skills**
- **Microservices Architecture**: Understanding how services communicate and work together
- **Containerization**: Docker fundamentals and best practices
- **Service Orchestration**: Managing multi-container applications
- **Environment Configuration**: Managing configs across different environments
- **Service Discovery**: How services find and communicate with each other

### **Practical Experience**
- **Running distributed applications** locally and in containers
- **Understanding frontend-backend separation** and API communication
- **Managing service dependencies** and startup order
- **Debugging containerized applications**
- **Container networking** and inter-service communication

## 🚀 Application Features

### Frontend Features
- **Modern UI/UX**: Vibrant design with glassmorphism effects and smooth animations
- **Product Catalog**: Browse jewelry items with detailed information
- **Shopping Cart**: Add, remove, and manage items in cart
- **User Authentication**: Registration, login, and profile management
- **Responsive Design**: Works on desktop, tablet, and mobile devices

### Backend Features
- **Product Management**: CRUD operations for jewelry items
- **Cart Management**: Session-based (anonymous) and user-based (authenticated) carts
- **User Authentication**: JWT-based secure authentication system
- **Password Security**: Bcrypt hashing with salt
- **API Documentation**: Auto-generated Swagger/OpenAPI documentation

## 📋 API Documentation

### Main API Service (Port 8000)

#### Product Endpoints
- `GET /api/products` - Retrieve all products
- `GET /api/products/{id}` - Get specific product details
- `GET /api/products?category={category}` - Filter products by category

#### Cart Endpoints
- `POST /api/cart/{session_id}/add` - Add item to cart
- `GET /api/cart?session_id={id}` - Get cart contents
- `PUT /api/cart/{session_id}/item/{item_id}` - Update item quantity
- `DELETE /api/cart/{session_id}/item/{item_id}` - Remove item from cart
- `DELETE /api/cart/{session_id}` - Clear entire cart

### Authentication Service (Port 8001)

#### Authentication Endpoints
- `POST /auth/register` - Register new user account
- `POST /auth/login` - User login (returns JWT token)
- `POST /auth/logout` - User logout
- `GET /auth/me` - Get current user profile
- `PUT /auth/me` - Update user profile
- `POST /auth/change-password` - Change user password

#### Admin Endpoints
- `GET /auth/users` - List all users (development only)
- `GET /health` - Service health check

## 🛠️ Technology Stack

### Frontend
- **React 18** - Modern React with hooks
- **CSS3** - Custom styling with glassmorphism effects
- **Fetch API** - HTTP client for API communication
- **LocalStorage** - JWT token persistence

### Backend
- **FastAPI** - Modern Python web framework
- **Pydantic** - Data validation and serialization
- **JWT** - JSON Web Tokens for authentication
- **Bcrypt** - Password hashing
- **HTTPX** - Async HTTP client for inter-service communication
- **Uvicorn** - ASGI server

## 🏃‍♂️ Running the Application Locally

### Prerequisites
- Python 3.8+
- Node.js 14+
- npm or yarn

### Step 1: Set Up Authentication Service

Navigate to the auth service directory and install dependencies. Start the authentication service on port 8001. Verify it's running by accessing the health endpoint and API documentation.

### Step 2: Set Up Main API Service

Navigate to the backend directory and install the required Python packages. Start the main API service on port 8000. Verify it's running by accessing the API documentation.

### Step 3: Set Up React Frontend

Navigate to the jewelry-store directory and install Node.js dependencies. Configure environment variables by copying the example file and modifying as needed. Start the React development server on port 3000.

### Step 4: Verify the Application

Access the application through your web browser. Test the following functionality:
- Browse products on the homepage
- Add items to cart
- Register a new user account
- Login with your credentials
- View your profile information

## 🔧 Configuration

### Environment Variables

The application uses environment variables for configuration:

#### Frontend (.env)
- `REACT_APP_API_BASE_URL` - Main API service URL
- `REACT_APP_AUTH_BASE_URL` - Authentication service URL

#### Backend Services
- `JWT_SECRET_KEY` - Secret key for JWT token signing
- `AUTH_SERVICE_URL` - Authentication service URL (for main API)

## 📊 Data Flow

### User Registration/Login Flow
1. User submits registration/login form in React frontend
2. Frontend sends request to Authentication Service
3. Authentication Service validates credentials and returns JWT token
4. Frontend stores token in localStorage
5. Subsequent requests include JWT token in headers

### Shopping Cart Flow
1. Anonymous users: Cart stored by session ID
2. Authenticated users: Cart stored by user ID
3. Main API service validates JWT tokens with Authentication Service
4. Cart data persists across user sessions when authenticated

## 🐳 Containerization Assignment

After successfully running the application locally, your next task is to containerize the entire application using Docker.

### Learning Goals
- Understand containerization concepts
- Create Dockerfile for each service
- Manage multi-container applications
- Configure container networking
- Handle environment variables in containers
- Implement container orchestration

### Assignment Requirements

#### Part 1: Individual Container Creation
Create Docker containers for each service:
1. **Authentication Service Container** - Containerize the Python FastAPI authentication service
2. **Main API Service Container** - Containerize the main API service
3. **Frontend Container** - Containerize the React application

#### Part 2: Container Configuration
- Configure appropriate base images for each service
- Set up proper working directories and file copying
- Install dependencies within containers
- Expose necessary ports
- Configure environment variables

#### Part 3: Multi-Container Orchestration
- Create a container orchestration solution to run all services together
- Configure container networking for inter-service communication
- Set up volume mounting for persistent data (if needed)
- Ensure proper startup order and dependencies

#### Part 4: Testing and Validation
- Verify all services start correctly in containers
- Test API endpoints through containerized services
- Validate frontend functionality with containerized backend
- Ensure user authentication works across containerized services

### Expected Deliverables
1. Individual Dockerfiles for each service
2. Container orchestration configuration file
3. Documentation explaining your containerization approach
4. Instructions for building and running the containerized application

### Evaluation Criteria
- **Functionality**: All services work correctly in containers
- **Best Practices**: Proper use of Docker best practices
- **Configuration**: Correct environment variable handling
- **Networking**: Proper inter-container communication
- **Documentation**: Clear instructions and explanations

## 🔍 Troubleshooting

### Common Issues

#### "Failed to fetch" errors
- Verify all services are running on correct ports
- Check CORS configuration in backend services
- Ensure environment variables are properly set

#### Authentication issues
- Verify JWT secret keys match between services
- Check token expiration times
- Ensure proper header formatting

#### Container networking issues
- Verify container port mappings
- Check inter-container communication configuration
- Ensure proper service discovery setup

## 📚 Additional Resources

### API Documentation
- Main API: `http://localhost:8000/docs`
- Auth Service: `http://localhost:8001/docs`

### Development Tools
- Browser Developer Tools for frontend debugging
- Postman or curl for API testing
- Docker Desktop for container management

## 🎓 Learning Extensions

After completing the basic assignment, consider these advanced topics:
- Database integration (PostgreSQL, MongoDB)
- Redis for session management
- Load balancing with multiple container instances
- CI/CD pipeline setup
- Kubernetes deployment
- Monitoring and logging solutions

## 🤝 Support

For technical questions or issues:
1. Check the troubleshooting section
2. Review API documentation
3. Examine browser console logs
4. Test individual services in isolation

Good luck with your development and containerization journey! 🚀
