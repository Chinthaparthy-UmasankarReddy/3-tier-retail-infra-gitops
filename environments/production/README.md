```python
import os

# Define the complete, highly technical README.md content for the retail-infra-gitops repo
readme_content = """# 🛒 Retail Infrastructure GitOps Blueprint (`retail-infra-gitops`)

This repository serves as the definitive cloud-native declarative GitOps orchestration layer for the production-grade, 3-tier retail microservices platform simulating an enterprise architecture layout. It leverages an automated declarative framework utilizing **ArgoCD** on **AWS EKS** matching secure cloud patterns, backed by persistent data storage inside an **AWS RDS PostgreSQL** instance in Mumbai (`ap-south-1`).

---

## 🏗️ Architecture Blueprint

The entire application lifecycle is fully managed via declarative manifests. Secure configurations and environment variables are synchronized dynamically at execution runtime using the **External Secrets Operator (ESO)** linking to **AWS Secrets Manager**.


```

```text
Successfully generated README.md

```text
       [ ArgoCD Control Loop Engine ]
                    │
                    ▼ (Applies Manifest Declarations)
┌─────────────────── EKS Cluster Infrastructure (production) ───────────────────┐
│                                                                               │
│  ┌───────────────────────┐            ┌──────────────────────────────────┐    │
│  │  retail-frontend Pod  │            │        retail-backend Pod        │    │
│  │   (ClusterIP: Port 80)│            │      (Engine Node: Port 8080)    │    │
│  └───────────▲───────────┘            └─────────────────▲────────────────┘    │
│              │                                          │                     │
└──────────────┼──────────────────────────────────────────┼─────────────────────┘
               │ (Port-Forward Loop)                      │ (Port-Forward Loop)
               │                                          │
    [Local Browser: :3000] ──(Dynamic Cross-Origin Fetch)─►[Local Loop: :5000]
                                                          │
                                                (Secure Port 5432 Egress)
                                                          │
                                                          ▼
                                             ┌──────────────────────────┐
                                             │    AWS RDS PostgreSQL    │
                                             │ (retail_prod @ ap-south-1)
                                             └──────────────────────────┘

```

---

## 📂 Repository Directory Structure

```text
.
├── apps/
│   └── application.yaml          # ArgoCD Enterprise Application Root Manifest
├── infrastructure/
│   ├── app-db-credentials.yaml   # External Secrets Operator (ESO) Mapping Definition
│   └── secret-store.yaml         # ESO ClusterSecretStore Bridge to AWS Secrets Manager
└── manifests/
    ├── backend-deploy.yaml       # Node.js API Service Deployment Configuration
    ├── frontend-deploy.yaml      # React Web Client UI Deployment Configuration
    └── db-bootstrap-job.yaml     # Lifecycle Hook: Automated Schema Seeder (Method-1)

```

---

## 🛠️ Production Manifest Blueprints

### 1. Database Automated Bootstrap Lifecycle Hook (`manifests/db-bootstrap-job.yaml`)

This Kubernetes `Job` intercepts the ArgoCD sync validation cycle to ensure your table mapping rules and catalog records exist natively inside AWS RDS by default on initialization.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: retail-db-bootstrap
  namespace: production
  annotations:
    argocd.argoproj.io/hook: Sync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    metadata:
      name: retail-db-bootstrap
    spec:
      containers:
      - name: bootstrap-engine
        image: postgres:15-alpine
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: app-db-credentials
              key: DB_PASSWORD
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-db-credentials
              key: DB_USER
        command: ["/bin/sh", "-c"]
        args:
        - |
          echo "Connecting to AWS RDS Instance..."
          psql -h production-postgres-db.c3u4eyeaiq53.ap-south-1.rds.amazonaws.com \\
               -U "$DB_USER" -d retail_prod -c "
          
          CREATE TABLE IF NOT EXISTS products (
              id SERIAL PRIMARY KEY,
              sku VARCHAR(50) UNIQUE NOT NULL,
              name VARCHAR(255) NOT NULL,
              description TEXT,
              price NUMERIC(10, 2) NOT NULL,
              image_url VARCHAR(512),
              category VARCHAR(100)
          );

          INSERT INTO products (sku, name, description, price, image_url, category) VALUES
          ('GROC-RICE-001', 'Premium Basmati Rice', 'Long-grain aromatic rice, perfect for biryani.', 149.00, '[https://images.unsplash.com/photo-1586201375761-83865001e31c](https://images.unsplash.com/photo-1586201375761-83865001e31c)', 'Groceries'),
          ('GROC-OIL-002', 'Organic Sunflower Oil', '100% pure cold-pressed healthy cooking oil.', 195.00, '[https://images.unsplash.com/photo-1474979266404-7eaacbcd87c5](https://images.unsplash.com/photo-1474979266404-7eaacbcd87c5)', 'Groceries'),
          ('FRUIT-MNG-003', 'Fresh Alphonso Mangoes', 'Sweet and juicy seasonal mangoes (1 Dozen).', 450.00, '[https://images.unsplash.com/photo-1553279768-865429fa0078](https://images.unsplash.com/photo-1553279768-865429fa0078)', 'Fruits'),
          ('GROC-ATTA-004', 'Whole Wheat Atta (5kg)', 'High-fiber natural whole wheat flour.', 260.00, '[https://images.unsplash.com/photo-1509440159596-0249088772ff](https://images.unsplash.com/photo-1509440159596-0249088772ff)', 'Groceries')
          ON CONFLICT (sku) DO NOTHING;
          "
          echo "Database Bootstrap execution completed successfully!"
      restartPolicy: OnFailure

```

### 2. Core Business API Layer (`manifests/backend-deploy.yaml`)

Includes aligned multi-level selector parameters alongside robust TCP-socket availability validation testing criteria.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: retail-backend
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: retail-backend
  template:
    metadata:
      labels:
        app: retail-backend
    spec:
      containers:
      - name: api
        image: [154290454465.dkr.ecr.ap-south-1.amazonaws.com/retail-backend:latest](https://154290454465.dkr.ecr.ap-south-1.amazonaws.com/retail-backend:latest)
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          value: "production-postgres-db.c3u4eyeaiq53.ap-south-1.rds.amazonaws.com"
        - name: DB_NAME
          value: "retail_prod"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "[http://opentelemetry-collector.monitoring.svc.cluster.local:4317](http://opentelemetry-collector.monitoring.svc.cluster.local:4317)"
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-db-credentials
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-db-credentials
              key: DB_PASSWORD
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "250m"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: retail-backend-service
  namespace: production
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
  selector:
    app: retail-backend

```

### 3. Front-End Web UI Layer (`manifests/frontend-deploy.yaml`)

Injects the concrete runtime environment routing paths required to cleanly isolate the REST dynamic fetch operations from local UI proxy layers.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: retail-frontend
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: retail-frontend
  template:
    metadata:
      labels:
        app: retail-frontend
    spec:
      containers:
      - name: web-ui
        image: [154290454465.dkr.ecr.ap-south-1.amazonaws.com/retail-frontend:latest](https://154290454465.dkr.ecr.ap-south-1.amazonaws.com/retail-frontend:latest)
        ports:
        - containerPort: 80
        env:
        - name: REACT_APP_API_URL
          value: "http://localhost:5000"
        - name: NODE_BE_URL
          value: "http://localhost:5000"
        resources:
          limits:
            cpu: "300m"
            memory: "256Mi"
          requests:
            cpu: "150m"
            memory: "128Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: retail-frontend-service
  namespace: production
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: retail-frontend

```

---

## 🚀 Step-by-Step Initial Deployment Guide

Follow these steps to deploy or re-bootstrap the platform workspace cleanly on your local machine configuration:

### 1. Synchronize GitOps State Variables

Navigate into your local workspace directory and push the unified declarative configurations down to your active origin repository branch:

```bash
cd /d/Projects/3-Tier_Microservices_Application/retail-infra-gitops
git add .
git commit -m "feat: align structural selectors, inject absolute runtime variables, and mount default database seeding engine"
git push origin main

```

### 2. Force ArgoCD Synchronization

Force the active controller to discard existing cache properties and initialize an explicit synchronization evaluation loop against the core repositories:

```bash
kubectl annotate application retail-3tier-production -n argocd argocd.argoproj.io/refresh=hard --overwrite

```

### 3. Evict Stale Sandbox Blocks

If a duplicate placeholder execution pod exists from a previous lifecycle test, execute a force-purge to ensure the newly declared schema job launches flawlessly:

```bash
kubectl delete pod db-table-injector --namespace production --force --grace-period=0 2>/dev/null || true

```

### 4. Recycle Running Deployment Pods

Force the Node.js business backend app to drop dead transaction loops, reload its internal connection pool patterns, and fetch live values out of the fresh database schema context:

```bash
kubectl rollout restart deployment retail-backend --namespace production

```

---

## 🌐 Local Workspace Access Routine

Since the enterprise runtime environments are protected within private EKS network boundaries, establish dedicated background execution channels on your Windows 11 system via Git Bash to preview the deployment:

### Step 1: Open Terminal Window 1 (Frontend Gateway Integration)

Map traffic loops for the user interface layout down onto local port `3000`:

```bash
kubectl port-forward svc/retail-frontend-service 3000:80 -n production

```

### Step 2: Open Terminal Window 2 (Backend Core Engine Proxy)

Map data API endpoints down to port `5000` to guarantee that runtime callbacks do not collide with active ArgoCD administration console ports (`8080`):

```bash
kubectl port-forward svc/retail-backend-service 5000:8080 -n production

```

### Step 3: Run Cache-Cleared Client Preview

Open your target browser, navigate to your local interface node address, and clear the local browser state memory arrays by issuing a hard reload command:

* **Address Location:** `http://localhost:3000`
* **Hard Refresh Sequence:** Press **`Ctrl + F5`** (or hold down `Shift` while selecting the reload icon).

The data loading fault banner will transition immediately into your production-ready, fully green microservices platform catalog grid view!
"""

# Write out the completed README.md file

output_filename = "README.md"
with open(output_filename, "w", encoding="utf-8") as f:
f.write(readme_content)

print(f"Successfully generated {output_filename}")

```