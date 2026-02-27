# Homelab

![Debian](https://img.shields.io/badge/Debian-D70A53?style=for-the-badge&logo=debian&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![NGINX](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Tailscale](https://img.shields.io/badge/Tailscale-%23FFFFFF.svg?style=for-the-badge&logo=tailscale&logoColor=black)

## 📌 Overview
A self-hosted, automated homelab environment built to simulate production infrastructure. This repository manages the deployment and configuration of various containerized services using GitOps methodologies, focusing on secure access, automated deployments, and infrastructure as code (IaC).

## 🏗️ Architecture & Tech Stack
* **OS:** Debian Linux
* **Orchestration:** Komodo 
* **Containerization:** Docker / Docker Compose
* **Networking & Proxy:** NGINX Reverse Proxy
* **VPN / Remote Access:** Tailscale / WireGuard

## ⚙️ Key Technical Decisions & Engineering Workflow

### Infrastructure as Code (IaC) & Automation
Rather than manually deploying containers, this environment utilizes **Komodo** for container orchestration. By syncing this Git repository with Komodo, the infrastructure state is declared in code, allowing for automated, repeatable deployments that mirror enterprise GitOps workflows.

### Secret Management & Environment Security
To align with production security standards, this repository does not rely on local `.env` files. Instead, it utilizes **environment variable injection** at the orchestration level. This ensures that sensitive credentials, API keys, and database passwords are securely managed outside the codebase, making the infrastructure highly portable and secure.

## 🔒 Security & Maintenance Posture
* **Access Control:** SSH key-only access with strict firewall hardening (UFW).
* **Remote Management:** Secure, zero-trust remote access managed via Tailscale/WireGuard.
* **Maintenance:** Automated system updates and configuration backups executed via custom cron jobs to minimize downtime and manual intervention.
