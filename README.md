# Homelab

Production-grade self-hosted infrastructure running on a single-node Debian server. Managed 
via GitOps with full observability, centralized log aggregation, zero-trust networking, and 
automated alerting.

## Hardware

| Component | Specification               |
|-----------|-----------------------------|
| CPU       | Intel N150                  |
| RAM       | 32GB DDR4                   |
| OS        | Debian 13 (Trixie)          |
| Storage   | ~45TB usable (mergerFS + SnapRAID parity) |

## Stacks

| Stack      | Key Services                                    |
|------------|-------------------------------------------------|
| Metrics    | Prometheus, Grafana, Loki, Uptime Kuma          |
| Media      | Plex, Jellyfin, Sonarr, Radarr, Seerr           |
| Downloads  | qBittorrent, Prowlarr, Gluetun                  |
| Networking | AdGuard Home, NGINX Proxy Manager               |
| Management | Dozzle, Homepage, Komodo                        |

<details>
<summary>Full service list</summary>

| Stack      | Services                                                                    |
|------------|-----------------------------------------------------------------------------|
| Metrics    | Prometheus, Grafana, Loki, Promtail, Uptime Kuma, cAdvisor, Node Exporter  |
| Media      | Plex, Jellyfin, Sonarr ×2, Radarr, Seerr, PlexAutoLanguages, Watchstate, Agregarr |
| Downloads  | qBittorrent, RDTClient, Prowlarr, FlareSolverr, Gluetun                    |
| Networking | AdGuard Home, NGINX Proxy Manager, Homebridge                               |
| Management | Dozzle, Homepage                                                            |

</details>

## Observability

- **Metrics** — Prometheus scrapes Node Exporter (host) and cAdvisor (containers) every 15s.
  Grafana dashboards cover host metrics (CPU, RAM, disk, network) and per-container resource usage.
- **Logs** — Promtail ships all container logs to Loki via Docker service discovery, tagged by
  container name, compose project, and log stream. Queryable in Grafana via LogQL.
- **Uptime** — Uptime Kuma monitors every service with HTTP health checks and tracks response
  time and uptime percentages.
- **Alerting** — Grafana alerts on disk usage >80%, memory pressure >85%, and container
  availability. Notifications route to Discord.

## Infrastructure

- **Orchestration** — Komodo syncs this repository and manages stack deployments via GitOps.
  Compose files are the source of truth. Changes are tested locally first, then promoted to
  Git and redeployed through Komodo.
- **Networking** — All services run on an internal Docker bridge network, exposed selectively
  via NGINX Proxy Manager with SSL termination. Host networking used only where technically
  required (AdGuard Home, Homebridge, Node Exporter).
- **VPN** — Gluetun manages a single WireGuard tunnel. qBittorrent, Prowlarr, and FlareSolverr
  route through it via `network_mode: service:gluetun`. Ports are exposed on the Gluetun
  container, not the individual services.
- **Secret Management** — Secrets are never committed to the repository. `.env` files are
  gitignored and managed on the host. `.env.example` files document required variables per
  stack. Application data is stored at `/opt/appdata` on the host, outside the repository.

## Storage

- **Pooling** — mergerFS combines physical disks into a single mount at `/mnt/nas` using a
  most-free-space allocation policy to balance writes across the array.
- **Parity** — SnapRAID provides snapshot-based parity protecting against single drive failure.
  Syncs run on a schedule via snapraid-runner with safety thresholds to prevent accidental
  data loss.
- **Layout** — Media and torrent data live on the DAS at `/mnt/nas`. Application config and
  metadata live on the host SSD at `/opt/appdata` for fast random I/O.

## Security

- SSH key-only authentication, password auth disabled
- UFW firewall with strict ingress rules
- Zero-trust remote access via Tailscale
- Host networking limited to services that require it
- No secrets in version control

## Repository Structure

```
stacks/
  metrics/        # Prometheus, Grafana, Loki, Promtail, Uptime Kuma
  media/          # Plex, Jellyfin, *arr stack, Seerr
  downloads/      # qBittorrent, RDTClient, Prowlarr, FlareSolverr, Gluetun
  networking/     # AdGuard Home, NGINX Proxy Manager, Homebridge
  monitoring/     # Dozzle, Homepage
```
