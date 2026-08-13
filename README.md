# Frappe v16 Builder & DevContainer System

A production-ready 2-tier Docker build system and 1-click VS Code DevContainer development environment for Frappe Framework v16, ERPNext, HRMS, Healthcare, CRM, and Custom Applications.

---

## 🏗️ Architecture Overview

This repository provides two isolated, self-contained workflows:

1. **Local Development Workflow (`development/`)**:
   - 1-Click VS Code Linux DevContainers pre-configured with Python 3.14, Node.js 24, and Frappe v16.
   - Resource-optimized multi-stack support: Shared MariaDB (`dev-mariadb`) and Redis (`dev-redis-*`) containers across all dev stacks (**66%+ RAM savings**).

2. **Production Build & Deployment Workflow (`deployment/`)**:
   - **Base Image Build (`deployment/images/custom/Containerfile`)**: Compiles heavy Python dependencies and core apps specified in `deployment/apps.base.json` (Run once, ~8–12 mins).
   - **Layered Custom App Build (`deployment/images/custom/Containerfile.layered`)**: Rapidly layers custom app source code specified in `deployment/apps.custom.json` on top of the base image (~25 seconds).
   - **Production Stack (`deployment/pwd.yml`)**: Production-ready Docker Compose stack (`frappe-prod`).

---

## 🎯 Select Your Workflow

Click on your required workflow below to open the dedicated documentation guide:

| Objective | Description | Dedicated Guide Link |
| :--- | :--- | :--- |
| **💻 Local Feature Development** | Set up VS Code DevContainers, run bench commands, install custom apps, or spin up multiple dev stacks | [👉 Open Development Guide](development/README.md) |
| **🚀 Production Build & Deploy** | Build 2-tier Docker images (base & layered) and deploy the production stack | [👉 Open Production Deployment Guide](deployment/README.md) |

---

## ⚙️ General System Prerequisites

Before starting, ensure your host system (Windows, Linux, or macOS) has:

- **Docker Desktop** (Windows / macOS) with WSL 2 backend enabled, OR **Docker CE** (Linux).
- **Docker Buildx Plugin**: `docker buildx version`
- **Docker Compose v2**: `docker compose version`

---

## 📁 Repository Structure

```bash
.
├── development/                 # Isolated local development directory
│   ├── README.md                # 📖 Local Development Guide
│   ├── compose.shared.yml       # Shared infrastructure stack (MariaDB & Redis)
│   ├── compose.dev.yml          # Dev container Compose stack (connected to frappe-dev-network)
│   ├── apps.base.json           # Base apps manifest
│   ├── .env                     # Dev environment configuration
│   └── images/
│       └── dev/
│           └── Containerfile    # DevContainer build definition
│
├── deployment/                  # Isolated production deployment directory
│   ├── README.md                # 📖 Production Deployment Guide
│   ├── pwd.yml                  # Production Docker Compose stack
│   ├── apps.base.json           # Base apps manifest for base image build
│   ├── apps.custom.json         # Custom apps manifest for layered image build
│   ├── apps.custom.example.json # Example custom app manifest template
│   ├── .env                     # Production environment configuration
│   ├── images/
│   │   └── custom/
│   │       ├── Containerfile        # Base production image Dockerfile
│   │       └── Containerfile.layered# Layered production image Dockerfile
│   └── resources/
│       └── core/                # Nginx templates & entrypoint scripts
│
├── .devcontainer/               # VS Code DevContainer stack configurations
│   └── stack-1/                 # Active developer stack (Port 8000)
│       ├── devcontainer.json    # Points to ../../development/compose.dev.yml
│       ├── docker-compose.override.yml
│       ├── .env
│       └── .env.example
│
├── README.md                    # Root Documentation Hub & System Overview
├── .gitignore
├── .dockerignore
└── .editorconfig
```
