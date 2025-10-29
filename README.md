## DevSecOps Tasks 1–3 (Clair v2, Nextcloud, Portainer)

This repository contains three Docker-based setups:

- Clair v2 vulnerability scanner backed by PostgreSQL
- Nextcloud with a PostgreSQL database
- Portainer Community Edition (CE)

All stacks are self-contained with Docker Compose files for quick local deployment.

### Repository structure

- `clair-v2/`
  - `docker-compose.yml`: Clair v2 + PostgreSQL stack
  - `docker-compose.override.yml`: Optional CA trust for scanning private registries
  - `config/config.yaml`: Clair runtime configuration (ports, DB connection, updater)
  - `clair-config-v2.yaml`: Alternative Clair config (not used by default)
  - `initdb/01-init.sql`: Example DB user/database bootstrap (not used by default)
- `secdevops-a3/nextcloud/`
  - `compose.yaml`: Nextcloud + PostgreSQL stack (host port 8080)
  - `nextcloud-swarm.yaml`: Swarm-mode variant
  - `README.md`: Example usage and screenshots
- `secdevops-a3/portainer/`
  - `compose.yaml`: Portainer CE (host port 9443)
  - `README.md`: Example usage
- `clairctl_21920794.zip`: Clair CLI bundle for client-side scans (optional)

### Prerequisites

- Docker Engine and Docker Compose v2 (`docker compose`)
- 2+ GB RAM and 5+ GB free disk space recommended

## Quick start

### 1) Clair v2

Deploy Clair v2 with PostgreSQL:

```bash
cd clair-v2
docker compose up -d
```

Ports exposed on the host:

- Clair API: `http://localhost:6063` (container 6060)
- Health: `http://localhost:6064/health` (container 6061)

Tail logs if needed:

```bash
docker compose logs -f clair
```

Scan images using your preferred client (e.g., clairctl or clair-scanner) by pointing it at `http://localhost:6063`. The provided `clairctl_21920794.zip` contains a CLI bundle you can extract and configure; refer to the tool’s documentation for usage.

Notes:

- If Clair can’t reach PostgreSQL, verify the DSN in `clair-v2/config/config.yaml`. Using the compose network hostname is usually simplest, for example:

  ```yaml
  source: host=postgres port=5432 user=postgres dbname=clair6000 sslmode=disable statement_timeout=120000
  ```

- If you need Clair to trust a private registry’s self-signed CA, place your certificate at `/opt/registry/certs/localhost.crt` on the host and use `docker-compose.override.yml` (auto-installed into the container at startup).

Stop the stack:

```bash
docker compose down
```

### 2) Nextcloud

Deploy Nextcloud with PostgreSQL:

```bash
cd secdevops-a3/nextcloud
docker compose up -d
```

Access the web UI at `http://localhost:8080` and complete the initial setup. Data persists in the `db_data` and `nc_data` volumes.

Stop the stack:

```bash
docker compose down
```

### 3) Portainer (CE)

Deploy Portainer CE:

```bash
cd secdevops-a3/portainer
docker compose up -d
```

Access the web UI at `https://localhost:9443` (self-signed certificate). Create the admin user and connect to the local environment.

Stop the stack:

```bash
docker compose down
```

## Troubleshooting

- Clair DB connectivity: ensure the `postgres` service is healthy and the DSN in `clair-v2/config/config.yaml` points to `host=postgres port=5432` on the compose network.
- Port conflicts: change the left-hand side host ports in the compose files if required (e.g., `8080:80`, `9443:9443`, `6063:6060`, `6064:6061`).
- Self-signed TLS: for Portainer, your browser may require you to proceed past the warning. For Clair + private registries, use the provided override file to install your CA.

## Cleanup

From each subdirectory, run:

```bash
docker compose down -v
```

This stops containers and removes named volumes (data loss). Omit `-v` to keep data.

---

Student ID: 21920794

