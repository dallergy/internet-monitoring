# Internet Monitoring Stack

A Docker Compose stack for monitoring your home network with [Prometheus](https://prometheus.io/), [Grafana](https://grafana.com/), the [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter), and a [Speedtest exporter](https://github.com/kutovoys/speedtest-exporter).

The stack probes HTTP endpoints (router, DNS, websites), records download/upload speed over time, and ships a pre-provisioned Grafana dashboard.

## Prerequisites

- Docker Engine 24+ with the [Compose plugin](https://docs.docker.com/compose/install/) (`docker compose`)
- Outbound internet access (for Speedtest and external probes)

## Quick start

```bash
git clone https://github.com/dallergy/internet-monitoring.git
cd internet-monitoring
cp .env.example .env   # optional: set Grafana password and speedtest interval
docker compose up -d
```

Open Grafana at [http://localhost:3030/d/o9mIe_Aik/internet-connection](http://localhost:3030/d/o9mIe_Aik/internet-connection).

Default credentials:

- Username: `admin`
- Password: `changeme` (or the value of `GRAFANA_ADMIN_PASSWORD` in `.env`)

## Configuration

### Monitored endpoints

Edit [`prometheus/pinghosts.yaml`](./prometheus/pinghosts.yaml). Each target uses the format:

```yaml
- targets:
    - url;humanname;routing;switch
```

Example:

```yaml
- targets:
    - https://1.1.1.1;cloudflare-dns;external;internet
    - http://192.168.1.1;router;local;lan
```

### Speedtest frequency

Speedtests run inside the exporter on a schedule (not on every Prometheus scrape). Adjust the interval in `.env`:

```bash
SPEEDTEST_INTERVAL_MINUTES=15
```

Or set `UPDATE_INTERVAL` directly on the `speedtest` service in `docker-compose.yml`.

### Optional Sonos exporter

The original stack included a Sonos exporter that requires host networking for discovery:

```bash
docker compose --profile sonos up -d
```

Edit the Sonos target in [`prometheus/prometheus.yml`](./prometheus/prometheus.yml) (`job_name: sonos`) to match your Sonos host IP.

The optional image is pinned to the March 2021 `maxandersen/sonos_exporter` digest. That image is unmaintained (Debian 10) and still has known OS vulnerabilities. There is no newer tag. Leave the profile off unless you accept that, or replace the exporter yourself.

## Services and ports

| Service    | Port  | Description                          |
|------------|-------|--------------------------------------|
| Grafana    | 3030  | Dashboards and visualization         |
| Prometheus | 9090  | Metrics database and query UI        |
| Blackbox   | 9115  | HTTP/TCP/ICMP probe exporter         |
| Speedtest  | 9696  | Speedtest.net metrics exporter       |
| Node exp.  | 9100  | Host metrics (CPU, memory, disk)     |
| Sonos      | 1915  | Optional (`--profile sonos`)         |

Useful URLs:

- Prometheus targets: http://localhost:9090/targets
- Blackbox exporter: http://localhost:9115
- Speedtest metrics: http://localhost:9696/metrics (first result may take a minute)

## What changed in this modernization

- Pinned container images (Prometheus 3.15, Grafana 12.4, Blackbox 0.29, Node Exporter 1.12)
- Compose v2 format (no legacy `version` / `links`)
- GitHub Actions CI instead of Travis CI
- Maintained Speedtest exporter (`kutovoys/speedtest-exporter`)
- Grafana 11 dashboard with native panels (stat, timeseries, status-history)
- Environment-based Grafana password via `.env`
- Generic example probe targets instead of personal hostnames

Prometheus 3 reads a TSDB written by 2.55. After upgrading, do not roll the Prometheus image back below 2.55 or the existing volume will be unreadable.

## Security notice

This stack is intended for home-lab use. It exposes admin UIs on local ports without TLS or hardening. Do not expose it to the public internet without additional security controls.

## Credits

Inspired by [vegasbrianc/github-monitoring](https://github.com/vegasbrianc/github-monitoring) and the original [maxandersen/internet-monitoring](https://github.com/maxandersen/internet-monitoring) project.
