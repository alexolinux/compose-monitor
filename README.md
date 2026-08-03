# compose-monitor

A lightweight monitoring stack for a Raspberry Pi or similar remote host, using Docker Compose, Prometheus, Grafana, and node-exporter.

This project is designed to run on a remote Raspberry Pi and expose a simple dashboard for system metrics without storing private host addresses in Git.

## Overview

The stack includes:

- Prometheus for scraping metrics
- node-exporter for system metrics
- Grafana for visualization
- Portainer for container management

The default repository configuration uses public-safe placeholder hostnames, while the real internal targets are kept in a local override file that is not committed to version control.

## Requirements

- Raspberry Pi (or any Linux machine with Docker and Docker Compose)
- Docker Engine
- Docker Compose v2
- Network access to the monitored hosts
- Optional: a local DNS name or private host entries for the internal nodes

## Project structure

- `docker-compose.yml` — base Compose configuration
- `prometheus.yml` — public-safe Prometheus config template
- `docker-compose.override.yml` — local runtime override for private targets
- `prometheus.local.yml` — locally stored real targets (not committed)
- `.env` — local runtime environment values

## Quick start

1. Clone the repository on the Raspberry Pi:

   ```bash
   git clone <your-repo-url>
   cd compose-monitor
   ```

2. Review the environment file and adjust values if needed:

   ```bash
   nano .env
   ```

3. Copy the local override template if you want to keep a separate private config:

   ```bash
   cp docker-compose.override.yml.example docker-compose.override.yml
   ```

4. Edit the local Prometheus target file with the real private addresses used in your network:

   ```bash
   nano prometheus.local.yml
   ```

   Example:

   ```yaml
   global:
     scrape_interval: 1m

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']

     - job_name: 'node-raspberry'
       static_configs:
         - targets: ['192.168.1.75:9100']

     - job_name: 'node-blackberry'
       static_configs:
         - targets: ['192.168.1.80:9100']
   ```

5. Start the stack:

   ```bash
   docker compose up -d
   ```

6. Check that the services are running:

   ```bash
   docker compose ps
   ```

## Accessing the services

After startup:

- Prometheus: http://<raspberry-host>:9090
- Grafana: http://<raspberry-host>:3003
- Portainer: http://<raspberry-host>:9000

Default Grafana credentials are configured in `.env`:

- username: `GRAFANA_USERNAME`
- password: `GRAFANA_PASSWORD`

## Security note

The repository is intentionally kept public-safe. It does not include private IP addresses or internal hostnames. The real targets are stored in a local override file that should not be committed. This avoids exposing internal infrastructure details in Git while still allowing the stack to work in a private network.

## Useful commands

```bash
# View logs
 docker compose logs -f prometheus
 docker compose logs -f grafana

# Restart services
 docker compose restart

# Stop services
 docker compose down
```

## Troubleshooting

### Prometheus is not scraping targets

- Confirm that the target host is reachable from the Raspberry Pi.
- Verify the `node-exporter` service is running on the remote machine.
- Check the Prometheus target status in the web UI.
- Ensure the private addresses in `prometheus.local.yml` are correct.

### Grafana is not loading

- Verify that Docker containers are healthy.
- Check the logs:

  ```bash
  docker compose logs -f grafana
  ```

### Port conflict

A port may already be in use by another service. Update the Compose port mappings in `docker-compose.yml` if needed.

## Notes

- The base repository config uses placeholder hostnames such as `raspberry-node.local` and `blackberry-node.local`.
- These are safe to commit.
- The real internal values are intentionally placed in the local override file only.
- This setup is intended for a remote Raspberry deployment and should not be used as a public production monitoring setup without additional hardening.
