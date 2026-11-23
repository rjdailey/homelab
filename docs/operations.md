# Homelab operations

## Rebuilding the host

1. Mount the mergerFS pool at `/mnt/nas` before deploying anything that uses it. Media and downloads live under `/mnt/nas/data`. Restore the app configs and databases under `/opt/appdata`.
2. Create the shared Docker network with `docker network create proxy` if it is missing. The stacks that join it expect it to exist.
3. Restore the named Docker volumes before bringing up the stacks. These are the names on `nas`.

   | Volume | Data |
   | --- | --- |
   | `prometheus_data` | Prometheus time series |
   | `loki_data` | Loki log data |
   | `grafana_data` | Grafana database and settings |
   | `komodo_mongo-data`, `komodo_mongo-config` | Komodo’s MongoDB data and configuration |

4. Restore `komodo/.env` using the host’s saved values and use [Komodo’s Compose file](../komodo/compose.yaml) to bring up Core, MongoDB, and Periphery. Komodo writes database backups to the host path set by `COMPOSE_KOMODO_BACKUPS_PATH`. Keep the Periphery root at the same path inside and outside its container.
5. In Komodo, confirm the linked repo and each stack’s run directory under `stacks/`. Restore the stack environment values from your secrets backup, using the checked-in `.env.example` files as references. Confirm the self-hosted GitHub Actions runner is online and has the Komodo API credentials through GitHub Actions secrets.

Compose may warn that an existing volume was created outside the project. Check that the running service mounts the expected volume with `docker inspect <container>`. Preserve the existing volumes when redeploying. In particular, avoid `docker compose down -v` on stacks whose data you need.

### Ebook ingest

Shelfmark writes to `/mnt/nas/data/torrents/books`. Calibre-Web Automated mounts that path as its ingest directory and removes files from it after processing. The Calibre library itself lives at `/opt/appdata/calibre-web-automated/library`.

## Verifying a deploy

- On a push to `main`, open the [Validate workflow](../.github/workflows/validate.yaml) run. The deploy job prints a Komodo update ID for each changed stack and waits for the result.
- In Komodo, open the stack’s update history. Check the Git revision and the Compose stages. If a config-only change was pulled and the running service still uses the old file, restart the affected service and verify it loaded the change. A single-file bind mount can keep pointing at the old file after Git replaces it.
- On the host, check container state with `docker ps -a` and recent logs with `docker logs --tail 100 <container>`. Gatus checks Loki’s readiness and reports service availability. To check Gatus itself from the host, run `curl -fsS http://127.0.0.1:8080/health`.
- For image updates, open **Global Auto Update → Auto Pull** in Komodo. Check that the expected stacks appear in the pull summary. Komodo v1.19.5 only pulls stacks it sees as **Running**.
