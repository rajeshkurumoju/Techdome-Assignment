# Multi-Container Application Deployment with Docker Compose and Kubernetes

## Introduction
This project demonstrates the deployment of a multi-container application with at least three containers (frontend, backend, and database). We use Docker Compose for local orchestration and Kubernetes for deploying it to a local Kubernetes cluster (Minikube).

## Tools and Technologies
- **Docker**: For creating and managing containers.
- **Docker Compose**: For defining and running multi-container applications.
- **Kubernetes**: For container orchestration and management.
- **Minikube**: A local Kubernetes cluster for testing.
- **Terraform (Bonus)**: For automating infrastructure and scaling.

## Architecture
The application consists of three main services:
- **Frontend**: A UI (React, Angular, Vue.js, etc.) that interacts with the backend API.
- **Backend**: A REST API or GraphQL server that connects to a database.
- **Database**: PostgreSQL for storing application data.

## Directory Structure
/frontend /backend /database (e.g., PostgreSQL)

## Step 1: Docker Compose Setup
Create a `docker-compose.yml` file in your project root directory to define all three services.

### Sample `docker-compose.yml`
```yaml
version: "3.8"

services:
  frontend:
    build:
      context: ./frontend
    ports:
      - "8080:80"
    depends_on:
      - backend
    networks:
      - app_network
    environment:
      - API_URL=http://backend:5000

  backend:
    build:
      context: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - database
    networks:
      - app_network
    environment:
      - DB_HOST=database
      - DB_USER=admin
      - DB_PASSWORD=secret
      - DB_NAME=app_db

  database:
    image: postgres:13
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: app_db
    ports:
      - "5432:5432"
    networks:
      - app_network
    volumes:
      - postgres_data:/var/lib/postgresql/data

networks:
  app_network:
    driver: bridge

volumes:
  postgres_data:
```

- ## Frontend
This will be a containerized version of the frontend. It depends on the backend service and exposes port 8080.

- ## Backend
This will be the API server container. It depends on the database service and exposes port 5000.

- ## Database
PostgreSQL container for storing application data.

This Docker Compose file will allow us to run the three containers locally, ensuring the frontend, backend, and database can communicate with each other via the app_network.

## Step 2: Kubernetes Deployment Manifests
Define Kubernetes deployment manifests for each service (frontend, backend, and database).
- Frontend Deployment (`Frontend-Deployment.yml`)
  
### `Frontend-Deployment.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: yourfrontendimage:latest
          ports:
            - containerPort: 8080
          env:
            - name: API_URL
              value: "http://backend:5000"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

- Backend Deployment (`Backend-Deployment.yml`)
  
### `Bankend-Deployment.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: yourbackendimage:latest
          ports:
            - containerPort: 5000
          env:
            - name: DB_HOST
              value: "database"
            - name: DB_USER
              value: "admin"
            - name: DB_PASSWORD
              value: "secret"
            - name: DB_NAME
              value: "app_db"
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
    - port: 5000
      targetPort: 5000
  type: ClusterIP
```

- Database Deployment (`Database-Deployment.yml`)
  
### `Database-Deployment.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
        - name: database
          image: postgres:13
          env:
            - name: POSTGRES_USER
              value: "admin"
            - name: POSTGRES_PASSWORD
              value: "secret"
            - name: POSTGRES_DB
              value: "app_db"
          ports:
            - containerPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  selector:
    app: database
  ports:
    - port: 5432
      targetPort: 5432
  type: ClusterIP
```

## Step 3: Deploy to Kubernetes
Follow these steps to deploy the application to Kubernetes.

- **Start Minikube**

```bash
minikube start
```

- **Set up `kubectl` to use Minikube**

```bash
kubectl config use-context minikube
```

- **Deploy the Backend, Frontend, and Database**

```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f database-deployment.yaml
```

- **Check the Status of Deployments**

```bash
kubectl get pods
kubectl get svc
```

- **Access the Frontend Service**
Use `kubectl port-forward` to access the frontend service from your local machine.
```bash
kubectl port-forward svc/frontend 8080:80
```
open a browser and visit http://localhost:8080 to see the frontend application communicating with the backend and using the database.

## Step 3: Documentation
Ensure the documentation includes:

- **Architecture Overview**: Describe the services involved (Frontend, Backend, Database).
- **Deployment Strategy**: Explain how Docker Compose was used for local deployment and how Kubernetes was used for scalable production deployment.
- **Instructions**: Provide clear steps for building and deploying the application.
- **Testing**: Include instructions for testing the application locally and on Kubernetes.

## Step 4: Bonus Points

- **Automating Infrastructure with Terraform**:
Write Terraform scripts to manage Kubernetes clusters.

- **Unit Testing for Terraform**:
Use terraform validate and terraform plan to ensure that your infrastructure code works as expected.

- **Rollback Strategy**:
Use Helm for Kubernetes management, which provides easy rollback capabilities, or use Kubernetes deployment strategies
