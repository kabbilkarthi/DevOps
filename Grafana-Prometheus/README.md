# Grafana Prometheus Configuration and Startup

## Overview
This document provides instructions on how to configure and start Grafana and Prometheus using Docker Compose.

## Prerequisites
- Docker installed and running
- Docker Compose installed
- Node Exporter configured and running on the needed hosts

## Docker Compose Configuration

Create a `docker-compose.yml` file with the following content:

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=kabbil
    volumes:
      - ./grafana:/var/lib/grafana
    restart: unless-stopped

networks:
  default:
    driver: bridge
```
## Prometheus Configuration
Create a Prometheus configuration file at prometheus/prometheus.yml with the following content, and modify localhost to the IP address of your Grafana server:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
        - '192.168.29.28:9100'
        - '192.168.29.58:9100'
```
## Starting the Services
To start the services, navigate to the directory containing your docker-compose.yml file and run:

docker-compose up -d   # This command will start Prometheus and Grafana in detached mode.

## Node Exporter Configuration
Node Exporter needs to be configured and running on the hosts you want to monitor. You can start Node Exporter with the following command:

./node_exporter &
Make sure Node Exporter is running on the specified targets (e.g., 192.168.29.28:9100 and 192.168.29.58:9100).

## Accessing Grafana and Prometheus

- **Grafana**: Open your web browser and navigate to `http://<your-grafana-server-ip>:3000`. Login with the default username `admin` and the password you set in the environment variable (`GF_SECURITY_ADMIN_PASSWORD`).

- **Prometheus**: Open your web browser and navigate to `http://<your-grafana-server-ip>:9090`.