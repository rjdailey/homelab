# Self-Hosted Homelab

## Overview
A self-hosted, automated homelab environment built to simulate production infrastructure. This repository manages the deployment and configuration of various containerized services using GitOps methodologies, focusing on secure access, automated deployments, and infrastructure as code (IaC).

## Architecture & Tech Stack
|Component    |Specification                |
|-------------|-----------------------------|
|CPU          |Intel N150                   |
|RAM          |32GB DDR4                    |
|OS           |Debian GNU/Linux 13 (Trixie) |
|Storage Pool |~45TB + parity               |

* **Orchestration:** Komodo 
* **Containerization:** Docker / Docker Compose
* **Networking & Proxy:** NGINX Reverse Proxy
* **VPN / Remote Access:** Tailscale / WireGuard

## Storage Stack (JBOD with Parity)
To balance storage efficiency and data safety, this environment utilizes a combination of mergerFS for pooling and SnapRAID for parity protection.
* **Pooling (mergerFS):** Combines multiple physical disks (/mnt/disk*) into a single virtual mount point (/mnt/nas). It uses the mfs (most free space) create policy to balance write operations across the array.
* **Parity (SnapRAID):** A snapshot-based backup/parity system. It provides protection against up to one simultaneous drive failure without the overhead or complexity of traditional RAID.

## Key Technical Decisions & Engineering Workflow

### Infrastructure as Code (IaC) & Automation
Rather than manually deploying containers, this environment utilizes **Komodo** for container orchestration. By syncing this Git repository with Komodo, the infrastructure state is declared in code, allowing for automated, repeatable deployments that mirror enterprise GitOps workflows.

### Secret Management & Environment Security
To align with production security standards, this repository does not rely on local `.env` files. Instead, it utilizes **environment variable injection** at the orchestration level. This ensures that sensitive credentials, API keys, and database passwords are securely managed outside the codebase, making the infrastructure highly portable and secure.

## Security & Maintenance Posture
* **Access Control:** SSH key-only access with strict firewall hardening (UFW).
* **Remote Management:** Secure, zero-trust remote access managed via Tailscale/WireGuard.
* **Data Integrity:** Automated parity syncs and bitrot scrubbing are orchestrated via `snapraid-runner`, which provides email alerting and a safety threshold to prevent accidental data loss during syncs.
* **Maintenance:** Automated system updates and configuration backups executed via custom cron jobs to minimize downtime and manual intervention.
