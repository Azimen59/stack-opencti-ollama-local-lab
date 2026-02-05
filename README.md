# Cyberwatch Local Lab – OpenCTI + Ollama + OpenClaw

A reproducible CTI lab stack (OpenCTI 6.5 + Elasticsearch 8 + Ollama + OpenClaw gateway) for experimenting with AI‑augmented threat intelligence on a single Linux host.

## Overview

This repository provides a self‑hosted cyber threat intelligence lab combining:

- OpenCTI 6.5 (platform & worker)
- Elasticsearch 8.14 (HTTP, single‑node, no TLS)
- MinIO (S3‑compatible file storage)
- RabbitMQ (message broker)
- Redis (cache and queues)
- Ollama (local LLM server)

The OpenClaw gateway is intentionally **not** containerized here: it is expected to run on the host (or another machine) and connect to Ollama over HTTP.

> This is a **lab** setup for local experimentation, not a hardened production deployment.

## Architecture

- OpenCTI uses Elasticsearch for indexing, MinIO for file storage, RabbitMQ and Redis for internal messaging and caching.  
- Ollama exposes a local LLM over HTTP on port 11434.  
- OpenClaw gateway (launched outside Docker) can use Ollama as its LLM backend and later be integrated with OpenCTI via API.

A typical deployment runs the full stack on a single Linux VM (Ubuntu or similar) with Docker and docker‑compose.

## Requirements

- Linux host (tested on Ubuntu Server)
- Docker and docker‑compose installed
- Recommended minimum:
  - 4 vCPU
  - 8 GB RAM
  - 30 GB of free disk space

## Configuration

All sensitive values in `docker-compose.yml` are placeholders and **must** be changed before use:

- `CHANGE_ME_ADMIN_PASSWORD`
- `CHANGE_ME_MINIO_PASSWORD`
- `CHANGE_ME_RABBITMQ_PASSWORD`
- `CHANGE_ME_VT_KEY`

You can optionally use a `.env` file based on `.env.example` to manage these variables centrally.

### Default admin (lab)

By default the compose uses:

- `APP__ADMIN__EMAIL=changeme@example.com`
- `APP__ADMIN__PASSWORD=CHANGE_ME_ADMIN_PASSWORD`
- `APP__ADMIN__TOKEN=11111111-1111-1111-1111-111111111111`

These are only for local lab usage and **must be replaced** in any serious environment.

## Quick start

Clone the repository:

```bash
git clone https://github.com/Azimen59/cyberwatch-local-lab.git
cd cyberwatch-local-lab

(Optional) Create a .env from the example and edit values:
cp .env.example .env
# edit .env with your own secrets

Start the stack:

docker-compose up -d
docker-compose ps

Accessing OpenCTI
OpenCTI is exposed on port 8080:

URL: http://<VM_IP>:8080

Admin email: changeme@example.com

Admin password: CHANGE_ME_ADMIN_PASSWORD

Change these values in your own deployment before exposing anything outside your lab.

Local LLM (Ollama)
Ollama runs in a dedicated container:

API: http://<VM_IP>:11434

Basic test from the host:
curl http://localhost:11434/api/tags
You can then pull models (e.g. llama3, mistral) using the standard Ollama CLI inside the container or from the host if you bind the socket appropriately.

OpenClaw gateway (outside Docker)
Once OpenClaw is installed on the host (or another machine on the same network), you can connect it to Ollama:
export OLLAMA_URL=http://localhost:11434
export OLLAMA_BASE_URL=http://localhost:11434

openclaw gateway --allow-unconfigured --bind lan
Then access the Canvas test page:
http://<VM_IP>:18793/__openclaw__/canvas/
If you do not plan to use VirusTotal, you can comment out or remove these services from docker-compose.yml.

Hardening & production notes
For a real deployment, you should:

Replace all default passwords and tokens, store them in a secrets manager or encrypted vault.

Put a reverse proxy (nginx/traefik) with HTTPS in front of OpenCTI and Ollama if exposed beyond localhost.

Restrict inbound ports on the host firewall (only expose what you actually need: 8080, 11434, 9000/9001, 15672, etc.).

Add monitoring (CPU/RAM/disk) given the resource usage of Elasticsearch and the LLM.

Roadmap
Add documented OpenCTI connectors (MISP, AbuseIPDB, etc.).

Publish example CTI playbooks leveraging OpenClaw and the local LLM.

Provide an architecture diagram and troubleshooting guide in docs/.

Optional: refactor the compose to fully use .env variables for secrets and tuning.


