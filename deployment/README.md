# 🚀 Production Build & Deployment Guide

For DevOps engineers building production Docker images and deploying production stacks.

---

## 1. Deployment Prerequisites

1. Install **Docker CE & Docker Buildx Plugin** on host or CI/CD build runner.
2. Verify Buildx: `docker buildx version`.

---

## 2. Understanding Deployment Files

| File Path | Purpose |
| :--- | :--- |
| **`deployment/pwd.yml`** | Production Docker Compose stack (`frappe-prod`) |
| **`deployment/images/custom/Containerfile`** | Base production image Dockerfile |
| **`deployment/images/custom/Containerfile.layered`** | Layered production image Dockerfile |
| **`deployment/apps.base.json`** | Base apps manifest for base image builds |
| **`deployment/apps.custom.json`** | Custom app Git repositories & tokens for layered builds |
| **`deployment/resources/core/`** | Core Nginx templates and container entrypoint scripts |
| **`deployment/.env`** | Production environment configuration |
| **`deployment/.env.example`** | Production environment template reference |

---

## 3. Step 1: Build Base Image (Run Once)

Compiles heavy Python dependencies, Node.js tooling, and fixed core apps specified in `deployment/apps.base.json`.

**PowerShell / Bash:**
```bash
docker buildx build \
  -f deployment/images/custom/Containerfile \
  --secret id=apps_json,src=deployment/apps.base.json \
  --build-arg FRAPPE_BRANCH=v16.30.0 \
  --tag custom-base:v16 \
  --load .
```
*Build time: ~8–12 minutes (run once or when core dependencies change).*

---

## 4. Step 2: Build Layered Custom App Image (Fast Updates)

When releasing custom app updates (every 2–3 days), list custom app Git URLs and branches in `deployment/apps.custom.json` and build on top of `custom-base:v16`.

1. **Configure `deployment/apps.custom.json`**:
   ```json
   [
     {
       "url": "https://<TOKEN>@dev.azure.com/org/repo/_git/project_management",
       "branch": "version-16"
     }
   ]
   ```

2. **Build Layered Image**:
   ```bash
   docker buildx build \
     -f deployment/images/custom/Containerfile.layered \
     --secret id=apps_json,src=deployment/apps.custom.json \
     --build-arg BASE_IMAGE=custom-base:v16 \
     --tag custom-app:latest \
     --load .
   ```
*Build time: ~25 seconds.*

---

## 5. Step 3: Deploy Production Stack & Run Migrations

1. **Deploy Production Stack**:
   ```bash
   docker compose -f deployment/pwd.yml up -d
   ```

2. **Install Custom App onto Site & Run Database Migration**:
   ```bash
   docker exec frappe-prod-backend-1 bench --site frontend install-app project_management
   docker exec frappe-prod-backend-1 bench --site frontend migrate
   ```

3. **Access Production App**: Open `http://localhost:8080` (Username: `Administrator`, Password: `admin`).
