# compose-monitor

A lightweight monitoring stack for Raspberry Pi hosts using Docker Compose, Prometheus, Grafana, and node-exporter.

This project monitors multiple Linux hosts in the same local network, with a Grafana dashboard ready for primary system metrics such as CPU, memory, network usage, disk, load average, and uptime.

## Overview

The stack includes:

- Prometheus for metric scraping
- node-exporter for node metrics
- Grafana for visualization
- Portainer for container management

The default setup is prepared to monitor at least two Raspberry hosts: `raspberry` and `blackberry`.

## Requirements

- Raspberry Pi or any Linux host with Docker and Docker Compose
- Docker Engine
- Docker Compose v2
- Network access to the monitored hosts
- Optional: local DNS or static private IPs in the network

## Project structure

- `docker-compose.yml` — base services and ports
- `docker-compose.override.yml` — local runtime override for private targets
- `prometheus.yml` — public-safe Prometheus template
- `prometheus.local.yml` — local list of hosts to scrape
- `grafana/provisioning/datasources/datasource.yml` — Grafana datasource config
- `grafana/provisioning/dashboards/dashboard.yml` — dashboard auto-provisioning
- `grafana/provisioning/dashboards/raspberry-system-overview.json` — final dashboard
- `.env` — local credentials and runtime values

## Dashboard included

The project already ships with a Grafana dashboard named:

- `Raspberry Pi System Overview`

It includes:

- CPU usage
- Memory usage
- Swap usage
- Network throughput
- Disk usage
- Load average
- Uptime
- Host selector for `raspberry` and `blackberry`

## Quick start

1. Clone the repository on the Raspberry Pi or monitoring host:

   ```bash
   git clone <your-repo-url>
   cd compose-monitor
   ```

2. Review the environment settings:

   ```bash
   nano .env
   ```

3. Confirm the hosts to be monitored:

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

4. Start the monitoring stack:

   ```bash
   docker compose up -d
   ```

5. Check the services:

   ```bash
   docker compose ps
   ```

## Accessing the services

After startup:

- Prometheus: http://<monitoring-host>:9090
- Grafana: http://<monitoring-host>:3003
- Portainer: http://<monitoring-host>:9000

Default Grafana credentials are configured in `.env`:

- username: `GRAFANA_USERNAME`
- password: `GRAFANA_PASSWORD`

## Configure the monitored Raspberry hosts

Each Raspberry host that should be monitored must expose Prometheus metrics via node-exporter on port `9100`.

Example command to run on each target host:

```bash
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 9100:9100 \
  -v "/proc:/host/proc:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/:/rootfs:ro" \
  quay.io/prometheus/node-exporter:latest \
  --path.procfs=/host/proc \
  --path.rootfs=/rootfs \
  --path.sysfs=/host/sys
```

Then verify the metrics endpoint:

```bash
curl http://<host-ip>:9100/metrics
```

## Grafana dashboard usage

Once the stack is running:

1. Open Grafana at http://<monitoring-host>:3003
2. Log in with the configured credentials
3. Open the dashboard `Raspberry Pi System Overview`
4. Use the `Host` selector to view either:
   - `All`
   - `raspberry`
   - `blackberry`

The dashboard is designed to work with both hosts at the same time, comparing health and usage in real time.

## Security note

This project intentionally keeps private IP addresses out of the main repository. The real targets are stored in the local override and local Prometheus config files, which should stay outside version control if needed.

## Useful commands

```bash
# View logs
 docker compose logs -f prometheus
 docker compose logs -f grafana

# Restart the stack
 docker compose restart

# Stop the stack
 docker compose down
```

## Troubleshooting

### Prometheus is not scraping targets

- Confirm the monitored hosts are reachable from the monitoring Raspberry.
- Verify that `node-exporter` is listening on port `9100` on each host.
- Check targets in Prometheus at http://<monitoring-host>:9090/targets
- Validate the IP and port values in `prometheus.local.yml`

### Grafana is not loading

- Verify the containers are running:

  ```bash
  docker compose ps
  ```

- Check the logs:

  ```bash
  docker compose logs -f grafana
  ```

### Port conflict

A service may already be using the configured port. Adjust the ports in `docker-compose.yml` if necessary.

## Notes

- This project is intended for a private LAN or local monitoring scenario.
- It is a practical monitoring setup for Raspberry Pi hosts and similar Linux devices.
- The dashboard is optimized for primary operational metrics rather than deep application tracing.
