# Three Tier Application Deployment – Kubernetes (Local)

## Architecture
- MySQL Database (StatefulSet + Headless Service)
- Spring Boot Backend (Deployment + NodePort)
- Frontend UI (Deployment + NodePort)

---

## Deployment Order
1. Database
2. Backend
3. Frontend

---

## Docker Images Used

The following container images are used in this three-tier Kubernetes deployment.

### Database (MySQL)

- Image: `mysql:8.0`
- Used in: `database/stateful-set.yml`
- Purpose: Stores application data using StatefulSet for stable identity and storage

---

### Backend (Spring Boot Application)

- Image: `simplybyte/simplybyte-backend:1.0`
- Used in: `backend/deployment.yml`
- Purpose: Handles business logic and communicates with MySQL database

---

### Frontend (UI Application)

- Image: `simplybyte/simplybyte-ui:latest`
- Used in: `frontend/deployment.yml`
- Purpose: Provides user interface and communicates with backend service
---


## Step 1: Database Deployment

    cd database
    kubectl apply -f secret.yml
    kubectl apply -f headless-service.yml
    kubectl apply -f stateful-set.yml

Verify:

    kubectl get pods

---

## Step 2: Verify MySQL

    kubectl exec -it mysql-0 -- sh
    mysql -u root -p

Password:

    simplybyte

Check:

    SHOW DATABASES;

To Select Database:

    USE simplybyte;

To view all tables:
    
    SHOW TABLES;

---

## MySQL DNS Used by Backend

    mysql-0.mysql

JDBC URL:

    jdbc:mysql://mysql-0.mysql:3306/simplybyte

## MySQL JDBC URL Explanation

The backend Spring Boot application connects to MySQL using the following JDBC URL:

    jdbc:mysql://mysql-0.mysql:3306/simplybyte

Let’s break this down part by part.

---

### `jdbc:mysql://`

- `jdbc`  
  Java Database Connectivity API used by Java/Spring Boot applications.

- `mysql`  
  Specifies that the database type is **MySQL**.

---

### `mysql-0.mysql`

This is the **Kubernetes DNS name** of the MySQL pod.

- `mysql-0`  
  First pod created by the MySQL StatefulSet.

- `mysql`  
  Headless Service name.

Because this is a StatefulSet with a Headless Service, Kubernetes provides
**stable DNS names** for each pod.

Kubernetes automatically resolves this internally, so no IP address is required.

| Part | Meaning |
|----|----|
| jdbc | Java database connection |
| mysql | Database type |
| mysql-0 | StatefulSet pod name |
| mysql | Headless service name |
| 3306 | MySQL port |
| simplybyte | Database name |

This approach is **recommended for databases in Kubernetes**.


---

## Step 3: Backend Deployment

    cd ../backend
    kubectl apply -f deployment.yml
    kubectl apply -f service.yml

To Access Backend through Node Port:

    http://localhost:30081

To Check if the service is alive:

    http://localhost:30081/health

Check logs:

    kubectl logs <backend-pod-name>

---

## Step 4: Frontend Deployment

    cd ../frontend
    kubectl apply -f config.yml
    kubectl apply -f deployment.yml
    kubectl apply -f service-nodeport.yml

---

## Step 5: Verify Frontend ConfigMap Mount

    kubectl exec -it <frontend-pod-name> -- sh

    cat /usr/share/nginx/html/config.json

---

## Access Application

Frontend URL:

    http://localhost:30080

---

## Kubernetes Concepts Covered
- StatefulSet
- Headless Service
- ConfigMap
- Secret
- NodePort
- Pod DNS
- Volume Mounts
- kubectl exec
- kubectl logs
