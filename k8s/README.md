# Kubernetes Deployment 🚀

Welcome to the Kubernetes (K8s) deployment guide for the **Hospital Management System**. This directory contains all the necessary manifests to orchestrate, deploy, and manage the application and its database in a robust containerized environment.

## 🌟 Why Kubernetes?

Using Kubernetes for this project provides several key advantages:

*   **🛡️ Self-Healing:** Automatically restarts containers that fail, replaces containers, and kills containers that don't respond to user-defined health checks.
*   **📈 Scalability:** Easily scale the application up or down based on demand.
*   **🔌 Service Discovery & Load Balancing:** No need to modify your application to use an unfamiliar service discovery mechanism. K8s gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.
*   **🔐 Secret & Configuration Management:** Deploy and update secrets and application configuration without rebuilding your image and without exposing secrets in your stack configuration.

## 📂 Directory Structure

Here is the structure of the `k8s/` directory and the purpose of each file:

```text
k8s/
├── 01-secret.yaml       # 🔒 Stores sensitive data (Passwords, Connection Strings)
├── 02-configmap.yaml    # 📄 Stores configuration files (SQL Init Script)
├── 03-mssql.yaml        # 🗄️ Deployment & Service for SQL Server Database
├── 04-app.yaml          # 💻 Deployment & Service for the .NET Web Application
└── 05-db-init-job.yaml  # 🛠️ One-time Job to initialize the DB schema & data
```

## 📝 File Details

### 1. `01-secret.yaml` 🔒
Contains sensitive information like the `SA_PASSWORD` and the full `CONNECTION_STRING`.
> **Note:** This file is added to `.gitignore` to prevent accidental commit of credentials.

### 2. `02-configmap.yaml` 📄
Holds the content of the `init.sql` script. This allows us to inject the SQL script into the initialization job without baking it into the Docker image.

### 3. `03-mssql.yaml` 🗄️
*   **Deployment:** Runs the SQL Server 2022 container.
*   **PersistentVolumeClaim (PVC):** Ensures database data persists even if the Pod restarts.
*   **Service:** Exposes the database internally to the application on port `1433`.

### 4. `04-app.yaml` 💻
*   **Deployment:** Runs the .NET Core application. It pulls the connection string securely from the Secret.
*   **Service:** Exposes the application. We use `NodePort` (or `LoadBalancer`) to make the app accessible from outside the cluster.

### 5. `05-db-init-job.yaml` 🛠️
A transient Job that:
1.  Waits for the SQL Server to be ready.
2.  Connects to the database using `sqlcmd`.
3.  Executes the `init.sql` script mounted from the ConfigMap.

## 🚀 How to Run

1.  **Create Secrets:**
    ```bash
    kubectl apply -f k8s/01-secret.yaml
    ```

2.  **Create ConfigMap:**
    ```bash
    # Create from the local file
    kubectl create configmap init-sql-config --from-file=init.sql
    ```

3.  **Deploy Database:**
    ```bash
    kubectl apply -f k8s/03-mssql.yaml
    ```

4.  **Deploy Application:**
    ```bash
    kubectl apply -f k8s/04-app.yaml
    ```

5.  **Initialize Database:**
    ```bash
    kubectl apply -f k8s/05-db-init-job.yaml
    ```

6.  **Access the App:**
    ```bash
    kubectl port-forward service/hospital-app 8080:80
    ```
    Then open [http://localhost:8080](http://localhost:8080).
