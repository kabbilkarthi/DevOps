# Grafana-Prometheus DevOps Project

This project sets up a **Grafana** and **Prometheus** monitoring stack running inside Docker containers using `docker-compose`. It also includes **Node Exporter** for system metrics collection and **Jenkins** for automating the build and deployment processes. The Jenkins agent node also serves as the **Ansible control node** for configuring and deploying services.

![image](https://github.com/user-attachments/assets/a4779677-688e-46b6-a710-d688223f6f71)

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Docker Compose Configuration](#docker-compose-configuration)
- [Prometheus Configuration](#prometheus-configuration)
- [Jenkins Configuration](#jenkins-configuration)
- [Ansible Playbook](#ansible-playbook)

---

## Project Overview

This DevOps setup enables:
- **Monitoring:** Grafana for visualization and Prometheus for metrics collection.
- **Metrics Collection:** Node Exporter collects system metrics from servers.
- **CI/CD Pipeline:** Jenkins automates the deployment using webhooks on code pushes.
- **Containerized Services:** All components run inside Docker containers for easy management.

---

## Prerequisites

Before starting, ensure the following tools are installed:

- **Docker**
- **Docker Compose**
- **Jenkins**
- **Ansible** (on the Jenkins agent node, which acts as the Ansible control node)

---

## Setup Instructions

### Docker Compose Configuration

Create a `docker-compose.yml` file in the project directory with the following content:

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
      - GF_SECURITY_ADMIN_PASSWORD=<PASSWD>
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
        - '192.168.29.28:9100' # Node exporter running
        - '192.168.29.58:9100' # Node exporter running
        - '192.168.29.128:9100' # Node exporter running
        - '192.168.29.174:9100' # Node exporter running
        - '192.168.29.234:9100' # Node exporter running
``` 
## Jenkins Configuration

Set up a Jenkins job to trigger on push events from your GitHub repository using webhooks. This ensures that any code modifications pushed to the repository automatically trigger the build process.

Job Steps:
1. Checkout Code: Pull the latest code from the GitHub repository.
2. Run Ansible Playbooks: Configure the job to run the Ansible playbooks for Node Exporter and Grafana-Prometheus setup.

## Ansible Playbook

Create a 'grafana-prometheus.yml' file with the following content:

```yaml
---
- name: Node Exporter config
  hosts: all
  gather_facts: no
  ignore_unreachable: yes
  tasks:
    - name: Running Node Exporter Configuration
      get_url:
        url: https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
        dest: /root/node_exporter-1.8.2.linux-amd64.tar.gz

    - name: Extracting the downloaded Node Exporter
      unarchive:
        src: /root/node_exporter-1.8.2.linux-amd64.tar.gz 
        dest: /root/
        remote_src: yes

    - name: Kill existing Node Exporter processes 
      shell: pkill -f node_exporter 
      ignore_errors: yes
    
    - name: Run Node Exporter
      shell: nohup ./node_exporter > node_exporter.log 2>&1 &
      args:
        chdir: /root/node_exporter-1.8.2.linux-amd64

- name: grafana-prometheus playbook config
  hosts: devops
  become_user: devops
  tasks:
    - name: Copy Docker Compose file
      copy:
        src: /home/ansible/DEVOPS/DevOps/Grafana-Prometheus/docker-compose.yml
        dest: /home/devops/docker-compose.yml

    - name: Ensure Prometheus configuration directory exists
      file:
        path: /home/devops/prometheus
        state: directory

    - name: Copy Prometheus configuration file
      copy:
        src: /home/ansible/DEVOPS/DevOps/Grafana-Prometheus/prometheus/prometheus.yml
        dest: /home/devops/prometheus/prometheus.yml

    - name: Ensure Grafana data directory exists
      file:
        path: /home/devops/grafana
        state: directory

    - name: Start the Grafana and Prometheus containers by Removing existing Containers
      shell: |
        docker-compose down
        docker rmi prom/prometheus:latest
        docker rmi grafana/grafana:latest
        docker-compose up -d
      args:
        chdir: /home/devops
```
