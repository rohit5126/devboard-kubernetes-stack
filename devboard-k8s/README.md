# 🚀 DevBoard Kubernetes Deployment

This repository contains all the Kubernetes manifests required to deploy the **DevBoard application**, including:

* Frontend (Node.js / Vite)
* Backend (Go API)
* PostgreSQL Database
* Services for internal/external communication
* Secrets for sensitive data
* ConfigMap for database initialization
* Database schema and seed data

---

## 📁 Repository Structure

```
.
├── frontend-deployment.yml
├── backend-deployment.yml
├── postgres.yml
├── frontend-service.yml
├── backend-service.yml
├── pg-service.yml
├── secrets.yml
├── 01_schema.sql
├── 02_seed.sql
└── README.md
```

---

## 🧩 Components Overview

### 🔹 Frontend

* Built using Node.js (Vite)
* Exposed via **NodePort**
* Accessible externally on:

  * `http://localhost:8080` (via Kind config mapping to NodePort `30080`)
* Communicates with backend using Kubernetes service

### 🔹 Backend

* Built using Go
* Exposes API at:

  * `http://backend:8080`
* Handles business logic and database operations

### 🔹 PostgreSQL

* Internal database service
* Initialized with schema and seed data using ConfigMap

---

## 🔐 Secrets

Sensitive data such as database credentials are stored securely using Kubernetes Secrets.

Example:

* `POSTGRES_USER`
* `POSTGRES_PASSWORD`
* `POSTGRES_DB`

---

## ⚙️ ConfigMap for Database Initialization

We use a ConfigMap to inject SQL files into the PostgreSQL container for initialization.

### Create ConfigMap

```bash
kubectl create configmap pg-config \
  --from-file=01_schema.sql=01_schema.sql \
  --from-file=02_seed.sql=02_seed.sql
```

This will:

* Load database schema (`01_schema.sql`)
* Insert initial data (`02_seed.sql`)

---

## 🛠️ Deployment Steps

### 1️⃣ Create Secrets

```bash
kubectl apply -f secrets.yml
```

---

### 2️⃣ Create ConfigMap

```bash
kubectl create configmap pg-config \
  --from-file=01_schema.sql=01_schema.sql \
  --from-file=02_seed.sql=02_seed.sql
```

---

### 3️⃣ Deploy PostgreSQL

```bash
kubectl apply -f postgres-statefulset.yml 
```

---

### 4️⃣ Deploy Backend

```bash
kubectl apply -f backend-deployment.yml
kubectl apply -f backend-service.yml
```

---

### 5️⃣ Deploy Frontend

```bash
kubectl apply -f frontend-deployment.yml
kubectl apply -f frontend-service.yml
```

---

## 🌐 Accessing the Application

### Frontend (External Access)

* URL: **http://localhost:8080**
* Backed by NodePort: **30080**
* Configured via Kind cluster port mapping

### Backend (Internal Access)

* URL: **http://backend:8080**
* Used by frontend for API calls

### PostgreSQL

* Accessible only inside cluster via service

---

## 🔗 Internal Service Communication

| Component | Service Name     | Endpoint              |
| --------- | ---------------- | --------------------- |
| Frontend  | frontend-service | External via NodePort |
| Backend   | backend          | http://backend:8080   |
| Database  | postgres         | Internal              |

---

## Clean up commands

```bash

 kubectl delete deployments backend-deployment frontend-deployment
 kubectl delete statefulsets postgres
 kubectl delete svc backend frontend-service 
 kubectl delete configmaps pg-config 
 kubectl delete secrets pg-sec

 ``` 

## ⚠️ Important Notes

* Frontend must call backend using:

  ```
  http://backend:8080
  ```

  ❌ Do NOT use `localhost` inside containers

* Ensure:

  * Services are correctly named (`backend`, `postgres`)
  * Pods are in same namespace
  * ConfigMap is created before PostgreSQL starts

* Apply resources in correct order:

  ```
  Secrets → ConfigMap → DB → Backend → Frontend
  ```

---

## 📌 Future Improvements

* Add Helm charts for easier deployment
* Add Ingress for domain-based routing
* Implement HPA (Horizontal Pod Autoscaler)
* Add CI/CD pipeline integration
* Add monitoring (Prometheus + Grafana)

---

## 👨‍💻 Author

**Rohit**
DevOps Enthusiast | Kubernetes | Docker | CI/CD

---

⭐ Star the repo if you found this useful!
